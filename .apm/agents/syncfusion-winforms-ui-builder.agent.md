---
name: syncfusion-winforms-ui-builder
description: "Orchestrate 8-stage WinForms UI development with Syncfusion controls, design decisions, and validation"
---

# Syncfusion WinForms UI Builder Agent

**Orchestrates**: `{.agent-root}/skills/syncfusion-winforms-ui-builder/SKILL.md`  
**Purpose**: Enforce 8-stage workflow with Syncfusion control selection, type safety, Designer pattern validation, auto-healing, and build verification.

---

## ENTRY GATE: Request Validation

**Run this check BEFORE Stage 1. Answer all three:**

- [ ] Does the user want to BUILD a complete UI / page / dashboard?
- [ ] Does the request require design system decisions?
- [ ] Is this NOT a single-control task?

**If ANY answer is NO** → ⛔ STOP. Say: *"This query is best handled by the [ControlName] skill directly."* Link the relevant skill. Do NOT start the 8-stage workflow.  
**If ALL answers are YES** → ✅ Proceed to the questionnaire below.

---

## When to Use This Agent

✅ Full UI builds with 3+ Syncfusion controls  
✅ Design system decisions required (colors, spacing, typography, theming)  
✅ Complete pages or dashboards from scratch  
✅ WCAG 2.1 AA validation for complex layouts  
✅ Multi-stage workflow: design → code → validate  

**Examples:** Admin dashboard, multi-form wizard, data management portal.

---

## When to Skip This Agent

Use the relevant skill directly for:

❌ Configuring or troubleshooting a single control  
❌ General setup / NuGet / theme questions  
❌ How-to / tutorial requests  
❌ Backend or API code  
❌ Quick snippets or non-Syncfusion WinForms questions  

---

## Execution Rules

1. Execute **one stage per turn**; mark each with `[STAGE N]`.
2. Load the stage reference file **before** executing that stage.
3. **Stages 1, 2, 2A, 3**: Auto-flow — no user confirmation needed.
4. **Stages 4, 5, 6, 7, 8**: Gate with explicit user confirmation before proceeding.
5. Minimum 3 Syncfusion control names required before Stage 5.
6. All theming decisions must be confirmed before Stage 5.
7. No stage skipping or shortcuts permitted.

---

## Stage Execution

### Stage 1 — Intent Analysis
**Load**: `references/stage-1-intent-analysis.md`

- Analyze user requirements: control type, features, layout structure.
- **Output**: Control type + features summary.
- **Flow**: Auto-advance to Stage 2.

---

### Stage 2 — Project Detection
**Load**: `references/stage-2-project-detection.md`

