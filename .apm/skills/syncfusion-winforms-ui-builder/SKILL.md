---
name: syncfusion-winforms-ui-builder
description: Generates production-ready WinForms desktop applications powered by Syncfusion WinForms Controls. Orchestrates a structured workflow that handles design thinking, control picking, code generation, and validation with built-in accessibility standards and DPI-aware responsive design. Use when the user asks to create WinForms controls, build UI windows/forms, design Windows desktop interfaces, or generate code for WinForms applications.
metadata:
  author: "Syncfusion Inc"
  version: "1.0.0"
---

# Syncfusion WinForms UI Builder

## Overview

The **Syncfusion WinForms UI Builder** skill is a desktop-only Windows Forms control generator that orchestrates an AI agent through 8 stages to generate production-ready UI controls powered by Syncfusion.

## What This Skill Does

**✅ Generates (UI Layer):**
- Windows Forms `.cs` + `.Designer.cs` using Syncfusion WinForms controls, with supporting `.resx` resources
- Accessibility markup (`AccessibleName`, `AccessibleDescription`, `AccessibleRole`, tab order) (WCAG 2.1 AA)
- DPI-aware responsive layouts (`TableLayoutPanel` / `FlowLayoutPanel`, `AutoScaleMode`)
- Client-side input validation and event handling

**✅ Generates (Backend Layer):**
- Service classes with business logic (e.g., `AuthService`, `CustomerService`)
- Repository interfaces and in-memory implementations
- Navigation / form-transition logic (modal, non-modal, MDI)
- Data models, DTOs, and presenter/binding interfaces

**❌ Does NOT Generate:**
- Real database schemas, ORM migrations, or SQL
- Live third-party API integrations
- Authentication infrastructure (OAuth, JWT issuing)
- Environment secrets beyond `SYNCFUSION_LICENSE_KEY`

> **Full-feature rule:** Every generated screen must be end-to-end functional — UI wired to backend logic, validation active, and navigation working. Partial logic or stub-only output is not acceptable.

---

## Quick Start

### Prerequisites
1. WinForms project targeting .NET Framework 4.6.2+ or .NET 8+ (`<UseWindowsForms>true</UseWindowsForms>`)
2. Visual Studio 2022+ with the ".NET desktop development" workload
3. Syncfusion WinForms library (auto-installed if missing):
   ```bash
   dotnet add package Syncfusion.Core.WinForms
   ```
4. Node.js 14+ (required for Stage 3 BM25 control-mapping script)

### Examples

**Login Form**
```
User: "Create a login form with email, password, and remember me checkbox"
Output:
  ✓ Forms/LoginForm/LoginForm.cs                — event handling, navigation on success
  ✓ Forms/LoginForm/LoginForm.Designer.cs       — TextBoxExt + SfButton + CheckBoxAdv layout
  ✓ Forms/LoginForm/LoginForm.resx              — form resources
  ✓ Services/AuthService.cs                     — credential validation logic
  ✓ Models/LoginModel.cs                        — email, password, rememberMe fields
```

**Customer Data Table**
```
User: "Build a customer data table with sorting and filtering"
Output:
  ✓ Forms/CustomerTable/CustomerTable.cs        — SfDataGrid with sort/filter
  ✓ Forms/CustomerTable/CustomerTable.Designer.cs
  ✓ Services/CustomerService.cs                 — data retrieval, search logic
  ✓ Models/CustomerModel.cs                     — typed model with sample data
```

---

## Reusable Workflow Instructions

### Key Architecture

| Property | Detail |
|----------|--------|
| Design | Stateless — conversation history is sole state store |
| Stages | 8 total (6 automated, 2 user-gated) |
| User gates | Stage 3 (control confirmation) + Stage 4 (theming) |
| Auto-healing | Stages 5A, 5B, 6A, 7 auto-fix errors before passing downstream |
| Hard block | Stage 2A blocks on WinForms / other-UI-framework mismatch |
| Code scope | Both UI and backend generated together as one complete feature |

### Stage Execution Flow (Mandatory Order)

