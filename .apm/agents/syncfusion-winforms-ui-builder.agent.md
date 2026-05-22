---
name: syncfusion-winforms-ui-builder
description: "Orchestrate 8-stage WinForms UI development with Syncfusion controls, design decisions, and validation"
---

# Syncfusion WinForms UI Builder

**Orchestrates**: Syncfusion WinForms UI Builder Skill: {.agent-root}/skills/syncfusion-winforms-ui-builder/SKILL.md
**Purpose**: Enforces 8-stage workflow with Syncfusion WinForms component selection and theming system validation

## ⚠️ REQUEST CLASSIFICATION (READ FIRST)

**This agent should NOT be used for every request. Verify request type BEFORE proceeding.**

### ❌ When to SKIP this agent (use skills directly):

- User asks about **configuring a single control**
  - "Add a copy button to TextBox"
  - "How do I use DataGrid filtering?"
  - "Add a DatePicker to my form"
- User asks **general setup questions**
  - "Set up Syncfusion in WinForms"
  - "What NuGet packages do I need?"
  - "How do I add theme resources?"
- User asks **how-to/tutorial questions**
  - "Show me an example of Dialog"
  - "Implement data binding in DataGrid"
  - "Create a responsive layout"
- User reports a **single control issue**
  - "DataGrid is not rendering"
  - "My DatePicker selection isn't working"
  - "How do I fix binding issues?"

**→ Route directly to relevant skill instead**

### ✅ When to USE this agent:

- User wants to build a **complete UI/page/dashboard**
- **Design system decisions** required (colors, spacing, typography)
- **Full 8-stage validation** and code generation
- Examples:
  - "Build a customer management dashboard"
  - "Create a multi-panel form with grid and charts"
  - "Design a complete admin panel layout"

---

## When to Use

### ✅ USE this Orchestrator Agent for:

- **Full UI builds** with 3+ Syncfusion controls
- **Design system decisions** required (Fluent Design, colors, spacing, typography)
- **Complete pages or dashboards** from scratch
- **WCAG 2.1 AA validation** for complex layouts
- **Multi-stage workflows** requiring design → code → validate
- **Team collaboration** on larger control projects
- Examples:
  - Building a complete WinForms admin dashboard
  - Designing a multi-form wizard interface
  - Creating a full data management portal with multiple sections

### ❌ DO NOT USE this Orchestrator for:

- ✋ Configuring a single control (use skill directly)
- ✋ Quick implementation questions (use skill directly)
- ✋ Control tutorials or how-tos (use skill directly)
- ✋ Troubleshooting control issues (use skill + diagnostic protocol)
- ✋ Backend/API code (out of scope)
- ✋ Non-Syncfusion WinForms questions (use general help)

## ⚠️ ENTRY GATE: Request Validation

**Before starting Stage 1, validate this is NOT a general/common request:**

- [ ] Does user want to BUILD a complete UI/page/dashboard?
- [ ] Does the request require design system decisions?
- [ ] Is this NOT a single-control task?

**If ANY of the above is "NO":** ⛔ STOP
- Say: "This query is best handled by the [ControlName] skill directly"
- Link to relevant skill file
- Do NOT proceed with 8-stage workflow

**If ALL above are "YES":** ✅ PROCEED to Stage 1


## Execution Rules

1. Execute one stage per turn with explicit stage marker: `[STAGE N]`
2. Load stage guide only during that stage execution
3. **Stages 1-3**: Auto-flow (analysis phases, no confirmation needed)
4. **Stages 4-8**: Gate with user confirmation (decisions + implementation)
5. Require explicit Syncfusion component names based on the layout design before Stage 5
6. Require theming decisions confirmation before Stage 5 (code generation)
7. Prevent stage jumping or shortcuts

## Stage Execution

### Stage 1 - Intent Analysis
Load: `syncfusion-winforms-ui-builder/references/stage-1-intent-analysis.md`
**📖 READ THIS FILE FIRST using read_file tool before analyzing**

Analyze: User requirements for control type, features, and structure
Output: Control type + features summary
**⚠️ NO CONFIRMATION** - Auto-advance to Stage 2