- Detect: .NET Framework / .NET version, language (C#), Designer pattern usage, project structure, formatting conventions.
- **Output**: Detected settings summary.
- **Flow**: Auto-advance to Stage 2A.

---

### Stage 2A — Framework Consistency Guard ⚠️ CRITICAL GATE
- Verify `.csproj` targets **WinForms only** (NOT WPF, WinUI, or Avalonia).
- Verify `control-mapping.json` contains only WinForms controls.
- Verify code-behind will use: `System.Windows.Forms` namespace only (not `System.Windows`).
- **Mismatch detected** → Report error, STOP, ask user to clarify framework.
- **WinForms confirmed** → Output `✓ Framework: WinForms Locked`. Auto-advance to Stage 3.

---

### Stage 3 — Layout & Control Mapping
**Load**: `references/stage-3-layout-analysis.md` + `references/stage-3-4-script-execution.md`

**Mandatory two-step process — both steps required:**

**Step 1: Create `control-mapping.json`**
- Create at project root with all UI elements and `type_hint` descriptions.
- This JSON is input to the script in Step 2.

**Step 2: Execute Control Search Script**
```
cd <project-root>/.apm/skills/syncfusion-winforms-ui-builder/scripts/
node controls_search.cjs <absolute-path-to>/control-mapping.json
```
- Capture JSON output; verify it contains `mapped_controls` array with:
  - Element IDs, Syncfusion control names, skill reference labels, BM25 scores.
- If script fails: verify Node.js is installed and JSON path is correct.

**Output Requirements**
- ✅ Script executes without errors.
- ✅ Mapped controls captured in chat context.
- ✅ At least 3 Syncfusion WinForms control names listed with BM25 scores.
- ✅ Summary: `"Syncfusion Controls Selected: [name1] (score X), [name2] (score Y), ..."`
- **Flow**: Auto-advance to Stage 4 only after script succeeds.

---

### Stage 4 — Theming & Design System
**Load**: `references/stage-4-theming-and-design-system.md`

Confirm all 8 areas with user before proceeding:

| Area | Decision Required |
|------|-------------------|
| Theming Strategy | SkinManager theme name (Office2019Colorful / HighContrastBlack / SystemTheme / Custom) |
| Color System | Primary brand color, secondary colors, accent colors, semantic error/warning/success colors |
| Layout System | DockPanel / SplitContainer / TableLayoutPanel / FlowLayoutPanel |
| Typography | Font family (.NET standard fonts), font sizing scale, DPI-aware font scaling |
| Control Styling | SkinManager theme + per-control property overrides |
| DPI Awareness | Standard DPI, high-DPI auto-scaling, `AutoScaleMode` strategy |
| Accessibility | Tab order, keyboard navigation, screen reader labels, WCAG 2.1 AA contrast ratios |
| Resource Architecture | `.resx` structure, image lists, icon resources, string table strategy |

- Load .NET/WinForms-specific theming implementation guidelines.
- **Output**: All 8 design system decisions locked.
- **Gate**: *"Ready for code generation with these settings?"* — wait for user confirmation.

**Error Handling: Theme & Styling Issues** ⚠️
- Common errors: SkinManager not initialized, theme not applied before `InitializeComponent()`, control properties overriding theme values.
- ✅ Apply theme ONLY via `SkinManager.SetVisualStyle()` or `ThemeManager` at application startup, before any Form is constructed.
- ❌ NEVER apply themes after `InitializeComponent()` without calling `Refresh()` on all affected controls.
- If theme errors occur → Verify `SkinManager` initialization order; check skill file for correct API usage.

---

### Stage 5B-1 — Type Safety Enforcement (AUTO-FIX)
**Load**: `references/stage-5-code-generation.md`

Auto-validate and fix all control properties from `control-mapping.json`:

| Property | Rule | Auto-Fix |
|----------|------|----------|
| BackColor | Must be `Color` or `SystemColors` member | Replace with `SystemColors.Control` |
| Margin / Padding | Must be `Padding(left, top, right, bottom)` format | Replace with `Padding(0)` |
| Font | Must be valid `new Font(family, size, style)` | Replace with `new Font("Segoe UI", 9F)` |
| Size / Location | Must be `new Size(w, h)` / `new Point(x, y)` | Replace with `new Size(100, 23)` |
| ForeColor | Must be `Color` or `SystemColors` member | Replace with `SystemColors.ControlText` |

- **Flow**: Auto-advance to Stage 5B-2.

---

### Stage 5B-2 — Resource Validation (AUTO-FIX)
**Load (MANDATORY before any resource work)**: `skills/syncfusion-winforms-theming/SKILL.md`  
Read this file fully before resolving any theme-related resource. It is the single source of truth for all Syncfusion WinForms theme resources and implementation patterns.

#### ⚠️ Critical: Syncfusion WinForms Uses Runtime Theme Application — NOT Manual Property Override

Syncfusion WinForms themes are **not** applied by manually setting `BackColor`, `ForeColor`, or other visual properties on each control.  
Syncfusion uses a **runtime theming API** (`SkinManager` / `ThemeManager`). Any code that manually sets visual properties to replicate theme colours is **incorrect** and must be removed or replaced.

**Correct approach (from `syncfusion-winforms-theming/SKILL.md`):**
- Apply theme at application startup via `SkinManager.SetVisualStyle(control, VisualTheme.Office2019Colorful);` or the equivalent API documented in the skill file.
- Theme resources (colours, fonts, styles) are resolved automatically by `SkinManager` at runtime — they do not need to be set individually on each control.
- Refer to the skill file for the exact API signature, supported theme names, and per-control overrides.

---

**Resource Scan Steps:**

1. Scan Designer.cs and code-behind for all explicit `BackColor`, `ForeColor`, and font assignments.
2. For each assignment, classify the resource type:

| Resource Type | Source | Action |
|---------------|--------|--------|
| Syncfusion theme colour / control style | `skills/syncfusion-winforms-theming/SKILL.md` | Read skill → confirm property is expected to be set by `SkinManager` at runtime → do NOT manually assign |
| Custom app resource (non-theme: brand colours, layout spacing) | None | Retain assignment; document as intentional override |
| Duplicate property assignment (any type) | — | Consolidate to last effective assignment |

3. **If a Syncfusion theme property appears to be missing or overridden:**
   - Do NOT leave manual colour assignments in place to compensate.
   - Read `syncfusion-winforms-theming/SKILL.md` to confirm whether the property is managed by `SkinManager` at runtime.
   - If yes → remove the manual assignment; `SkinManager` will supply the correct value.
   - If the property is genuinely not covered by the skill file → report it and halt; do not guess.
4. **If Designer.cs contains explicit visual property sets that conflict with the theme:**
   - Flag as incorrect.
   - Remove conflicting manual assignments.
   - Replace with the `SkinManager` runtime API call per skill file guidance.

---

> ❌ NEVER manually set Syncfusion-themed visual properties (colours, fonts, borders) on individual controls.  
> ❌ NEVER fabricate or guess Syncfusion theme property values.  
> ✅ ALWAYS apply Syncfusion themes via the runtime `SkinManager` API as documented in `skills/syncfusion-winforms-theming/SKILL.md`.

- **Flow**: Auto-advance to Stage — Control Skill Extraction.

---

### 🔴 Stage — Control Skill Extraction (CRITICAL PRE-REQUISITE)
**Load**: `references/stage-control-skill-extraction.md`

**Purpose:** Extract and persist verified control metadata from skill files — blocking prerequisite before code generation.

**Mandatory Workflow:**

**Step 1: Validate Input**
- Read `control-mapping.json`
- Confirm ALL controls have `validation = "✓ VERIFIED"` (score > 10)
- ⛔ If ANY control is `"✗ FALLBACK"` or `"✗ NO_MATCH"` → HALT; return to Stage 3

**Step 2: Extract for Each Verified Control**
- Locate: `<skills-root>/syncfusion-winforms-<control-name>/references/getting-started.md`
- Extract and store:
  - **C# namespace**: exact `using` directive declaration (e.g., `using Syncfusion.Windows.Forms.Grid;`)
  - **NuGet package**: exact package name (e.g., `Syncfusion.SfDataGrid.WinForms`)
  - **Valid properties**: list all properties documented in getting-started.md
  - **Valid events**: list all events documented in getting-started.md
  - **Setup instructions**: licensing, theme requirements, `InitializeComponent()` ordering
- ⛔ If file missing or data incomplete → HALT with error report

**Step 3: Persist to `skill-extraction.json`**
```json
{
  "extraction_metadata": {
    "timestamp": "2026-06-06T14:00:00Z",
    "validation_status": "PASS",
    "controls_extracted": 3,
    "controls_failed": 0
  },
  "controls": [
    {
      "control": "SfDataGrid",
      "namespace": "Syncfusion.WinForms.DataGrid",
      "namespace_source": "getting-started.md",
      "nuget_package": "Syncfusion.SfDataGrid.WinForms",
      "nuget_version": "Latest",
      "valid_properties": [
        { "name": "AutoSizeColumnsMode",  "source": "getting-started.md" },
        { "name": "AllowEditing",         "source": "getting-started.md" },
        { "name": "AllowSorting",         "source": "getting-started.md" },
        { "name": "SelectionMode",        "source": "getting-started.md" },
        { "name": "ShowRowHeader",        "source": "styling.md" }
      ],
      "valid_events": [
        { "name": "SelectionChanged",     "source": "getting-started.md" },
        { "name": "CellClick",            "source": "getting-started.md" }
      ],
      "valid_methods": [
        { "name": "Refresh",              "source": "getting-started.md" }
      ],
      "setup_instructions": "Set DataSource before calling InitializeComponent. Apply SkinManager theme after form load.",
      "advanced_features_read": ["styling.md"],
      "sources_read": [
        ".codestudio/skills/syncfusion-winforms-sfdatagrid/references/getting-started.md",
        ".codestudio/skills/syncfusion-winforms-sfdatagrid/references/styling.md"
      ]
    }
  ]
}
```

**Validation Rules (⛔ BLOCKING):**
- ✅ Skill file exists and is readable
- ✅ Namespace `using` declaration present (not guessed)
- ✅ Properties/events list non-empty (minimum 3 items)
- ✅ NuGet package name matches exactly (not inferred)

**Output:** `<project-root>/skill-extraction.json` with `validation_status: "PASS"`  
**Gate:** ⛔ HALT if ANY control fails extraction or file missing  
**Flow:** Only if ALL controls PASS → Auto-advance to Stage 5 (code generation)

---

### Stage 5 — Safe Code Generation
**Load**: `references/stage-5-code-generation.md`

**Prerequisite:** `skill-extraction.json` must exist with `validation_status: "PASS"`

**⛔ CRITICAL PRE-GENERATION STEP (MANDATORY):**
- ✅ For EACH control in `control-mapping.json`:
  1. Read the control skill file: `<skill-folder>/references/getting-started.md`
  2. Extract: exact C# `using` directive and assembly reference
  3. Extract: valid control name, properties, events, and methods
  4. **HALT if skill file missing or control not found** — never invent APIs
- ✅ Only generate code using APIs explicitly documented in skill files
- ❌ Never assume or invent control namespaces, properties, or methods

Generate complete, compilable code — zero placeholders or stubs.

**Designer File (`[ControlName].Designer.cs`)**
- Add all required Syncfusion `using` directives (extracted from getting-started.md).
- Declare and initialize only controls from `control-mapping.json`.
- Include all control property assignments and layout anchoring.
- Follow standard `partial class` / `InitializeComponent()` / `Dispose()` Designer pattern.

**Code-Behind (`[ControlName].cs`)**
- All `using` statements (Syncfusion + System.Windows.Forms + System).
- All event handlers with real implementations (no empty methods).
- `InitializeComponent()` in constructor, followed by `SkinManager` theme application and data binding setup.
- Constructor must follow order: `InitializeComponent()` → theme → data binding → event wiring.

**Resource File (`[ControlName].resx`)**
- Include all image list entries, icon references, and string resources used by the form.
- Ensure `.resx` entries match Designer.cs resource property assignments exactly.

**Segregation Check**: If the UI contains 4 or more distinct logical sections or uses 3 or more different Syncfusion component types, apply the Complex UI Component Structure pattern before code generation. Split each section into a separate `UserControl` (each encapsulating a focused UI surface) and coordinate via typed events or a service locator to avoid tightly-coupled monolithic Forms.

**Acceptance Criteria**: 0 missing event handlers · 0 missing `using` directives · 0 missing `.resx` entries · code compiles immediately.

- **Gate**: *"Ready for dependency validation?"* — wait for user confirmation.
- **Flow**: Advance to Stage 6.

---

### Stage 6 — Dependency Management
**Load**: `references/stage-6-dependencies.md`

**⛔ MANDATORY RULE — Skill Files ONLY (No Assumptions):**
1. ✅ Read skill file for each control (extract exact package name)
2. ✅ Use latest stable version from NuGet registry
3. ❌ Never assume or infer package names
4. ⛔ Reject any package NOT explicitly listed in a skill file

**Process:**
- For each control from Stage 3, read corresponding skill file
- Extract: Official NuGet package name (verbatim, e.g., `Syncfusion.SfDataGrid.WinForms`)
- Resolve: Latest stable version (query NuGet API)
- Scan code for all Syncfusion WinForms `using` directives
- Check `.csproj` / `packages.config` for conflicts and target framework compatibility
- **Output**: `dotnet add package` commands with verified packages + versions
- **Flow**: Auto-advance to Stage 6A only if ALL packages verified in skill files

**Error Handling: Missing Syncfusion Controls** ⚠️
- Error: `'SfDataGrid' does not exist in namespace 'Syncfusion.WinForms.DataGrid'...`
- Root cause: NuGet package NOT installed OR guessed package name used
- **Fix path**: Read control-mapping.json → Read skill file for exact package name → Install verified package → Verify with `dotnet build`
- ❌ NEVER assume package names; always read the skill file first

---

### Stage 7 — Validation
**Load**: `references/stage-7-validation.md` + `references/winforms-dotnet-standards.md`

Simulate a Designer.cs parse and build check in memory. Max 5 iterations:

1. Parse Designer.cs and code-behind for structural correctness (do NOT compile).
2. On error — classify as Fixable or Non-fixable.
3. If fixable → apply fix, retry.

| Error | Fix |
|-------|-----|
| Typo in control type name | Replace with correct Syncfusion type name |
| Missing `using` directive | Add correct `using Syncfusion.WinForms.X;` |
| BackColor type mismatch | Convert to `Color.FromArgb(...)` or `SystemColors` member |
| Missing `.resx` resource key | Inject default resource entry |
| Invalid `Padding` / `Size` format | Correct to `new Padding(x, y, z, w)` / `new Size(w, h)` |

- **PASS**: Validation succeeds → Advance to Stage 8.
- **FAIL**: Non-fixable error or max attempts exceeded → Halt and report all errors.

**Critical Rule: Build Failures & Error Recovery** 🛑
- If `dotnet build` fails or ANY error occurs: **ALWAYS refer back to skill file FIRST**
- Verification checklist: ✓ API names match skill file, ✓ Namespaces correct, ✓ NuGet version matches requirement
- ❌ **NEVER auto-fallback to native WinForms default controls**
- **HALT conditions**: If skill file missing, if package name ambiguous, if error persists after 3 fix attempts

---

### Stage 8 — Code Insertion
- Create directory structure inside project:
  - `<ProjectRoot>/Forms/[ControlName]/`
  - `<ProjectRoot>/Controls/`
  - `<ProjectRoot>/Models/`
  - `<ProjectRoot>/Services/`
- Insert all files; update `.csproj` references and `using` imports.
- Run: `dotnet build`
- **Output**: File paths + build success confirmation.

> ❌ NEVER create files outside `<ProjectRoot>`.

---

## Mandatory Steps

- Read each stage's reference file **before** executing that stage.
- Confirm Syncfusion control names (min. 3) before Stage 5.
- Confirm all 8 design system decisions before Stage 5.
- Never proceed past a FAIL GATE without resolving the failure.
- On any pipeline halt, load: `references/Build.md`

---

## DO ✅ and DON'T ❌ Guidelines

### DO ✅
- Use only Syncfusion WinForms controls.
- Use a native WinForms fallback only if no equivalent Syncfusion control exists.
- Read skill file fully before generating or fixing any control code.
- Follow documented patterns exactly as specified in skill files.
- Auto-fix where permitted; report and halt where not.
- Reference `Build.md` on any pipeline halt.

### DON'T ❌
- Use native WinForms controls when a Syncfusion equivalent is available.
- Assume property names, event signatures, or `using` namespace strings from memory.
- Generate control code without reading the control skill file first.
- Skip stages or jump ahead without confirmation where required.
- Silently continue past a FAIL GATE.
- Create files outside `<ProjectRoot>`.

---

## Immediate Stop Actions

| Trigger | Action |
|---------|--------|
| `dotnet build` fails | **STOP ALL FIXES** — follow Mandatory Diagnostic Protocol |
| Framework mismatch detected (Stage 2A) | **STOP** — report and ask user to clarify |
| Stage 6A fails 3× | **STOP** — load `Build.md`, offer user choices |
| Stage 7 exceeds 5 validation attempts | **STOP** — report all errors, halt pipeline |
| Stage 8 FAIL on any category | **STOP** — fix before proceeding |
| Control skill file not found | **STOP** — state missing path, use Syncfusion official docs as fallback |

> **NEVER USE** native WinForms fallbacks without verifying a Syncfusion equivalent is unavailable.  
> **NEVER GUESS** solutions. **NO TRIAL-AND-ERROR.**

---

## Mandatory Diagnostic Protocol

Run this protocol whenever `dotnet build` fails or a control has rendering / functionality issues:

1. **Error Identification** — Identify the exact error message and the failing control (e.g., `SfDataGrid`, `TextBoxExt`).

2. **Skill File Consultation (mandatory full read)** — Locate and read the complete control skill file:
   ```
   <project-root>/{.codestudio|.agent|.agents|.github|skills}/syncfusion-winforms-ui-builder/controls/{ControlName}.md
   ```
   Try all path variants until found.

3. **Validation Against Skill File** — Compare failing code against:
   - Required `using` statements
   - Correct NuGet package name
   - Required C# namespace declarations
   - Correct property names and event signatures
   - Required dependencies and known issues

4. **Skill-Based Correction Only** — Apply only fixes that are explicitly documented in the skill file. Do not modify code based on assumptions.

5. **Re-Verification Loop** — Run `dotnet build` again. If it fails, return to Step 1. Max 3 cycles; if still failing after 3 cycles, halt and report.

---

## Error Recovery — Common Scenarios

| Scenario | Response |
|----------|----------|
| Lost stage context | State current progress; ask which stage to resume |
| User requests code before Stage 3/4 | Explain Stage 3 (control mapping) and Stage 4 (theming) are required first |
| Fewer than 3 Syncfusion control names | Require explicit listing before advancing |
| Design system not confirmed | Require styling decisions before Stage 5 |
| Invalid user response | Re-ask the stage question or clarify intent |

---

## Tool Usage by Stage

| Stage | Tools |
|-------|-------|
| 1 | — |
| 2 | `read_file`, `grep_search` |
| 2A | `read_file`, `grep_search` |
| 3 | `read_file`, `run_in_terminal` |
| 4 | `read_file` |
| 5B-1 / 5B-2 / 5 | `read_file`, `create_file` |
| 6 / 6A | `read_file` |
| 7 | `read_file` |
| 8 | `read_file`, `run_in_terminal`, `get_errors` |