```
User Request
    ↓
[Stage 1] Intent Analysis
  → Parse query, identify form/control type & features, resolve ambiguities
  → Identify backend requirements (services, validation, navigation) implied by the screen
  → Read: references/stage-1-intent-analysis.md
  → Output: Form/control type + modifiers + target directory + backend scope summary
    ↓
[Stage 2] Project Detection
  → Auto-detect framework, .NET version, theming, project structure
  → Detect existing service/repository patterns to match generated backend style
  → Read: references/stage-2-project-detection.md
  → Output: Project config + user confirmation (with override option)
    ↓
[Stage 2A] Framework Consistency Guard  ⛔ FAIL-FAST
  → Enforce WinForms project type: `System.Windows.Forms` reference + `Form`/`UserControl`-derived classes
  → BLOCK if WPF/WinUI/MAUI/console controls or namespaces are mixed
  → Output: Confirmed namespace/`using` declarations or halt with mismatch report
    ↓
[Stage 3] Layout Analysis & Control Mapping  ⭐ USER GATE #1 + SCRIPT REQUIRED
  → Read: references/stage-3-layout-analysis.md
  → Read control-mapping.json to identify:
      • Relevant Syncfusion WinForms controls for each UI element
      • Associated skill reference files per control
  → Run BM25 script: node controls_search.cjs <project-root>/control-mapping.json
    (cd <project-root>\.apm\skills\syncfusion-winforms-ui-builder\scripts first)
  → Map backend actions to each control (e.g., SfButton[Login] → AuthService.ValidateAsync)
  → Output: control-mapping.json + Syncfusion control map + backend action map + user confirmation
    ↓
[Stage 4] Theming & Design System  ⭐ USER GATE #2
  → Read: references/stage-4-theming-and-design-system.md
  → Lock: Syncfusion WinForms visual style (Office2019 / Material / FluentLight / FluentDark), color table, 4px DIP grid spacing, 1.25 type ratio
  → Select: SfSkinManager visual style + accent color
  → Output: Design tokens confirmed; locked before any code generation
    ↓
[Stage — Control Skill Extraction] Control Skill Extraction  🔴 BLOCKING PRE-REQUISITE — Before Code Generation
  → Validate: ALL controls in control-mapping.json have validation='✓ VERIFIED' (score > 10)
  → For each control: Read skill file → Extract namespace, NuGet package, properties, events
  → Persist to: <project-root>/skill-extraction.json with validation_status="PASS"
  → Halt if: Skill file missing OR namespace/package/properties incomplete
  → Output: skill-extraction.json (pre-validated control metadata for Stage 5 code generation)
    ↓
[Stage 5B-1] Type Safety Enforcement  🔒 CRITICAL — runs BEFORE designer code generation
  → Validate BackColor/ForeColor → must be `System.Drawing.Color` (named or `Color.FromArgb`)
  → Validate Margin/Padding → must be `Padding(left, top, right, bottom)`
  → Validate Font → must be `Font` with size as float > 0
  → Validate Size/Width/Height → must be int > 0, or governed by Dock/Anchor/AutoSize
  → Validate Colors → must be `Color.FromArgb(a,r,g,b)` or a named `Color`
  → Auto-fix: Replace invalid values with safe defaults
    ↓
[Stage 5B-2] Resource Validation  🔒 CRITICAL — runs BEFORE designer code generation
  → Scan all `resources.GetString(...)` / `resources.GetObject(...)` references in `.Designer.cs`
  → Verify each key exists in the corresponding `.resx` file
  → Auto-inject missing keys with fallback values (e.g., empty string, default icon)
  → Check for duplicate resource keys
    ↓
[Stage 5] Safe Code Generation  🔒 COMPLETE IMPLEMENTATION — UI + BACKEND
  **Prerequisite:** skill-extraction.json exists with validation_status="PASS" (from Stage — Control Skill Extraction)
  **Data source:** All namespaces/properties/events/packages from skill-extraction.json (never guessed)
  **Pre-Generation Analysis (Mandatory):**
  → Read: references/stage-5-code-generation.md
  → Read control-mapping.json FIRST → identify all Syncfusion controls, events, commands
  → Read corresponding Syncfusion skill file → extract required properties & behaviors
  → Map Designer controls ↔ control-mapping.json ↔ skill directives for alignment

  **Code Generation Principles (Strict Adherence):**
  → Generate ONLY methods, properties, events explicitly required by mapped controls
  → No generic boilerplate, utility methods, or unused stub code
  → Preserve existing codebase structure; avoid overwriting unrelated members
  → Tight alignment: every control + event + binding traces back to skill directive

  **UI Generation (Complete Implementation):**
  → `.Designer.cs`: all Syncfusion `using` namespaces, all mapped controls, full `InitializeComponent()` wiring
  → `.cs`: full event handler implementations, data binding via `BindingSource`, all using statements
  → `.resx`: all referenced resource entries (strings, icons, images)

  **Backend Generation (Skill-Driven):**
  → Service classes: implement only business logic declared in control map + skill file
  → Repository + in-memory data: only if skill directives require data access
  → Navigation: open/close/show forms (modal, non-modal, MDI child) per skill success/failure paths
  → Validation: required fields + format checks per skill specification
  → Error propagation: surface errors from service → form → `MessageBoxAdv`/inline error display

  **Functional Completeness & Safety:**
  → Every control must be fully wired to backend logic (no dead buttons or unbound fields)
  → Output: complete, compilable, tested code — zero missing implementations
  → Constraints: no overwritten code, no unused members, minimal-but-full functionality
    ↓
[Stage 6] NuGet Dependency Management
  → Read: references/stage-6-dependencies.md
  → Detect required Syncfusion WinForms + theme NuGet packages
  → Verify all Designer `using` namespaces have corresponding packages
  → Output: dotnet add command(s) or auto-install
    ↓
[Stage 7] Design-Time Compile Validation
  → Read: references/stage-7-validation.md + assets/validation-rules.md
  → Simulate `InitializeComponent()` compilation / designer instantiation
  → Auto-fix: invalid property assignments, missing namespaces, type mismatches
  → Loop until compile succeeds (max 5 iterations)
  → Abort on: circular reference, unsupported control type, licensing error
  → Output: PASS ✓ or FAIL ✗
    ↓
[Stage 8] Code Insertion
  → Insert all validated files (UI + backend) into project
  → Update project references, verify build
  → STOP on errors; report all inserted file paths on success
    ↓
✓ Complete
```