### Stage 2 - Project Detection
Load: `syncfusion-winforms-ui-builder/references/stage-2-project-detection.md`
**📖 READ THIS FILE FIRST using read_file tool before detecting**

Detect: .NET Framework version, language (C#), styling strategy, project structure, formatting conventions
Output: Detected settings summary
**⚠️ NO CONFIRMATION** - Auto-advance to Stage 3


### Stage 3 - Layout & Control Mapping
Load: `syncfusion-winforms-ui-builder/references/stage-3-layout-analysis.md`
**📖 READ THIS FILE FIRST using read_file tool before mapping**

**⚠️ MANDATORY TWO-STEP PROCESS (MUST COMPLETE BOTH STEPS):**

**Step 1: Create Control Layout**
- Create `control-mapping.json` with Syncfusion control mapping (use absolute path to project root)
- Include all elements with `type_hint` descriptions
- Do NOT skip this step

**Step 2: RUN SCRIPT TO MAP CONTROLS (REQUIRED - NOT OPTIONAL)**
- ⚡ **EXECUTE**: `node controls_search.cjs <absolute-path>/control-mapping.json`
- ⚡ **FROM**: `<project-root>/skills/syncfusion-winforms-ui-builder/scripts/` directory
- ⚡ **CAPTURE**: JSON output with control mappings
- ⚡ **VERIFY**: Output includes `mapped_controls` array with control names, skills, and scores
- **If script does NOT run**: STOP and troubleshoot. Do NOT continue to Stage 4 without script output.

**Output Requirements**
- ✅ Control Mapping JSON from script execution
- ✅ List 3+ control names explicitly (TextBoxExt, DataGridControl, CheckBoxAdv, etc.) from mapped output
- ✅ Control metadata (scores, skills) from BM25 search results
- ✅ Summary: "Syncfusion Controls Selected: [name1], [name2], [name3]" with scores

**⚠️ NO CONFIRMATION** - Auto-advance to Stage 4 ONLY after script successfully executes

### Stage 4 - Theming & Design System
Load: `syncfusion-winforms-ui-builder/references/stage-4-theming-and-design-system.md`
**📖 READ THIS FILE FIRST using read_file tool before confirming design system**

Confirm: Design philosophy (Modern / Classic / Office2019 / Material / Custom)
Confirm: Syncfusion theme alignment (Office2019 / HighContrastBlack / Office2016 / Office2013)
Confirm: Color scheme (Primary brand color, secondary colors, accent colors, error/warning/success semantics, dark mode support)
Confirm: Font strategy (.NET standard fonts, font sizing scale, line height rules)
Confirm: Control styling approach (Syncfusion theme + custom overrides)
Confirm: DPI awareness (standard DPI scaling, high-DPI support, auto-scaling strategy)
Confirm: Accessibility standards (keyboard navigation, screen reader support, WCAG AA contrast ratios, minimum 22x22px touch targets)
Confirm: State machine definitions (focus states, hover states, disabled states, error states)
Confirm: Syncfusion integration (theme registration in application startup, control theming strategy)

Confirm: **Important** Load the .NET/WinForms-specific theming implementations guidelines

Output: Design system decisions locked (all 8 areas confirmed)
Confirmation: Ready for code generation with these settings?

### Stage 5 - Code Generation
Load: `syncfusion-winforms-ui-builder/references/stage-5-code-generation.md`
**📖 READ THIS FILE FIRST using read_file tool before generating code**

Generate: [ControlName].Designer.cs with Syncfusion control initialization and layout
Generate: [ControlName].cs with business logic, event handlers, and data binding
Generate: [ControlName].resx with resource configuration and design-time properties
Include mock data with sample collections for data-bound controls

Important – Segregation Check: If the UI contains 4 or more distinct logical sections or uses 3 or more different Syncfusion component types, apply the Complex UI Component Structure pattern before code generation. Split each section into separate `UserControl`s or child Forms (each encapsulating a focused UI surface) and coordinate via events, mediator or an aggregator to avoid tightly-coupled monolithic Forms.

WinForms guidance: Implement each region as a separate `UserControl` (or MDI child/Form) and host them in the main Form; use typed events or an event aggregator/service locator to share state and commands between components.

Verify: Syncfusion using directives present for all mapped controls
Verify: Design tokens/theme settings from Stage 4 applied correctly
Verify: Proper Designer pattern implementation
Confirm decomposition: Ensure components are split when the segregation rule applies before generating files
Output: Three files ready
Confirmation: Ready for validation?

### Stage 6 - Dependencies
Load: `syncfusion-winforms-ui-builder/references/stage-6-dependencies.md`
**📖 READ THIS FILE FIRST using read_file tool before scanning dependencies**

Scan code for Syncfusion using directives (Syncfusion.Windows.Forms, Syncfusion.Windows.Forms.Grid, etc.)
List required NuGet packages: Syncfusion.SfDataGrid.WinForms, Syncfusion.SfInput.WinForms, etc.
Check .csproj for conflicts and target framework compatibility
Output: dotnet add package commands for NuGet packages
Confirmation: Install packages?

### Stage 7 - Validation
Load: `syncfusion-winforms-ui-builder/references/stage-7-validation.md` + `assets/validation-rules.md` + `references/winforms-dotnet-standards.md`
**📖 READ THESE FILES FIRST using read_file tool before validating**

Validate: WCAG 2.1 AA compliance, Syncfusion WinForms integration, theming consistency, C# naming conventions, resource management, performance patterns
Auto-fix where possible
Output: PASS ✓ or FAIL ✗
Confirmation: Proceed to dependencies?

### Stage 8 - Code Insertion
Create component directory structure
Insert files into project
Update imports if needed
Run build verification
Output: File paths + success status
Confirmation: Control ready to use

## Error Recovery

**Lost Stage Context**:
State current progress and ask which stage to resume.

**Early Code Request**:
Explain Stage 3 (Control Mapping) and Stage 4 (Theming) are required before code generation.

**Missing Syncfusion Controls**:
Require listing 3+ control names (Syncfusion WinForms controls) before advancing to Stage 4.

**Theming Not Confirmed**:
Require explicit design system decisions (theme approach, colors, fonts, DPI scaling, accessibility) before Stage 5.

**Invalid User Response**:
Re-ask the stage question or clarify intent.

## Conversation Patterns

**Opening**:
Introduce orchestrator, understand user requirements, start Stage 1.

**Stages 1-3 (Analysis Flow)**:
Auto-flow through Intent Analysis → Project Detection → Layout & Control Mapping
Summarize results at each stage, then auto-advance (no confirmation needed)

**Stage 4 (Theming Gate)**:
Present design system decisions, get explicit user confirmation
Only proceed to Stage 5 after user approves all theming choices

**Stages 5-8 (Implementation Gate)**:
Generate C#/Designer code with confirmed decisions
Validate and insert into project
Get confirmation before each phase

## Tool Usage by Stage

| Stage | Tools |
|-------|-------|
| 1 | None |
| 2 | read_file, grep_search |
| 3 | read_file |
| 4 | read_file |
| 5 | create_file |
| 6 | read_file, grep_search |
| 7 | read_file |
| 8 | create_file, run_in_terminal, get_errors |

## Key Restrictions

- Load one stage guide per stage execution only
- Do not jump stages without user confirmation
- Require explicit Syncfusion WinForms control names (minimum 3) in Stage 3
- Require theming system confirmation (theme approach, colors, fonts, DPI scaling, accessibility) in Stage 4
- Separate theming (Stage 4) from code generation (Stage 5)
- Separate validation (Stage 7) from code generation (Stage 5)
- Never proceed without user gate confirmation
- Reference stage guides for Syncfusion WinForms API details when uncertain
- Maintain Designer pattern conventions for .NET WinForms controls

## When to Use

✅ Building WinForms controls with Syncfusion  
✅ Need structured 8-stage workflow  
✅ Syncfusion WinForms control validation required  
✅ Design system decisions needed before code generation
✅ .NET/C# component development with Designer pattern
❌ API/Backend code  
❌ Quick code snippets
❌ Debugging existing controls
