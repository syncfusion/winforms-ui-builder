# Stage 5: Code Generation

**Purpose:** Generate production-ready WinForms code (C# + Designer + Services + Repositories) that is fully wired, compilable, and feature-complete. Code generation begins only after all pre-validation steps pass.

**Inputs:** `control-mapping.json` (Stage 3) + `skill-extraction.json` (Stage — Control Skill Extraction) + locked design decisions (Stage 4)
**Output:** Complete UI + backend implementation — zero stubs, zero missing handlers. Only controls verified in `skill-extraction.json` appear in any generated file.

---

## Pre-Validation Workflow (MANDATORY — Complete Before Any Code)

Execute all steps in order. ⛔ Do not skip any step.

### Step 1: Read and Validate `control-mapping.json`

1. Locate: `<project-root>/control-mapping.json`
2. Validate structure:
   - `project_type`: `"Simple"` or `"Complex"`
   - Simple → `elements[]` present with all controls
   - Complex → `pages[]` with `page_id`, `component_type` (`generate_form` | `generate_usercontrol`), `elements` or `sections`
   - Every element has: `id`, `name`, `type_hint`, mapped control name
3. Extract all unique mapped Syncfusion controls (e.g., `SfDataGrid`, `SfButton`, `TextBoxExt`)
4. Record page/form context per control (Simple = one context; Complex = per-page)

⛔ **If control-mapping.json is missing or invalid → HALT. Return to Stage 3.**

---

### Step 1A: 🔴 CONTROL VALIDATION GATE (BLOCKING — Before Skill Reading)

Inspect every entry in `mapped_controls[]`. Reject any control that is not a verified Syncfusion mapping before proceeding further.

```
FOR EACH entry in mapped_controls[]:

  CHECK control, validation, score, skill fields:

  IF control == "NATIVE_WINFORMS"
  OR validation == "✗ FALLBACK"
  OR validation == "✗ NO_MATCH"
  OR skill == null OR skill == ""
  OR score < 10
  → ❌ HALT:
      "❌ HALT: <element_name> is not a verified Syncfusion control
       validation='<validation>', score=<score>
       No valid skill mapping found.
       Return to Stage 3 and refine user intent or control mapping."

  IF validation == "✓ VERIFIED" AND skill != null AND score >= 10
  → ✅ Control cleared for extraction and code generation
```

**Allowed (✅):**
```
✅ emailInput → TextBoxExt  (validation=✓ VERIFIED, score=18.2)
   Proceed to skill extraction and code generation.
```

**Blocked (❌):**
```
❌ HALT: emailInput  validation='✗ FALLBACK'  score=8.5 (< 10 threshold)
   No verified Syncfusion control found.
   Must return to Stage 3 for remapping.
```

ALL entries pass → ✅ Gate cleared — proceed to Step 2
ANY entry fails → ❌ HALT entire pipeline; list all failing elements before stopping

---

### Step 2: 🔴 ATOMIC PRE-GENERATION VALIDATION GATE (BLOCKING — No Exceptions)

Execute for **every** control extracted in Step 1. Each condition is a hard stop — no fallback, no assumption, no partial continuation.

```
FOR EACH control in control-mapping.json:

  1. LOCATE skill folder: `<skills-root>/syncfusion-winforms-<control-name>/`
     `<skills-root>` = `.codestudio/skills | .agent/skills | .agents/skills | .github/skills | skills`
     ❌ Folder NOT found → HALT: "Skill folder missing for <control-name> — cannot proceed"

  2. READ `<skill-folder>/SKILL.md`  ← PRIMARY — read before any reference file
     ❌ File NOT found → HALT: "SKILL.md missing for <control-name> — cannot proceed"
     EXTRACT:
       ✅ Control overview and purpose
       ✅ Which reference files cover which features (guides Step 3 reference scanning)
       ✅ Feature-level documentation structure
       ✅ High-level expected APIs (used to cross-validate extraction)
     ❌ Do NOT read reference files before SKILL.md is processed

  3. READ `<skill-folder>/references/getting-started.md`  ← MANDATORY (after SKILL.md)
     ❌ File NOT found → HALT: "getting-started.md missing for <control-name> — cannot proceed"

  4. EXTRACT using statement / namespace declaration (from getting-started.md or other reference files)
     ❌ Namespace NOT present → HALT: "Namespace undefined for <control-name> — invalid control usage"

  5. EXTRACT + VERIFY properties, events, methods (guided by SKILL.md structure)
     ❌ Required API NOT listed → HALT: "Unverified API for <control-name> — cannot generate code"

  6. EXTRACT NuGet package name + version
     ❌ Package NOT listed → HALT: "Unknown package for <control-name> — do NOT install"
     ❌ Version mismatches Stage 2 resolved version → HALT: "Version conflict for <control-name>"

  7. IF advanced features needed: READ relevant reference files identified in SKILL.md (Step 2)
     ❌ Referenced guide NOT found → HALT: "Feature guide missing — cannot implement advanced behavior"

ALL controls pass → ✅ Validation gate cleared
ANY control fails → ❌ HALT entire code generation; report all failed controls before stopping
```

**Read Syncfusion theme guidance** (also blocking):
- Read `<skills-root>/syncfusion-winforms-theming/SKILL.md`
- ❌ File NOT found → HALT: "Theme skill file missing — cannot apply SkinManager pattern"
- Extract locked theme name (from Stage 4) + `SkinManager.ApplicationVisualTheme` / `SkinManager.LoadAssembly` pattern
- ✅ Apply theme via `SkinManager` only (`ApplicationVisualTheme` in `Program.cs` or `VisualTheme` property per form) — ❌ do NOT use `ThemeProvider.SyncTheme` (does not exist in WinForms) — ❌ do NOT apply Syncfusion theme styles manually via property overrides on individual controls

> **Skill files are the single source of truth. No code may be generated from memory, assumption, or inference.**

---

### Step 3: 🔴 READ `skill-extraction.json` + SKILL FILES BEFORE EVERY CONTROL (BLOCKING)

⛔ **This step is atomic and non-negotiable. No control may appear in generated code unless it passes both checks below.**

```
CONFIRM <project-root>/skill-extraction.json exists
  ❌ Missing → HALT: "skill-extraction.json not found — run Stage — Control Skill Extraction first"

CONFIRM skill-extraction.json → validation_status == "PASS"
  ❌ Not PASS → HALT: "Extraction not validated — fix and re-run Stage — Control Skill Extraction"

FOR EACH control that will appear in any generated file (Form, Designer, Service):

  CHECK 1 — Present in skill-extraction.json:
    FIND entry in controls[] where control == "<ControlName>"
    ❌ Not found → HALT: "<ControlName> not in skill-extraction.json
                          Do NOT invent or assume this control.
                          Either add it to control-mapping.json and re-run Stage — Control Skill Extraction,
                          or remove it from the generated file entirely."

  CHECK 2 — Re-read the referred skill file before applying the control:
    READ source file listed in skill-extraction.json → controls[].sources_read[0]
    (minimum: getting-started.md; also read feature guides if listed in advanced_features_read[])
    ❌ File not readable → HALT: "Skill reference unreadable for <ControlName> — cannot safely generate"

  ONLY AFTER both checks pass:
    USE namespace   → exact value from controls[].namespace (do NOT modify or construct)
    USE properties  → only names in controls[].valid_properties[].name
    USE events      → only names in controls[].valid_events[].name
    USE methods     → only names in controls[].valid_methods[].name
    USE nuget       → controls[].nuget_package at controls[].nuget_version
```

**No-assumption rule (applies to every generated file):**
- ❌ Do NOT include any Syncfusion control not present in `skill-extraction.json → controls[]`
- ❌ Do NOT use a property, event, or method not listed in the corresponding entry
- ❌ Do NOT construct or guess a namespace — use the exact string from `controls[].namespace`
- ❌ Do NOT add controls "because they seem useful" — only controls explicitly mapped and extracted are allowed

---

### Step 3A: 🔴 READ STAGE 4 THEMING & DESIGN SYSTEM (MANDATORY — Before Any File is Written)

Load all design decisions locked in Stage 4. Stage 5 must not apply any theme, color, spacing, typography, or event-binding pattern that was not explicitly decided there.

```
READ Stage 4 output → extract and lock the following:

  THEME:
  ✅ locked_theme_name       (e.g., "Office2019Colorful")
  ✅ theme_nuget_package     (e.g., "Syncfusion.Office2019Theme.WinForms")
  ❌ theme_name empty/null → HALT: "No theme locked in Stage 4 — cannot generate SkinManager calls"

  APPLICATION TYPE:
  ✅ application_type        (Enterprise / Consumer / LOB / Creative)
  → Determines layout density, control sizing, and interaction patterns used in code

  COLOR SYSTEM:
  ✅ custom_colors_defined   (true / false)
  ✅ IF true: extract color values from Stage 4 decisions
     → Use these as named Color constants or resource values in C# code
  ✅ IF false (Syncfusion theme provides colors): do NOT override control colors manually

  SPACING & TYPOGRAPHY:
  ✅ custom_spacing_defined   (true / false)
  ✅ IF true: extract spacing values (e.g., Padding, Margin amounts)
  ✅ IF false: use default WinForms Padding/Margin values; do NOT define custom spacing constants

  EVENT-BINDING MAP (Section 8A of Stage 4):
  ✅ Read every UI element → event handler and data-binding declaration
  ✅ Read every navigation flow → handler declaration
  ❌ Any interactive control not in the event-binding map → HALT:
     "Control '<elementId>' has no event connection defined in Stage 4
      Fix: add event handler/binding to Stage 4 Section 8A and re-confirm before generating"

  ACCESSIBILITY:
  ✅ touch_target_min_px     (default: 44)
  ✅ wcag_contrast_ratio     (default: 4.5:1)
  → Apply to all interactive controls during generation

LOCK all extracted values — they are the authoritative source for every generated file.
❌ Do NOT override or deviate from Stage 4 decisions during code generation.
```

**Data consumed from Stage 4 per deliverable:**

| Deliverable | Stage 4 Data Used |
|---|---|
| `Program.cs` | `locked_theme_name` → `SkinManager.ApplicationVisualTheme = "<name>"` + `SkinManager.LoadAssembly(...)` if required by theme |
| Custom color constants | Color values from Stage 4 → defined as `Color.FromArgb(...)` constants in a static class |
| Padding/Margin values | Spacing decisions from Stage 4 or default WinForms `Padding` |
| Every interactive control | Event-binding map → `control.EventName += Handler` wired in `InitializeComponent` or constructor |
| Every navigation trigger | Event-binding map → handler method wired in Form constructor |
| `AccessibleName` / `TabIndex` | Accessibility standards → applied to all interactive controls |

---

### Step 4: 🔴 DETECT TARGET FRAMEWORK & SDK (MANDATORY)

Read the project configuration locked in Stage 2 before generating any code.

```
READ Stage 2 output → resolved framework settings:
  target_framework   (e.g., net10.0-windows, net462)
  platform           (must be WinForms)
  dotnet_version     (e.g., .NET 8, .NET Framework 4.6.2)

VALIDATE:
  ❌ target_framework is empty or null
     → HALT: "Target SDK unknown — cannot generate code. Re-run Stage 2."
  ❌ platform ≠ WinForms
     → HALT: "Platform '<platform>' is not WinForms — Stage 5 generates WinForms only."
  ❌ target_framework contains 'winui', 'maui', 'uwp', or 'android'
     → HALT: "Non-WinForms target framework detected: <value> — cannot proceed."

LOCK for code generation:
  ✅ WinForms base namespace:   "System.Windows.Forms"
  ✅ WinForms drawing namespace: "System.Drawing"
  ✅ Syncfusion namespace:       from skill-extraction.json → controls[].namespace only
  ✅ SDK property set:           WinForms-supported properties only (validated in Step 5)
```

---

### Step 5: 🔴 NAMESPACE & PROPERTY COMPATIBILITY VALIDATION (CRITICAL)

Run for every C# file before writing it. Each check is a hard block — no silent pass.

#### 5A — Namespace Validation

```
FOR EACH using statement that will appear in the C# file:

  WinForms base namespaces:
  ✅ MUST be from: System.Windows.Forms, System.Drawing, System.ComponentModel, etc.
  ❌ Any WPF namespace (PresentationFramework, System.Windows.Controls, etc.) → HALT:
     "WPF namespace detected in WinForms file: <namespace>"

  Syncfusion namespaces:
  ✅ MUST come from: skill-extraction.json → controls[].namespace
  ❌ Namespace contains 'Xaml', 'WPF', 'Wpf', 'MAUI', 'maui', 'UWP', or 'WinUI'
     → HALT: "WPF/WinUI/MAUI namespace detected in WinForms file: <namespace>
              Mixed-framework namespaces are not allowed."

  ❌ Any namespace not in the WinForms base set AND not in skill-extraction.json
     → HALT: "Unverified namespace: <namespace> — source unknown"
```

#### 5B — Property Compatibility Validation

```
FOR EACH property used on a WinForms or Syncfusion control in the generated C#:

  ── CHECK 1: Framework-level blocked properties ──────────────────────────────
  BLOCKED PROPERTIES (not valid in WinForms at all):
  ├── RowSpacing, ColumnSpacing          → WinUI Grid only
  ├── x:Bind, {Binding}                 → WPF/WinUI XAML only
  ├── CornerRadius (as property)        → WPF Border only; WinForms uses custom painting
  ├── CommandBarFlyout, MenuFlyout      → WinUI/UWP only
  ├── DataContext                       → WPF MVVM only; WinForms uses direct binding or events
  └── Any Windows.UI.* type            → WinUI/UWP namespace

  IF property is in the blocked list above
  → HALT: "Property '<name>' is not valid in WinForms SDK (belongs to <framework>)"

  ── CHECK 2: Element-level property support (CRITICAL) ───────────────────────
  Not every WinForms control exposes every common property.
  Before setting Padding, BackColor, or Font on any control, verify
  the property is declared on that specific class in the WinForms SDK:

  PADDING — supported on:
  ✅ Control base class and all subclasses: Form, Panel, TableLayoutPanel, FlowLayoutPanel, etc.
  ❌ NOT directly effective on: GroupBox labels, TabPage interior (use Panel child instead)

  DOCKING / ANCHOR — supported on:
  ✅ All Control subclasses via Dock and Anchor properties
  ❌ Do NOT manually position controls with Left/Top when Dock or Anchor achieves the same result

  BACKCOLOR / FORECOLOR — supported on:
  ✅ All Control subclasses
  ❌ May be overridden by Syncfusion theme — do NOT set on Syncfusion controls unless explicitly verified in skill-extraction.json

  IF property is set on an element that does not declare it in the WinForms SDK
  → HALT: "Property '<name>' does not exist on '<ControlType>' in WinForms
           (namespace: System.Windows.Forms)
           Use a supported container (e.g., Panel for layout padding, TableLayoutPanel for grid spacing)"

  ── CHECK 3: Syncfusion property verification ────────────────────────────────
  IF property is on a Syncfusion control AND not in skill-extraction.json → valid_properties[].name
  → HALT: "Property '<name>' on <ControlName> not verified in skill file — do NOT use"

  ✅ All three checks pass → property is allowed
```

**Quick-fix reference for common violations:**

| Error Pattern | Cause | Fix |
|---|---|---|
| `CornerRadius` on `Panel` | WinForms has no built-in CornerRadius | Override `OnPaint` with `GraphicsPath` and `DrawRoundedRectangle`, or use a Syncfusion panel that supports it |
| `Spacing` on `FlowLayoutPanel` | No native Spacing property | Use `Margin` on child controls instead |
| `DataContext` assignment | WPF MVVM concept | Remove; wire data directly via events or `DataBindings.Add()` |
| `{Binding X}` expression | XAML binding syntax | Replace with `control.DataBindings.Add("Property", source, "SourceProperty")` |

---

## Implementation Validation Pattern (Pseudocode — MANDATORY)

Build three registries from `skill-extraction.json` before generating any code. Every gate in Steps 3 and 5B calls into these registries. A `BlockingException` thrown by any function halts generation entirely — no catch, no fallback.

### BlockingException Pattern
```
CLASS BlockingException:
  message: string
  control: string       // which control triggered the halt
  field:   string       // which field was invalid (property/event/namespace)
  valid:   string[]     // list of valid values (shown in error for quick fix)

FUNCTION halt(control, field, found, valid[]):
  THROW BlockingException {
    message: "❌ HALT: '" + found + "' is not a valid " + field + " for '" + control + "'\n"
           + "Valid " + field + "s: [" + join(valid, ", ") + "]\n"
           + "Source: skill-extraction.json → controls['" + control + "']." + field + "s",
    control: control,
    field:   field,
    valid:   valid
  }
```

### Registry 1 — Control Registry
```
// Build once from skill-extraction.json
validControls = SET of control.control for each control in controls[]
// e.g. { "SfDataGrid", "TextBoxExt", "SfButton", "SfCircularProgressBar" }

FUNCTION validateControl(controlName):
  IF controlName NOT IN validControls:
    THROW BlockingException {
      message: "❌ HALT: Control '" + controlName + "' not in skill-extraction.json\n"
             + "Do NOT invent or assume this control.\n"
             + "Fix: (A) add to control-mapping.json and re-run Stage — Control Skill Extraction\n"
             + "     (B) remove from generated files entirely",
      control: controlName,
      field:   "control",
      valid:   LIST(validControls)
    }
```

### Registry 2 — Property Registry
```
// Build once from skill-extraction.json
propertyRegistry = MAP of controlName → SET of valid_properties[].name
// e.g. propertyRegistry["SfDataGrid"] = { "DataSource", "AllowSorting", "SelectionMode" }
//      propertyRegistry["TextBoxExt"] = { "Text", "MaxLength", "WatermarkText", ... }

FUNCTION validateProperty(controlName, propertyName):
  validProps = propertyRegistry.get(controlName)
  IF validProps == null:
    validateControl(controlName)   // will halt — control not registered
  IF propertyName NOT IN validProps:
    halt(controlName, "property", propertyName, LIST(validProps))

// Example halts:
// validateProperty("SfDataGrid", "RowSpacing")
//   → "RowSpacing" not in valid_properties for "SfDataGrid"
//   → "Valid properties: [DataSource, AllowSorting, ...]"
//
// validateProperty("Button", "Text")
//   → "Button" NOT IN validControls → halt with control error first
```

### Registry 3 — Event Registry
```
// Build once from skill-extraction.json
eventRegistry = MAP of controlName → SET of valid_events[].name
// e.g. eventRegistry["SfDataGrid"] = { "SelectionChanged", "CellClick", "FilterChanged" }

FUNCTION validateEvent(controlName, eventName):
  validEvents = eventRegistry.get(controlName)
  IF validEvents == null:
    validateControl(controlName)
  IF eventName NOT IN validEvents:
    halt(controlName, "event", eventName, LIST(validEvents))
```

### Registry 4 — Namespace Registry
```
// Build once from skill-extraction.json
namespaceRegistry = MAP of controlName → namespace string
// e.g. namespaceRegistry["SfDataGrid"] = "Syncfusion.WinForms.DataGrid"

FUNCTION validateNamespace(controlName, usedNamespace):
  expected = namespaceRegistry.get(controlName)
  IF expected == null:
    validateControl(controlName)
  IF usedNamespace != expected:
    THROW BlockingException {
      message: "❌ HALT: Namespace mismatch for '" + controlName + "'\n"
             + "Expected: " + expected + "\n"
             + "Used:     " + usedNamespace + "\n"
             + "Source: skill-extraction.json → controls['" + controlName + "'].namespace",
      control: controlName,
      field:   "namespace",
      valid:   [expected]
    }
```

### Enforcement Call Points (MANDATORY — No Exceptions)
```
// ① Before writing any control instantiation:
FOR EACH control in planned C# output:
  validateControl(control.className)

// ② Before writing any property assignment on a Syncfusion control:
FOR EACH property set on a Syncfusion control:
  validateProperty(control.className, property.name)

// ③ Before wiring any event handler on a Syncfusion control:
FOR EACH event wired on a Syncfusion control:
  validateEvent(control.className, event.name)

// ④ Before writing using statements:
FOR EACH Syncfusion using statement in planned C# output:
  validateNamespace(control.className, usingStatement.value)

// Any BlockingException at ①–④ → HALT entire generation.
// Log full exception message. Do NOT continue or suppress.
```

**Concrete examples from reported errors:**
```
// Error: "Button not in skill-extraction.json"
validateControl("Button")
→ "Button" not in validControls (native MS control — not extracted)
→ FIX: Use "SfButton" (the mapped Syncfusion control) or add Button to mapping

// Error: "Property 'RowSpacing' on SfDataGrid not verified"
validateProperty("SfDataGrid", "RowSpacing")
→ "RowSpacing" not in propertyRegistry["SfDataGrid"]
→ Valid properties shown in halt message for immediate correction
→ FIX: Check skill file for correct property name (e.g., "RowHeight" vs "RowSpacing")
```

---

## Code Generation (Execute ONLY After Steps 1A, 2, 3, 4, and 5 All Pass)

**GATE CHECK (mandatory before writing the first line of any file):**
```
✅ skill-extraction.json exists + validation_status == "PASS"
✅ Every control verified against its skill file (Step 3)
✅ No control absent from skill-extraction.json appears in output
✅ Stage 4 decisions loaded: locked_theme_name, event-binding map, color/spacing strategy (Step 3A)
✅ Every interactive control has an event handler or data binding from Stage 4 Section 8A
✅ Target framework = WinForms; platform confirmed (Step 4)
✅ All using statements validated — no WPF/WinUI/MAUI namespaces (Step 5A)
✅ All properties validated against WinForms SDK + skill-extraction.json (Step 5B)
```

**Data source rules (absolute — no exceptions):**

| Data | Authoritative Source | ❌ Never From |
|---|---|---|
| C# using statement (`using ...`) | `controls[].namespace` in `skill-extraction.json` | Memory, training data, or inference |
| Properties on Syncfusion controls | `controls[].valid_properties[].name` | Assumption or prior usage patterns |
| Events on Syncfusion controls | `controls[].valid_events[].name` | Guessing based on similar controls |
| NuGet package + version | `controls[].nuget_package` + `nuget_version` | Any source not in the JSON |
| Theme name (`SkinManager.ApplicationVisualTheme`) | Stage 4 `locked_theme_name` | Hardcoded strings or defaults |
| Color values | Stage 4 color system decisions | Hardcoded hex values on controls |
| Spacing / padding values | Stage 4 spacing decisions | Hardcoded pixel values |
| Event handler wiring per control | Stage 4 event-binding map (Section 8A) | Assumptions about what handlers exist |
| Navigation wiring | Stage 4 event-binding interaction map | Inline navigation logic outside handlers |

Generate all layers together as a single cohesive feature. Never generate UI without the corresponding backend.

---

### Folder Structure (MANDATORY)

```
Forms/<Feature>/          # .cs + .Designer.cs + .resx per Form or UserControl
Services/                 # Business logic and backend services
Repositories/             # IRepository interface + in-memory implementation
Models/                   # Data models and DTOs
Resources/                # Images, icons, and static assets
```
Rules: separation of concerns strictly enforced; business logic in Services only; consistent naming `<Feature>Form.cs`, `<Feature>Service.cs`, `<Feature>Repository.cs`; no business logic in Form code or Designer files.

---

### Deliverable 1: Form / UserControl (.cs)

- All using statements from `skill-extraction.json → controls[].namespace` only (Step 3 + 5A enforced)
- Only controls and properties verified by the four registries (Step 3 + 5B enforced)
- All interactive controls: event handlers wired + `AccessibleName` set
- Responsive layout: `TableLayoutPanel` with percentage column/row sizing; `FlowLayoutPanel` for groups; no hardcoded pixel positions for fluid areas
- `InitializeComponent()` called first in constructor
- `DataBindings.Add()` or manual property assignments for data display — no WPF-style `DataContext`

**Program.cs — license + theme bootstrap:**
```csharp
using Syncfusion.Licensing;
using Syncfusion.Windows.Forms;
using Syncfusion.WinForms.Themes;  // required for Office2016/2019/HighContrast theme types
using System.Windows.Forms;

static class Program
{
    [STAThread]
    static void Main()
    {
        SyncfusionLicenseProvider.RegisterLicense(
            Environment.GetEnvironmentVariable("SYNCFUSION_LICENSE_KEY"));

        // Load the theme assembly BEFORE setting ApplicationVisualTheme.
        // Only required for Office2016, Office2019, and HighContrast themes.
        // Built-in themes (Office2007, Office2010, Office2013, Metro) do NOT need LoadAssembly().
        // ⚠ Replace Office2019Theme with the type matching the locked_theme_name from Stage 4.
        SkinManager.LoadAssembly(typeof(Office2019Theme).Assembly);

        // Apply theme application-wide BEFORE Application.Run().
        // Use the exact locked_theme_name string from Stage 4.
        SkinManager.ApplicationVisualTheme = "Office2019Colorful"; // locked theme from Stage 4

        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new LoginForm());
    }
}
```

---

### Deliverable 2: Designer File (.Designer.cs)

- Contains the `InitializeComponent()` method — generated automatically by Visual Studio and maintained manually only for Syncfusion controls that require coded initialization
- Control declarations, size/location assignments, and event wiring placed here
- No business logic in Designer file — delegate all logic to the `.cs` file
- Syncfusion controls instantiated here using exact class names from `skill-extraction.json`

**Form constructor pattern:**
```csharp
public LoginForm()
{
    InitializeComponent();
    // Wire service and data after designer initialization
    _authService = new AuthService();
    BindFormData();
}
```

---

### Deliverable 3: Service & Repository

- Service class contains all business logic for the screen's actions (e.g., `AuthService.ValidateCredentials`)
- Repository interface + in-memory implementation for data access (e.g., `IUserRepository`, `InMemoryUserRepository`)
- Navigation logic: on success → open target Form and close current; on failure → surface error via label or MessageBox
- Server-side validation independent of UI (validates inputs within the service itself)

---

### Deliverable 4: Resource File (.resx)

Generated for each Form or UserControl:
```xml
<!-- Forms/Login/LoginForm.resx -->
<root>
  <data name="lblEmail.Text" xml:space="preserve">
    <value>Email Address</value>
  </data>
  <data name="lblPassword.Text" xml:space="preserve">
    <value>Password</value>
  </data>
</root>
```
Rules: unique key per file; localization-ready strings; no hardcoded UI text in `.cs` — reference via `.resx` keys; no Syncfusion theme resources embedded here.

---

## Login Feature Example (UI + Backend — Abbreviated)

> Values below are illustrative. All namespaces, properties, and events are resolved from `skill-extraction.json` via the four registries before any file is written.

### LoginForm.cs
```csharp
using System;
using System.Windows.Forms;
using Syncfusion.WinForms.Controls;       // from skill-extraction.json
using Syncfusion.WinForms.Input;          // from skill-extraction.json

public partial class LoginForm : Form
{
    private readonly AuthService _authService;

    public LoginForm()
    {
        InitializeComponent();
        _authService = new AuthService(new InMemoryUserRepository());
        sfButtonLogin.Click += OnLoginClicked;
        txtPassword.KeyDown += OnPasswordKeyDown;
    }

    private async void OnLoginClicked(object sender, EventArgs e)
    {
        lblError.Visible = false;
        var result = await _authService.ValidateCredentialsAsync(txtEmail.Text, txtPassword.Text);
        if (result.Success)
        {
            new DashboardForm().Show();
            this.Close();
        }
        else
        {
            lblError.Text = result.ErrorMessage;
            lblError.Visible = true;
        }
    }

    private void OnPasswordKeyDown(object sender, KeyEventArgs e)
    {
        if (e.KeyCode == Keys.Enter) OnLoginClicked(sender, e);
    }
}
```

### LoginForm.Designer.cs (excerpt)
```csharp
private void InitializeComponent()
{
    this.txtEmail        = new Syncfusion.WinForms.Input.SfTextBoxExt();
    this.txtPassword     = new Syncfusion.WinForms.Input.SfTextBoxExt();
    this.sfButtonLogin   = new Syncfusion.WinForms.Controls.SfButton();
    this.lblError        = new System.Windows.Forms.Label();
    this.tableLayout     = new System.Windows.Forms.TableLayoutPanel();
    // ... property assignments and layout ...
    txtEmail.WatermarkText         = "Email";
    txtEmail.AccessibleName        = "Email address";
    txtPassword.UseSystemPasswordChar = true;
    txtPassword.AccessibleName     = "Password";
    sfButtonLogin.Text             = "Login";
    sfButtonLogin.AccessibleName   = "Login button";
    sfButtonLogin.Size             = new Size(100, 44);
}
```

### AuthService.cs
> Pattern: `ValidateCredentialsAsync` calls `IUserRepository.FindByEmail`, verifies password, returns a `Result<User>` with `Success` flag and `ErrorMessage`; navigation on success handled by the caller (Form); service has no UI dependency.

---

## Post-Generation Validation Rules (MANDATORY)

Run all checks after generating code. Fix failures before passing to Stage 6.

| # | Check | Fail Condition |
|---|---|---|
| 1 | **Control scope** | Any Syncfusion control not present in `skill-extraction.json → controls[]`; any control assumed or invented |
| 2 | **Namespace consistency** | Using statement not taken from `skill-extraction.json → controls[].namespace`; duplicate or constructed namespaces |
| 3 | **Property & event validity** | Property or event not in `skill-extraction.json → valid_properties/valid_events`; event wired with no handler body |
| 4 | **No empty handlers** | Any event handler method is empty / stub only |
| 5 | **Service wired** | Any Form or UserControl missing a service or repository assignment in constructor |
| 6 | **Data binding resolution** | Any `DataBindings.Add("X", ...)` where `X` is not a valid property of the target control |
| 7 | **Event handler resolution** | Any `control.EventName += Handler` where `Handler` is not implemented in the Form class |
| 8 | **Service completeness** | Any service method called from a Form that is not implemented in the service class |
| 9 | **Navigation wired** | Complex layout: success path does not open target Form and close current |
| 10 | **Resource file integrity** | Any string reference not defined in `.resx`; duplicate keys in same file |
| 11 | **Theme applied** | `SkinManager.ApplicationVisualTheme` not set in `Program.cs` before `Application.Run()`; `SkinManager.LoadAssembly()` not called for themes that require it (Office2016/2019/HighContrast); theme name differs from Stage 4 locked value |
| 12 | **Event-binding map coverage** | Any interactive control missing an event handler or binding declared in Stage 4 Section 8A |
| 13 | **Page/component type** | `generate_form` → must produce `Form` class; `generate_usercontrol` → must produce `UserControl` class |

**⛔ Failure on any check → fix before Stage 6. Do not proceed with broken code.**

---

## Code Generation Standards

| Standard | Rule |
|---|---|
| **APIs** | Only use properties, events, and methods listed in `skill-extraction.json → valid_properties/valid_events/valid_methods` — never invent or assume |
| **Packages** | Only use `nuget_package` + `nuget_version` from `skill-extraction.json` — never guess |
| **Controls in scope** | Only controls present in `skill-extraction.json → controls[]` may appear in any generated file |
| **Theme** | `SkinManager.ApplicationVisualTheme` set in `Program.cs` before `Application.Run()`; `SkinManager.LoadAssembly()` called first for themes requiring a separate assembly (Office2016, Office2019, HighContrast); applies globally to all Syncfusion controls |
| **Architecture** | Business logic in Service; data access in Repository; UI-only code in Form/Designer |
| **Accessibility** | `AccessibleName` + `TabIndex` on all interactive controls; min 44×44 px touch target; keyboard navigation via `TabStop = true` |
| **Data binding** | Use `DataBindings.Add()` for display fields; event handlers for user input |
| **Responsive** | `TableLayoutPanel` with `*` percentage sizing; `FlowLayoutPanel` for groups; `Dock = Fill` for main containers; never hardcode positions for fluid areas |
| **Performance** | `VirtualMode` on large grids; `async/await` for I/O; `Dispose()` called on all controls and resources; `DoubleBuffered = true` for custom-painted panels |
| **Security** | No hardcoded credentials or secrets in Form code or Designer files |

## Workflow Summary

```
Step 1:  Read control-mapping.json → extract all controls
    ↓
Step 1A: 🔴 CONTROL VALIDATION GATE (blocking)
         → NATIVE_WINFORMS / ✗ FALLBACK / ✗ NO_MATCH / score < 10 → ❌ HALT
    ↓
Step 2:  🔴 ATOMIC SKILL VALIDATION GATE (blocking)
         For each control:
         → Locate skill folder               ❌ Missing → HALT
         → Read SKILL.md (PRIMARY)           ❌ Missing → HALT
         → Read getting-started.md           ❌ Missing → HALT
         → Extract namespace                 ❌ Not found → HALT
         → Verify properties/events          ❌ Not in file → HALT
         → Validate NuGet package            ❌ Not listed / version conflict → HALT
         → Read feature guides (from SKILL.md guidance) ❌ Missing → HALT
         → Read syncfusion-winforms-theming/SKILL.md ❌ Missing → HALT
         ALL pass ✅ → gate cleared
    ↓
Step 3:  🔴 READ skill-extraction.json + skill files (blocking, per control)
         → JSON exists + PASS; every control has entry; SKILL.md + skill file re-read → ❌ else HALT
         → Only controls in JSON may appear in output
    ↓
Step 3A: 🔴 READ STAGE 4 THEMING & DESIGN SYSTEM (blocking)
         → Load: locked_theme_name, event-binding map, color/spacing strategy → ❌ empty theme → HALT
         → Every interactive control must have event binding from Stage 4 Section 8A → ❌ else HALT
    ↓
Step 4:  🔴 DETECT TARGET FRAMEWORK (blocking)
         → Confirm WinForms; reject WinUI/MAUI/UWP/WPF → ❌ HALT if mismatch
         → Lock WinForms base namespace + Syncfusion namespace source
    ↓
Step 5:  🔴 NAMESPACE & PROPERTY COMPATIBILITY (blocking)
         → 5A: All using statements = WinForms base or skill-extraction.json → ❌ HALT on WPF/WinUI/MAUI namespace
         → 5B: Blocked WinForms properties (RowSpacing, x:Bind, DataContext, etc.) → ❌ HALT if detected
    ↓
Generate: Form (.cs) + Designer (.Designer.cs) + Resource (.resx) + Service + Repository
    ↓
Post-Validation: Run all 13 checks → fix failures
    ↓
✅ Pass to Stage 6 (NuGet dependency management)
```

**User Interaction:** Optional review. AI generates without blocking confirmation.