### Stage Gate Summary

| Stage | Interaction | Behavior |
|-------|-------------|----------|
| 1–2 | Auto-detect | Auto-flow |
| 2A | Framework check | ⛔ BLOCK on mismatch |
| 3 | Control + backend action confirmation | ⭐ User gate |
| 4 | Theming confirmation | ⭐ User gate |
| **5A** | **Skill extraction + validation** | **⛔ BLOCK if file missing or extraction fails** |
| 5B-1–5B-2 | Property + resource validation | Auto-fix |
| 5 | UI + backend code generation + skill alignment | Auto-flow (only if Stage — Control Skill Extraction passed) |
| 6 | Dependency validation gate | ⛔ BLOCK if skill-extraction.json missing |
| 6–6A | Dependency + binding + service validation | Auto-fix / Fail gate |
| 7 | Design-time compile validation | Auto-fix loop |
| 8 | Code insertion + build verification | Auto-flow |

---

## Agent Instructions

1. **Validate scope**: Confirm request is for a WinForms screen. Generate both UI and backend together — never UI alone.
2. **Stage — Control Skill Extraction is mandatory**: Before ANY code generation, execute Stage — Control Skill Extraction (Control Skill Extraction). Halt if skill-extraction.json cannot be created or validated.
3. **Read control-mapping.json before Stage 5**: Identify which controls appear, which events fire, and which backend actions are implied. Generate only what those controls need.
4. **Follow stage order strictly**: Never skip or reorder stages. Stage — Control Skill Extraction must complete before Stage 5 (code generation).
5. **Load references on-demand**: Read each stage's `.md` file immediately before executing that stage.
6. **Stateless execution**: Read all prior decisions from conversation context at each stage start.
7. **License key handling**:
   - Check for `SYNCFUSION_LICENSE_KEY` in `app.config`/`appsettings.json` or environment
   - If missing, prompt: *"Get a free Community License at https://www.syncfusion.com/account/manage-trials"*
   - If provided, inject into config + call `Syncfusion.Licensing.SyncfusionLicenseProvider.RegisterLicense()` in `Program.cs`
   - If skipped, warn that an evaluation watermark will appear

---

## Code Generation Rules (Mandatory)

### ⛔ Stage — Control Skill Extraction Prerequisite (CRITICAL)
- **Before ANY code generation in Stage 5**: Execute Stage — Control Skill Extraction
- Confirm `skill-extraction.json` exists with `validation_status: "PASS"`
- **All code generation must use data from `skill-extraction.json`** (namespaces, properties, events, packages)
- ❌ Never guess or assume APIs; ❌ Never infer package names
- ✅ All control metadata must be pre-extracted and verified

