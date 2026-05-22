---
name: syncfusion-winforms-ui-builder
description: Generates production-ready Windows desktop applications powered by Syncfusion WinForms Controls. Orchestrates 8-stage workflow that handles design thinking, control picking, code generation, and validation with built-in accessibility standards and responsive WinForms design. Use when the user asks to create desktop forms, build UI windows, design Windows interfaces, or generate desktop code for .NET applications.
metadata:
  author: "Syncfusion Inc"
  version: "1.0.0"
---

# Syncfusion WinForms UI Builder

## Overview

The **Syncfusion WinForms UI Builder** skill is a desktop application control generator that orchestrates an AI agent through 8 stages to generate production-ready UI controls powered by Syncfusion for Windows Forms.

This spec is **Agent Skills Specification compliant** with one-level-deep file references for optimal context loading by AI agents.

---

## ⚠️ CRITICAL: Syncfusion Package Verification Requirements

**This skill uses Syncfusion WinForms controls exclusively. When developing applications:**

1. **Use Syncfusion WinForms controls FIRST** where functionality is available
2. **ONLY fall back to standard Microsoft WinForms controls** if Syncfusion does NOT provide the required functionality
3. **NEVER assume or infer Syncfusion NuGet package names** by following patterns (e.g., ❌ appending `.WinForms` to a control name)
4. **ALWAYS verify package names** are EXPLICITLY documented in:
   - The corresponding control's SKILL.md file, OR
   - Official Syncfusion documentation at [Syncfusion WinForms Controls](https://help.syncfusion.com/windowsforms/overview)
5. **Use ONLY officially documented Syncfusion NuGet packages** from control skill files or official Syncfusion resources — assumed or inferred package names are NOT acceptable

**Example of INCORRECT assumptions:**
- ❌ `DataGrid.WinForms` (inferred from control name)
- ❌ `Syncfusion.DataGrid.WinForms` (assumed pattern)
- ✅ `Syncfusion.GridControl.WinForms` (explicitly documented in official resources)

**Verification Workflow:**
- Stage 3 generates control mappings using REFERENCE LABELS (e.g., `syncfusion-winforms-datagrid`)
- Stage 5 must consult corresponding control skill SKILL.md or official Syncfusion documentation for exact namespaces
- Stage 6 MUST convert reference labels to official NuGet package names by verifying against:
  - Control skill file (`.codestudio/skills/syncfusion-winforms-<name>/SKILL.md`) if available
  - [Official Syncfusion documentation](https://help.syncfusion.com/windowsforms/overview) 
  - **REJECT any package that is not explicitly documented in one of these sources**

---

## What This Skill Does

**✅ Generates:**
- WinForms classes (`.cs`) extending `Form` and custom controls
- UI designer files (`.Designer.cs`) with Initializecontrol()
- TypeScript-style C# interfaces and strongly-typed event handlers
- Syncfusion WinForms control integration with correct properties and events
- Client-side form validation logic
- Accessibility compliance (keyboard navigation, screen reader support)
- Responsive layouts using TableLayoutPanel and FlowLayoutPanel
- control class organization with separation of concerns

**❌ Does NOT Generate:**
- Backend code (Data Access Layer, Business Logic Layer)
- Database schemas or Entity Framework models
- Authentication/authorization logic (unless WinForms UI only)
- Application configuration files
- Dependency Injection setup
- Connection strings or infrastructure config

## Supported Controls

- **Forms & Input**: Login form, registration wizard, TextEdit, MaskedEdit, ComboBox, NumericUpDown, DateEdit, CheckEdit, RadioButton
- **Data Display**: DataGridView with Syncfusion DataGrid, TreeView, ListView, Report Viewer
- **Navigation**: Ribbon bar, Navigation drawer, Tab control, Menu bar, Status bar
- **Common Patterns**: Message boxes, dialogs, tooltips, progress bar, loading indicators
- **Page Templates**: Dashboard windows, data entry forms, MDI applications, document editors

## Skill Structure

```
syncfusion-winforms-ui-builder/
├── SKILL.md                                     # This file (Agent Skills spec compliant)
├── references/                                  # One-level-deep stage guides + support docs
│   ├── stage-1-intent-analysis.md              # Stage 1: Intent Analysis
│   ├── stage-2-project-detection.md            # Stage 2: Project Detection
│   ├── stage-3-layout-analysis.md              # Stage 3: Layout Analysis & control Mapping
│   ├── stage-4-theming-and-design-system.md    # Stage 4: Theming & Design System Selection
│   ├── stage-5-code-generation.md              # Stage 5: Code Generation
│   ├── stage-6-dependencies.md                 # Stage 6: Dependencies
│   ├── stage-7-validation.md                   # Stage 7: Validation
│   ├── winforms-dotnet-standards.md            # .NET + WinForms standards + accessibility rules
│   └── syncfusion-themes.md                    # Syncfusion WinForms theming reference
└── assets/                                     # Static resources
    └── validation-rules.md                     # Validation checklist for Stage 6

scripts/
├── controls.csv                                # Control metadata for BM25 search (reference labels)
└── controls_search.cjs                          # BM25 semantic search for control mapping
```

**Note:** The `Skill Name` column in `controls.csv` contains REFERENCE LABELS for semantic search purposes only. They are NOT official Syncfusion NuGet package names. See Stage 6 documentation for converting reference labels to official packages.

## Quick Start

### Prerequisites

1. **Active .NET project** (.NET Framework 4.6.2+, .NET 8+, .NET 9+, or .NET 10+)
2. **Visual Studio 2022+** or Visual Studio Code with C# extension
3. **.NET SDK 8.0+** installed and in PATH
4. **Syncfusion WinForms License** (Community or Commercial)
   - Obtain at: [Syncfusion Community License](https://www.syncfusion.com/products/communitylicense) or commercial license
   - For setup, refer to: [Syncfusion License Registration Guide](https://help.syncfusion.com/windowsforms/licensing/how-to-register-in-an-application)
5. **Syncfusion WinForms controls library** (installed via NuGet):
   ```bash
   dotnet add package Syncfusion.Core.WinForms
   ```
   Or add directly to `.csproj` file. See [NuGet Package Reference](https://www.nuget.org/packages?q=syncfusion+winforms) for complete list of available packages.

### Basic Usage

**Example 1: Generate a Login Form**

```
User: "Create a login form with email, password, and remember me checkbox"

Skill executes:
  → Stage 1: Identifies login form dialog type
  → Stage 2: Detects project structure (.NET, C#, Framework version, etc.)
  → Stage 3-4: AI creates optimal control-mapping.json → maps to Syncfusion WinForms controls
  → Stage 5: Generates LoginForm.cs and LoginForm.Designer.cs with validation
  → Stage 6: Installs/updates NuGet dependencies
  → Stage 7: Validates accessibility and .NET standards compliance
  → Stage 8: Inserts code into project

Output:
  ✓ Forms/LoginForm.cs
  ✓ Forms/LoginForm.Designer.cs
  ✓ Forms/LoginForm.resx
```

**Example 2: Generate a Data Table**

```
User: "Build a customer data table with sorting and filtering"

Output:
  ✓ Forms/CustomerGridForm.cs (with Syncfusion DataGridViewEx or DataGridView)
  ✓ Forms/CustomerGridForm.Designer.cs
  ✓ Mock data included (List<Customer> with sample data)
  ✓ Accessibility keyboard navigation
  ✓ Column sorting and filtering enabled
```

## How It Works: 8-Stage AI Orchestration (Stateless)

The skill orchestrates **8 stages of pure AI reasoning** with **two user decision points**.

**Key Architecture:**
- **Stateless design**: Conversation history maintains state
- **Pure AI reasoning**: Each stage reads guidance docs, analyzes context, makes decisions
- **2 user decision gates**: Stage 3 (control confirmation) + Stage 7 (validation result)
- **6 fully automated stages**: 1, 2, 4, 5, 6, and final code insertion
- **Dedicated theming stage**: Stage 4 locks design system before code generation

```
User Request
    ↓
[Stage 1: Intent Analysis] 
  AI reads query → identifies form/control type & features
    ↓
[Stage 2: Project Detection]
  AI scans project → detect .NET framework, language, layout strategy, preferences
    ↓
[Stage 3: Layout Analysis & Control Mapping] ⭐ MANDATORY SCRIPT EXECUTION
  1. AI analyzes requirements → creates control-mapping.json (saved to workspace root, OUTSIDE project)
  2. AI runs controls_search.cjs script (BM25 control semantic search)
  3. Script maps elements to Syncfusion WinForms controls automatically
  4. Results captured in chat for Stage 4-5 processing
    ↓
[Stage 4: Theming & Design System]
  AI locks .NET theme → Syncfusion theme mapping
  AI selects color system (RGB, brand colors)
  AI confirms spacing/typography scale (DPI-aware sizing)
  Design system decisions locked before code generation
    ↓
[Stage 5: Code Generation]
  AI generates .cs classes, .Designer.cs UI code, C# interfaces
  Uses theming decisions from Stage 4
  With accessibility + keyboard navigation built-in
    ↓
[Stage 6: Dependencies]
  AI detects required NuGet packages (Syncfusion + frameworks)
  Presents dotnet add package command or runs it
    ↓
[Stage 7: Validation] ⭐ USER DECISION #2
  AI validates .NET standards + accessibility + security + performance + theming
  Binary result: PASS ✓ or FAIL ✗
  User confirms or overrides
    ↓
[Stage 8: Code Insertion]
  AI inserts files into project
  Updates project file references, verifies build
    ↓
✓ Complete
```

**Stage Descriptions:**

- **Stage 1 (Intent Analysis)**: Parse user query, identify form/control type and features. Read: `references/stage-1-intent-analysis.md`
- **Stage 2 (Project Detection)**: Auto-detect .NET framework, language, layout strategy, form directory. Read: `references/stage-2-project-detection.md`
- **Stage 3 (Layout Analysis & Control Mapping)**: AI analyzes requirements, creates control-mapping.json, runs controls_search.cjs script to map elements to Syncfusion WinForms controls automatically using BM25 semantic search. Script results captured in chat. Read: `references/stage-3-layout-analysis.md`
- **Stage 4 (Theming & Design System)**: Lock .NET theme, Syncfusion theme, color system (RGB), DPI-aware sizing, typography scale. Read: `references/stage-4-theming-and-design-system.md`
- **Stage 5 (Code Generation)**: Generate .cs and .Designer.cs files, C# interfaces using theming from Stage 4 applied + accessibility + keyboard navigation. Read: `references/stage-5-code-generation.md`
- **Stage 6 (Dependencies)**: Detect NuGet packages (Syncfusion + frameworks), resolve conflicts, prepare install command. Read: `references/stage-7-dependencies.md`
- **Stage 7 (Validation)**: Validate .NET standards, accessibility, security, performance, theming integration. Binary pass/fail. Read: `references/stage-6-validation.md` + `assets/validation-rules.md`
- **Stage 8 (Code Insertion)**: AI inserts files, updates project file references, verifies build succeeds.

**User Interaction Summary:**

| Stage | Interaction |
|-------|-------------|
| 1 | None (AI analyzes) |
| 2 | Confirm auto-detected settings |
| 3 | ⭐ MANDATORY: Create control-mapping.json + Run controls_search.cjs script |
| 4 | Confirm theming decisions (.NET theme, colors, DPI sizing, typography) |
| 5 | None (AI generates) |
| 6 | Optional (confirm dotnet add package) |
| 7 | ⭐ Confirm validation result (pass/fail/override) |
| 8 | None (AI executes) |

**Total user decision gates: 2** (Stage 3: script execution, Stage 7: validation). Rest fully automated with AI reasoning + guidance docs.

## Scripts & Tools

### Stage 3: ControlMapper Script (`controls_search.cjs`)

**Purpose:** Automatically map UI elements to Syncfusion WinForms controls using BM25 semantic search algorithm.

**Location:** `scripts/controls_search.cjs`

**What It Does:**
- Reads `control-mapping.json` with element descriptions and `type_hint` values
- Searches Syncfusion WinForms control keywords using BM25 ranking algorithm
- Matches each element to the best-fit Syncfusion control
- Falls back to native System.Windows.Forms for unmatched elements
- Returns control mapping with BM25 scores (0-100 range)

**Data Source:**
- `scripts/controls.csv` - 100+ Syncfusion WinForms controls with keywords (auto-loaded)

**Execution Syntax:**

```powershell
# Navigate to scripts directory
cd <project-root>\.apm\skills\syncfusion-winforms-ui-builder\scripts

# Run with absolute path to control-mapping.json
node controls_search.cjs <project-root>\control-mapping.json
```

**Example (Windows):**

```powershell
cd d:\MyWinFormsApp\.apm\skills\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs d:\MyWinFormsApp\control-mapping.json
```

**Prerequisites:**
- Node.js 14+ installed on system
- `control-mapping.json` must exist at specified path
- `controls.csv` must be in same directory as script

**Output:**
- JSON printed to console with mapped controls + BM25 scores
- Copy output into chat for Stage 4 (theming) and Stage 5 (code generation)
- Do NOT save output to file (keep in conversation only)

**Error Handling:**
- If `control-mapping.json` not found → Error message with full path
- If `controls.csv` not found → Error message
- If JSON parse error → Error with line number and context

**BM25 Algorithm Details:**
- **Tokenization:** Splits keywords on whitespace and commas
- **Term Frequency (TF):** Counts occurrences in each control
- **Inverse Document Frequency (IDF):** Ranks rare keywords higher
- **Saturation (k1=1.5):** Diminishing returns on term frequency
- **Length Normalization (b=0.75):** Adjusts for control keyword length

---

## Agent Instructions

### When User Requests UI control Generation

1. **Validate scope**: Confirm request is for WinForms UI controls (not backend/API/services)
2. **Load guidance**: Read `stage-1-intent-analysis.md` to understand Stage 1
3. **Execute 8-stage flow**: Follow the orchestration flow shown above
4. **Progressive disclosure**: Load stage guides on-demand; load support references only when needed
5. **Maintain conversation history**: Each stage reads previous decisions from conversation context (stateless)

### Stage Execution & Reference Loading

**Stage 1: Intent Analysis**
- Read: `references/stage-1-intent-analysis.md`
- Task: Parse user query, identify control type, resolve ambiguities
- Output: control type + modifiers + target directory

**Stage 2: Project Detection**
- Read: `references/stage-2-project-detection.md`
- Task: Auto-detect .NET framework, language (C# version), layout strategy, formatting rules
- Output: Project configuration + user confirmation

**Stage 3: Layout Analysis & Control Mapping** ⭐ MANDATORY SCRIPT EXECUTION
- Read: `references/stage-3-layout-analysis.md`
- Task: Analyze user requirements → create optimal `control-mapping.json` (at workspace root) → **RUN controls_search.cjs script** → map to Syncfusion controls
- Script: `scripts/controls_search.cjs` (uses BM25 algorithm for semantic control matching)
- Execution: `node controls_search.cjs <workspace-root>/control-mapping.json`
- Output: `control-mapping.json` (file at workspace root) + Control mapping results (chat context) + Summary table

**Stage 4: Theming & Design System**
- Read: `references/stage-4-theming-and-design-system.md`
- Task: Lock .NET theme, Syncfusion theme, color system (RGB), DPI-aware spacing, typography scale, responsive breakpoints
- Output: Design system decisions confirmed and ready for code generation

**Stage 5: Code Generation**
- Read: `references/stage-5-code-generation.md`
- Task: Generate .cs, .Designer.cs, C# interfaces using theming from Stage 4
- Ensure: Accessibility (keyboard navigation, screen reader support), DPI awareness, MVVM patterns applied
- Output: Generated files ready for review

**Stage 6: Dependencies**
- Read: `references/stage-7-dependencies.md`
- Task: Detect required NuGet packages (Syncfusion + .NET frameworks), resolve version conflicts
- Output: dotnet add package command or auto-install

**Stage 7: Validation** ⭐ USER DECISION #2
- Read: `references/stage-6-validation.md` + `assets/validation-rules.md` + `references/winforms-dotnet-standards.md`
- Task: Validate against .NET standards, accessibility, security, performance, theming integration standards
- Auto-apply fixes where possible
- Output: Binary result (PASS ✓ or FAIL ✗) → user confirms or overrides

**Stage 8: Code Insertion**
- Task: Insert generated files into project, update imports, verify build
- Output: Success report with file paths



### Boundary Rules (CRITICAL)

**AI agents executing this skill MUST:**

1. **UI layer only**: Never generate backend code (Data Access, Business Logic, API clients)
2. **Mock data only**: Use `List<T>` with hardcoded samples; no HttpClient calls to real APIs
3. **No secrets**: Exception: `app.config` or user secrets for `SYNCFUSION_LICENSE_KEY` when user provides
4. **WinForms classes only**: Generate `.cs` (Form, UserControl) files in Forms directories
5. **Redirect backend requests**: *"This skill generates WinForms UI only. Backend services and data integration are your app's responsibility. Ready to generate the UI?"*

### Error Handling

If any stage fails:

1. **Retry once** with same approach
2. **If retry fails**, attempt workaround or skip to next stage
3. **Notify user** with error message from stage output
4. **Offer recovery**: *"Would you like to go back to Stage 3 and choose a different layout?"*
5. **Debugging**: Check Stage 6 validation output and `winforms-dotnet-standards.md` for common issues

### Resource Loading Strategy (Progressive Disclosure)

**Load SKILL.md first** (you're reading it now) ~400 lines

**Load stage guides on-demand** (each <200 lines):
- `stage-1-intent-analysis.md` → During Stage 1
- `stage-2-project-detection.md` → During Stage 2
- `stage-3-layout-analysis.md` → During Stage 3
- etc.

**Load support references only when needed**:
- `winforms-dotnet-standards.md` → When validating in Stage 5 or debugging issues
- `assets/validation-rules.md` → When validating in Stage 6
- `syncfusion-themes.md` → When theming in Stage 4

**Result**: Initial load ~400 lines (SKILL.md only). Full spec available on-demand, never exceeding Agent Skills context limits.

## Configuration & User Customization

### Auto-Detected Settings

During **Stage 2 (Project Detection)**, AI automatically detects:

- **Framework**: .NET Framework 4.6.2+, .NET 8, .NET 9, .NET 10
- **Language**: C# version (8.0, 9.0, 10.0, 11.0+)
- **Project Type**: WinForms, WPF, or other .NET UI framework
- **Formatting**: EditorConfig rules (indentation, quotes, naming conventions)
- **Form Directory**: `Forms/`, `Views/`, `UI/`, or similar

### User Override Options

In **Stage 2**, user can override any detected setting:

```
Detected Settings:
  Framework: .NET 8
  Language: C# 12
  Project Type: WinForms
  Form Directory: Forms/

[Confirm] [Override Each] [Cancel]
```

### Syncfusion License Configuration

The skill handles license key setup:

1. **Check** for existing `SYNCFUSION_LICENSE_KEY` in app configuration or user secrets
2. **If missing**, prompt user: *"Get a free Community License at https://www.syncfusion.com/account/manage-trials"*
3. **If provided**, write to `app.config` or user secrets + inject `Syncfusion.Licensing.SyncfusionLicenseProvider.RegisterLicense()` in Program.cs or Form_Load
4. **If skipped**, proceed but warn that evaluation watermark will appear

---

## Code Generation Standards

All generated code includes:

### Accessibility (.NET Standards)
- ✅ Proper control TabIndex and TabStop properties
- ✅ Accessible names for controls via `Text` or `AccessibleName`
- ✅ Keyboard navigation (enter, escape, tab order)
- ✅ Screen reader support via `AccessibleRole` and `AccessibleDescription`
- ✅ Color contrast ≥ 4.5:1 for UI elements
- ✅ Focus indicators on interactive elements

### DPI Awareness & Responsive Design
- ✅ DPI-aware sizing (AutoScaleMode, AutoScaleDimensions)
- ✅ TableLayoutPanel/FlowLayoutPanel for dynamic resizing
- ✅ No hardcoded pixel values for multi-monitor support
- ✅ Responsive font scaling
- ✅ Touch-friendly controls (minimum 44x44px)

### Security
- ✅ Input validation and sanitization
- ✅ No hardcoded connection strings or secrets
- ✅ Parameterized queries (no SQL injection)
- ✅ No reflection security issues
- ✅ HTTPS only for external API calls

### Performance
- ✅ Efficient data binding (BindingSource with INotifyPropertyChanged)
- ✅ Lazy loading for large datasets
- ✅ Proper disposal of resources (IDisposable)
- ✅ Event handler cleanup

### C# & Type Safety
- ✅ Full type coverage (no dynamic types unless necessary)
- ✅ Nullable reference types enabled
- ✅ XML documentation comments
- ✅ Proper inheritance hierarchy

## Supported Use Cases

- **Login form**: TextBox (email), TextBox (password), CheckBox (remember), Button (submit)
- **Data table**: DataGridView or Syncfusion SfDataGrid with sorting, filtering, pagination, row selection
- **Document editor**: MDI form with RichTextBox or ScintillaNET
- **Settings dialog**: PropertyGrid or custom form with multiple tab pages
- **Data entry wizard**: Multi-step form with WizardControl and validation
- **Business forms**: Invoice, order, customer management with data binding

## Troubleshooting

**Common Issues:**

| Issue | Solution |
|-------|----------|
| "Project type not detected" | Ensure `.csproj` file exists with WinForms framework dependency |
| "Syncfusion evaluation watermark appears" | Add license key via Stage 2 prompt |
| "Build fails after insertion" | Check Stage 7 validation output and `winforms-dotnet-standards.md` |
| "Form not displaying controls" | Verify InitializeComponent() is called in Form constructor |
| "Controls misaligned on different DPI" | Ensure AutoScaleMode is set correctly for DPI awareness |

## Additional Resources

### Quick Reference by Use Case

| Need | Reference File |
|------|-----------------|
| Understanding workflow | This SKILL.md file |
| How Stage X works | `references/stage-X-*.md` |
| Validation rules | `assets/validation-rules.md` |
| .NET/Accessibility/security/troubleshooting | `references/winforms-dotnet-standards.md` |
| WinForms theming | `references/syncfusion-themes.md` |

### Architecture Compliance

✅ **Agent Skills Specification Compliant v2.0**
- YAML frontmatter with metadata
- One-level-deep file structure (no nested references between guides)
- Progressive disclosure (core <500 lines, details on-demand)
- Markdown-based guidance (AI-native)
- Clear scope boundaries (UI layer only, .NET WinForms)

## License & Attribution

- **UI Builder Skill**: Proprietary (Syncfusion)
- **Syncfusion Controls**: Requires valid Syncfusion license (Community, Trial, or Commercial)
- **Official Controls Repository**: https://github.com/syncfusion/winforms-ui-components-skills

## Support

For issues or questions:
1. Check `references/winforms-dotnet-standards.md` for common troubleshooting guidance
2. Verify your project meets prerequisites (.NET 8+, Visual Studio 2022+, C# 8.0+)
3. Ensure Syncfusion license is valid and registered
4. Review generated code compliance report for warnings

---

**Version**: 2.0 (AI-Native Architecture for WinForms/.NET)  
**Last Updated**: May 5, 2026  
**Next Phase**: MVVM pattern support, WPF variant, data binding orchestration