### Control-Mapping-Driven Generation
- **Before generating any code**, read `control-mapping.json` to identify:
  - Every Syncfusion control required by the screen
  - The associated skill reference file for each control
  - The backend action (service method) mapped to each interactive control
- Generate **only** the methods, properties, and event handlers that are directly required by the mapped controls in the Designer
- Do not add unrelated utility methods, extra services, or placeholder code not tied to a mapped control

### Minimal-but-Complete Rule
Every generated file must be:
- **Context-aware**: driven by the specific skill and its control map, not a generic template
- **Feature-complete**: all controls in the Designer are fully wired to logic (no dead buttons or unbound fields)
- **Minimal**: no boilerplate beyond what the mapped controls require

### Screen Completeness Checklist
Before finalizing Stage 5 output, verify each screen satisfies:

| Requirement | Example (Login Screen) |
|-------------|------------------------|
| Input handling | Email → `TextBoxExt`, Password → `TextBoxExt` (`UseSystemPasswordChar` = true) bound to model properties |
| Client-side validation | Required field check, email regex, password min-length |
| Event handling | Login `SfButton.Click` → `AuthService.ValidateCredentials` |
| Backend logic | `AuthService.ValidateCredentials(email, password)` returns success/failure |
| Navigation on success | Opens `DashboardForm`, closes/hides `LoginForm` |
| Error display | Failure message shown via `MessageBoxAdv` or inline error label |
| Server-side validation | `AuthService` rejects empty or malformed inputs independently of UI |

---

## Boundary Rules (Critical)

| Rule | Detail |
|------|--------|
| UI + backend together | Always generate both layers as one complete feature; never UI-only |
| Syncfusion controls only | Use only Syncfusion WinForms controls; never native MS controls (`TextBox` → `TextBoxExt`, `Button` → `SfButton`, `ComboBox` → `ComboBoxAdv`, `DataGridView` → `SfDataGrid`, `MessageBox` → `MessageBoxAdv`, `ProgressBar` → `ProgressBarAdv`, `TreeView` → `TreeViewAdv`, `TabControl` → `TabControlAdv`, `CheckBox` → `CheckBoxAdv`, `RadioButton` → `RadioButtonAdv`, `DateTimePicker` → `DateTimePickerAdv`, `NumericUpDown` → `UpDownExt`) |
| Skill file + control-mapping.json first | **Mandatory pre-generation:** Read the Syncfusion skill file to extract required properties, behaviors, and constraints BEFORE any code generation in Stage 5. Cross-reference with control-mapping.json to ensure all controls, events, and backend actions align with skill directives. |
| Dependency rule: Skill files ONLY | **Before adding ANY NuGet package:** (1) Read skill file, (2) Extract exact package name, (3) Use latest stable version, (4) Never assume/infer names (e.g., ❌ appending `.WinForms` to a control name as a guess). Only packages documented in skill files or official Syncfusion WinForms docs are permitted. Reject all others. |
| Mock data only | Use in-memory repositories with sample data; no live DB or real API calls |
| No secrets | Only `SYNCFUSION_LICENSE_KEY` when user explicitly provides it |
| Minimal-but-complete | Generate exactly what the mapped controls need — no extra boilerplate |
| Compilation guaranteed | Stage 6A must pass ALL checks (UI + backend) before any file is inserted |
| Framework purity | Never mix WinForms and WPF/WinUI/MAUI controls or namespaces in the same project |

---

## DO ✅ / DON'T ❌ Guidelines

**DO:**
- ✅ Read the Syncfusion skill file FIRST to identify required properties, behaviors, and constraints
- ✅ Read `control-mapping.json` SECOND to identify mapped controls, events, and backend actions
- ✅ Cross-reference skill file + control-mapping.json + Designer code for tight alignment before any code generation
- ✅ Generate both UI and backend in Stage 5 as a single cohesive output
- ✅ Implement full event handler logic (login → validate → navigate), never stubs
- ✅ Wire every control in the Designer to a model property, service call, or event handler
- ✅ Use `MessageBoxAdv` for dialogs, `TextBoxExt` for text inputs, `SfButton` for buttons
- ✅ Use `SfDataGrid` for all tabular data; never native `DataGridView`
- ✅ Lock design tokens (theme, colors, spacing) in Stage 4 before generating any code in Stage 5
- ✅ Run the BM25 `controls_search.cjs` script in Stage 3
- ✅ Apply `AccessibleName` / `AccessibleDescription` / `AccessibleRole` for all interactive controls
- ✅ Use `SfSkinManager` for theme application
- ✅ Use relative layouts (`TableLayoutPanel`/`FlowLayoutPanel`); never hardcode pixel sizes for responsive areas

**DON'T:**
- ❌ Generate code without reading the Syncfusion skill file first
- ❌ Skip reading both skill file AND `control-mapping.json` before Stage 5
- ❌ Generate UI without the corresponding backend service and navigation logic
- ❌ Use native MS controls (`TextBox`, `Button`, `ComboBox`, `DataGridView`, `MessageBox`, etc.)
- ❌ Generate code not directly required by mapped controls or skill directives (no unused helpers or empty stubs)
- ❌ Skip Stage 2A framework guard
- ❌ Generate Designer code before Stage — Control Skill Extraction/5B validation passes
- ❌ Insert code before Stage 6A compilation gate passes
- ❌ Use `dynamic` types without explicit justification
- ❌ Hardcode secrets in the Designer or code-behind

---

## Error Handling & Validation

**Per-stage recovery:**
1. Retry once with same approach
2. If retry fails → apply workaround or skip to next stage
3. Notify user with error message
4. Offer: *"Would you like to go back to Stage 3 and choose a different layout?"*
5. Reference `references/Build.md` for common errors

**Compilation fail gate (Stage 6A):**
- Missing event handler → HALT, regenerate Stage 5
- Missing bound property → HALT, regenerate Stage 5
- Missing service method called from form → HALT, regenerate Stage 5
- Missing `using` statement → HALT, regenerate Stage 5

**Design-time validation loop (Stage 7):**
- Max 5 auto-fix iterations
- Abort on: circular reference, unsupported control, licensing error

---

## ⛔ MANDATORY ERROR HANDLING PROTOCOL

**If ANY build error or validation failure occurs:**

### Issue 1 & 3: Theme / Resource Errors
**Errors:** `Property 'BorderStyle' does not accept value...` or `resources.GetObject(...) returned null`
- ✅ **Fix:** Stage 4 + Stage 7
- ✅ Apply Syncfusion theme ONLY via `SfSkinManager.SetVisualStyle(this, VisualTheme.<LockedThemeName>)` immediately after `InitializeComponent()` in the form constructor
- ✅ Apply the same locked visual style consistently to every generated form (via a shared base `Form` or a `Program.cs` helper)
- ❌ NEVER hand-edit `.resx` XML directly outside the Designer-generation step
- ✅ Custom resources ONLY: form-scoped `.resx` entries for strings, icons, and images

### Issue 2: Missing Syncfusion Control
**Error:** `The type or namespace name 'SfDataGrid' could not be found...`
- ✅ **Fix:** Stage 6 (Dependencies)
- ✅ Read `control-mapping.json` → identify mapped control
- ✅ Read skill file (`syncfusion-winforms-[control]/SKILL.md`) → extract exact NuGet package name
- ✅ Install package: latest stable version matching Stage 2 version
- ❌ NEVER assume or infer package names

### Critical Rule: ALWAYS Read Skill Files First
**If build fails OR control error occurs:**
1. ✅ Refer back to control's skill file FIRST
2. ✅ Verify: API names, namespace declarations, NuGet package version
3. ❌ DO NOT fallback automatically to Microsoft/WinForms default controls (e.g., `TextBox`, `ComboBox`, `DataGridView`)
4. ⛔ HALT if skill file missing or ambiguous — no silent corrections
5. ✅ Retry build with skill-verified changes before next stage

---

## Resource Loading Strategy (Mandatory)

Load files **on-demand only** — never preload all references.

| When | Load |
|------|------|
| Before Stage 1 | `references/stage-1-intent-analysis.md` |
| Before Stage 2 | `references/stage-2-project-detection.md` |
| Before Stage 3 | `references/stage-3-layout-analysis.md` |
| Before Stage 4 | `references/stage-4-theming-and-design-system.md` (+ `references/syncfusion-themes.md` for visual style reference) |
| **Before Stage — Control Skill Extraction** | **`control-mapping.json` (validate ALL controls are ✓ VERIFIED)**<br/>**For each control: read `<skills-root>/syncfusion-winforms-<control>/references/getting-started.md`** |
| After Stage — Control Skill Extraction | ✅ Confirm `skill-extraction.json` exists with `validation_status: "PASS"` before proceeding |
| Before Stage 5 | `references/stage-5-code-generation.md` + `skill-extraction.json` (pre-extracted data source) |
| Before Stage 6 | `references/stage-6-dependencies.md` + verify `skill-extraction.json` present |
| Before Stage 7 | `references/stage-7-validation.md` + `assets/validation-rules.md` |
| Before Stage 8 | `references/winforms-dotnet-standards.md` |
| On error | `references/Build.md` |

**Initial load:** SKILL.md only. Full spec available on-demand.

**Critical:** Stage — Control Skill Extraction must complete successfully (producing `skill-extraction.json`) before Stage 5 code generation can begin. This is NOT optional.

---

## Code Generation Standards

### Accessibility (WCAG 2.1 AA)
- `AccessibleName` and `AccessibleDescription` set on all Syncfusion controls, with correct `AccessibleRole`
- Keyboard navigation: correct `TabIndex`/`TabStop` order, focus management
- Color contrast ≥ 4.5:1; visible focus indicators on `SfButton` and `TextBoxExt`

### Responsive & DPI
- DPI-aware sizing using `AutoScaleMode.Dpi` and `AutoScaleDimensions`
- `TableLayoutPanel`/`FlowLayoutPanel` layouts; no fixed pixel widths for fluid areas
- Touch targets ≥ 44×44 pixels at 96 DPI

### Security
- Input validation in the form and service layer; no hardcoded secrets in Designer or code-behind
- Secure event-handler-to-service call patterns; no code injection vectors

### Performance
- `SfDataGrid` virtualization enabled for large datasets
- Lazy loading for heavy resources; efficient `BindingSource`/`BindingList<T>` binding

### C# Quality
- Full type coverage (no unexplained `dynamic`)
- `INotifyPropertyChanged` on bound models with correct property-change notifications
- XML doc comments on all public service interfaces and models
- Consistent event-handler-to-service delegation pattern for all control actions

---

## Supported Use Cases

| Request Type | Key Syncfusion Controls | Backend Generated |
|---|---|---|
| Login form | `TextBoxExt`, `SfButton`, `CheckBoxAdv`, `MessageBoxAdv` | `AuthService`, `LoginModel` |
| Registration wizard | `TextBoxExt`, `ComboBoxAdv`, `SfButton` | `UserRegistrationService`, step validators |
| Customer data table | `SfDataGrid` (sort, filter, paginate) | `CustomerService`, `ICustomerRepository` |
| Dashboard | `ChartControl`, `SfDataGrid`, `TabControlAdv`, `ProgressBarAdv` | Aggregation services, summary DTOs |
| Kanban board | `SfKanban` with swimlanes | `TaskService`, status-transition logic |
| Data analysis tool | `ChartControl`, `SfDataGrid`, `DateTimePickerAdv` | Filter/query service, export logic |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Project type not detected | Ensure `.csproj` has `<UseWindowsForms>true</UseWindowsForms>` and targets a Windows TFM |
| Syncfusion watermark appears | Add license key during Stage 2 prompt |
| Build fails after insertion | See `references/Build.md` |
| Control not rendering | Verify `using` namespace declarations match installed NuGet packages |
| Designer compile error loops | Check Stage 7 abort conditions; report control type to user |
| Missing binding at runtime | Re-run Stage 6A validation; ensure `BindingSource.DataSource` is set |
| Service method not found | Confirm Stage 5 backend generation included the service; re-run Stage 6A |
| Navigation not working | Verify success handler in event method opens target `Form` and closes/hides current |

**Full guide:** `references/Build.md`

## Additional Resources

### Quick Reference by Use Case

| Need | Reference File |
|------|-----------------|
| Understanding workflow | This SKILL.md file |
| How Stage X works | `references/stage-X-*.md` |
| Validation rules | `assets/validation-rules.md` |
| Accessibility/security | `references/winforms-dotnet-standards.md` |

## Support

For issues or questions:
1. Verify your project meets prerequisites (.NET Framework 4.6.2+ or .NET 8+, Visual Studio 2022+)
2. Ensure Syncfusion license is valid and registered
3. Review generated code compliance report for warnings
