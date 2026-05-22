# Stage 3: Layout Analysis & Control Mapping (Combined)

**Purpose:** Analyze user requirements, create optimal control-mapping.json, and map to Syncfusion WinForms controls automatically. **FULLY AUTOMATED — NO user interaction.**

---

## ⚠️ CRITICAL: Control Mapping Accuracy & BM25 Search Validation

**Stage 3 is critical for application success.** The BM25 search in this stage maps UI requirements to Syncfusion controls. **Incorrect mappings in Stage 3 lead to wrong control selection, requiring manual correction in Stage 5.**

### Prevention Safeguards

**To prevent BM25 search errors:**

1. **Ensure `control-mapping.json` accuracy**:
   - Every element MUST have a precise `type_hint` with keywords matching controls.csv keywords
   - Use **compound keywords** from controls.csv - e.g., `"grid data grid table sortable filterable"` not just `"grid"`
   - Include **context keywords** - e.g., for appbar elements, include `"appbar"` or `"header"` keyword
   - Add **modifiers** - e.g., `sortable`, `filterable`, `collapsible`, `paginated`

2. **Validate keyword selection against controls.csv**:
   - Before running `controls_search.cjs`, scan the relevant row in controls.csv
   - Ensure type_hint keywords EXIST in the control's Keywords column
   - Example: For TextBox control (row 99), keywords include `"textbox, text input, input, form, single line, multiline, floating label, icon"`
   - Example: For DataGrid control (row 25), keywords include `"grid, data grid, table, data table, sorting, filtering, paging, editing, grouping, aggregation, virtualization, export"`

3. **Use EXACTLY controls in controls.csv**:
   - Only use control names and keywords that appear in columns 1-5 of controls.csv
   - Never invent or infer control names
   - Refer to the **Control Name column (column 2)** for exact spelling and capitalization

4. **Review BM25 score output**:
   - Score **40+** = Excellent match (use confidently)
   - Score **20-40** = Good match (review context)
   - Score **<20** = Weak match (verify or reconsider element type_hint)
   - Score **0** = No match (fallback to native WinForms control, consider type_hint revision)

5. **Validate mapping results before Stage 4**:
   - Check the `validation` column in script output: **"✓ VERIFIED in controls.csv"** vs **"✗ NOT FOUND in controls.csv"**
   - If `✗ NOT FOUND`, the control fell back to native WinForms—verify if this is intentional
   - If unexpected fallback, revise type_hint in control-mapping.json and **re-run the script**

---

## ⚠️ CRITICAL: Understanding Control Skill Names vs. Official NuGet Packages

**The `skill_hint` and `Skill Name` values in controls.csv (e.g., `syncfusion-winforms-button`, `syncfusion-winforms-datagrid`) are REFERENCE LABELS for BM25 semantic search purposes ONLY.** They are NOT official Syncfusion NuGet package names.

**Stage 3 generates control mappings using these reference labels.** 
**Stage 5-7 must convert these reference labels to official Syncfusion NuGet package names that are EXPLICITLY documented in:**
- The corresponding control skill file (`.codestudio/skills/syncfusion-winforms-<name>/SKILL.md`), OR
- Official Syncfusion documentation at [Syncfusion WinForms Controls](https://help.syncfusion.com/windowsforms/overview)

**Examples of correct NuGet packages (EXPLICITLY documented in official sources):**
- Control: `DataGrid` → Skill: `syncfusion-winforms-datagrid` → Official NuGet: `Syncfusion.SfDataGrid.WinForms` (verified in official docs)
- Control: `Chart` → Skill: `syncfusion-winforms-chart` → Official NuGet: `Syncfusion.Chart.Windows` (verified in official docs)
- Control: `Button` → Skill: `syncfusion-winforms-button` → Official NuGet: `Syncfusion.Core.WinForms` or `Syncfusion.SfInput.WinForms` (verified in official docs)

**DO NOT use inferred package names.** Verify that EACH package name is EXPLICITLY documented in control skill files or official Syncfusion resources before use.

---

## Stage 3: Layout Analysis

### AI Should:

1. **Read control type** from Stage 1 intent analysis
2. **Analyze user query** for specific requirements and context
3. **Determine optimal layout variant** (no user choice needed)
4. **Create structured control-mapping.json** with all elements
5. **Output JSON** for Stage 3-4 combined processing (maps to WinForms controls)

### Decision Framework

Based on user requirements, AI selects the **best** layout (not multiple options):

| Control Type | Decision Criteria | Best Variant |
|---|---|---|
| **Login Form** | Enterprise? 2FA needed? Social login? | Choose: Minimal/Standard/Advanced |
| **Data Table** | Read-only or editable? Export needed? | Choose: Simple/Interactive/Full-featured |
| **Dashboard** | Internal or customer-facing? Complexity? | Choose: Focused/Standard/Enterprise |
| **Navigation** | Mobile traffic high? Multi-level needed? | Choose: Simple/Sidebar/Progressive |
| **Form** | Single-step or multi-step? Validation level? | Choose: Basic/Standard/Advanced |

**Key Principle:** AI makes the optimal choice based on the user query context and best practices. No variant selection UI needed.

### Output: Structured JSON (for Stage 3-4 Control Mapping)

```json
{
  "control_type": "Login Form",
  "variant": "Standard",
  "elements": [
    {
      "id": "email_input",
      "name": "Email Address",
      "description": "Email field with validation",
      "type_hint": "text input email form validation"
    },
    {
      "id": "password_input",
      "name": "Password",
      "description": "Password field, masked",
      "type_hint": "text input password form masked"
    },
    {
      "id": "remember_me",
      "name": "Remember Me",
      "description": "Keep me logged in",
      "type_hint": "checkbox form input"
    },
    {
      "id": "submit_button",
      "name": "Submit",
      "description": "Login button",
      "type_hint": "button primary action submit cta"
    }
  ]
}
```

**JSON Structure Details:**

- `control_type`: The UI control being built (e.g., "Login Form", "Data Table", "Dashboard")
- `variant`: Chosen variant based on user requirements (e.g., "Standard", "Advanced", "Minimal")
- `sections` (optional): For complex layouts, group elements into logical sections
  - `section_id`: Unique identifier (e.g., "header_section")
  - `section_name`: Display name (e.g., "Header")
  - `elements`: Array of elements within section
- `elements`: Array of control elements
  - `id`: Unique identifier (snake_case)
  - `name`: Display name for UI
  - `description`: What this element does
  - `type_hint`: UI element type for BM25 search (e.g., "text input", "button", "dropdown", "table", "chart")

**Important:**
- `type_hint` is critical for Stage 4 ControlMapper BM25 search accuracy (maps to WinForms controls)
- Keep descriptions concise and functional
- Use lowercase for `id` and `type_hint`
- Create `control-mapping.json` in project root (reused in Stage 4 & 5)
- Output minimal summary table in chat for visibility
- Map to Syncfusion WinForms controls and .NET Standard namespaces
- WinForms does not support icon mapping—focus on control type matching only

### Type Hint Best Practices

**For Header/AppBar Elements:**
- Always include `appbar` or `header` keyword in type_hint
- Examples:
  - Logo: `"image logo appbar header branding"` (not just `"image logo"`)
  - Notifications: `"icon button notification appbar header"`
  - User Menu: `"dropdown button menu user profile appbar header"`

**General Guidelines:**
- Use **compound keywords** - `"icon button notification"` scores better than `"notification"`
- Include **context keywords** - appbar/header/sidebar context improves matching
- Add **modifiers** - sortable, filterable, collapsible, paginated
- Reference **control keywords from CSV** - Use exact control keywords for best BM25 matching

**Scoring Guide:**
- Score **40+** = Excellent match (multiple keywords + context)
- Score **20-40** = Good match (several keywords)
- Score **<20** = Weak match (may fall back to NATIVE_HTML)
- Score **0** = No match (unrelated keywords)



## Complex Layouts (Multi-Section)

For dashboards, admin panels, or multi-section layouts:

```json
{
  "control_type": "Admin Dashboard",
  "variant": "Classic Admin Dashboard",
  "layout_grid": "2-column",
  "sections": [
    {
      "section_id": "header",
      "section_name": "Header",
      "description": "Fixed top panel/toolbar navigation",
      "docking": "top",
      "elements": [
        {
          "id": "logo",
          "name": "Company Logo",
          "description": "Brand logo in toolbar",
          "type_hint": "picturebox image logo toolbar header branding"
        },
        {
          "id": "notification_bell",
          "name": "Notifications",
          "description": "Button with icon and count in header",
          "type_hint": "button icon notification toolbar header",
          "skill_hint": "syncfusion-winforms-button"
        },
        {
          "id": "user_menu",
          "name": "User Profile",
          "description": "ComboBox or dropdown menu for user profile",
          "type_hint": "combobox dropdown button menu user profile toolbar header",
          "skill_hint": "syncfusion-winforms-combobox"
        }
      ]
    },
    {
      "section_id": "sidebar",
      "section_name": "Sidebar",
      "description": "Left navigation panel",
      "docking": "left",
      "collapsible": true,
      "elements": [
        {
          "id": "nav_menu",
          "name": "Navigation Menu",
          "description": "Main navigation tree or list",
          "type_hint": "treeview listbox navigation menu sidebar collapsible",
          "skill_hint": "syncfusion-winforms-treeview"
        }
      ]
    },
    {
      "section_id": "main_content",
      "section_name": "Main Content",
      "docking": "fill",
      "elements": [
        {
          "id": "kpi_cards",
          "name": "KPI Cards",
          "description": "Panel with metric cards displaying statistics",
          "type_hint": "panel card grid statistics dashboard metrics kpi"
        },
        {
          "id": "data_grid",
          "name": "Users Table",
          "description": "Data table with sorting and filtering",
          "type_hint": "grid data grid table sortable filterable paging",
          "skill_hint": "syncfusion-winforms-datagrid"
        }
      ]
    }
  ]
}
```

---

## Stage 3: MANDATORY SCRIPT EXECUTION

### ⚠️ CRITICAL: Create & Process control-mapping.json

**WORKFLOW:**
1. ✅ **Create `control-mapping.json`** at project root with element structure
2. ✅ **Run `controls_search.cjs` script** to map elements to Syncfusion WinForms controls
3. ✅ **Capture output** in chat for Stage 4 (theming) & Stage 5 (code generation)

---

### Step 1: Create `control-mapping.json` (Workspace Root - Outside Project)

**Location:** `<workspace-root>/control-mapping.json` (OUTSIDE the project folder, NOT in scripts folder)

**Why outside the project folder:**
- ✅ Supports multi-project environments (same mapping can be reused across projects)
- ✅ Script execution reliability with absolute paths (less prone to relative path errors)
- ✅ Prevents accidental inclusion in project version control
- ✅ Keeps project folder clean (only project-specific files)
- ✅ Allows workspace-level control mapping auditing and management

**Example directory structure:**
```
D:\MyWorkspace\                                 ← workspace root
├── control-mapping.json                        ← CREATE HERE (workspace root)
├── WinFormsApp1\                               ← project root
│   ├── WinFormsApp1.csproj
│   ├── Forms\
│   └── ...
├── WinFormsApp2\                               ← another project
│   ├── WinFormsApp2.csproj
│   ├── Forms\
│   └── ...
└── .apm\skills\syncfusion-winforms-ui-builder\
    ├── scripts\
    │   ├── controls.csv
    │   └── controls_search.cjs
    └── ...
```

**File will contain:** Elements with `type_hint` for BM25 control mapping

---

### Step 2: MANDATORY - Run Controls Mapper Script

**Why:** Automatically map UI elements to Syncfusion WinForms controls using BM25 semantic search

**Prerequisites:**
- Node.js 14+ installed
- `control-mapping.json` created in Step 1
- Script located at: `<project-root>/.apm/skills/syncfusion-winforms-ui-builder/scripts/controls_search.cjs`

**Execution (Windows with absolute path - REQUIRED):**

```powershell
cd <workspace-root>\.apm\skills\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs <workspace-root>\control-mapping.json
```

**Replace placeholders:**
- `<workspace-root>` = Your workspace directory (parent of all projects, e.g., `d:\MyWorkspace`)

**Real example:**
```powershell
cd d:\MyWorkspace\.apm\skills\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs d:\MyWorkspace\control-mapping.json
```

**Alternative directory structures (hidden config dirs):**
```powershell
# If using .agent or .agents or .github
cd d:\MyWorkspace\.agent\skills\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs d:\MyWorkspace\control-mapping.json
```

**Expected Output:**
- JSON with `mapped_controls` array
- Each element mapped to Syncfusion control + BM25 score
- Unmatched controls → fallback to native System.Windows.Forms

---

### Step 3: Capture Output in Chat

**Actions:**
- ✅ Script outputs control mapping JSON to terminal/console
- ✅ Copy the output into chat context
- ✅ Reference mapping in Stage 4 (theming) & Stage 5 (code generation)
- ✅ Do NOT save script output to file (keep in conversation only)

**Why:**
- Token efficiency - only control results in chat, not redundant JSON
- Clean context for reasoning stages
- Avoid file proliferation

### Workflow Benefits
| Aspect | Benefit |
|--------|---------|
| **Automation** | Single script run maps all controls instantly |
| **Accuracy** | BM25 algorithm ranks best Syncfusion control per element |
| **Persistence** | `control-mapping.json` stays in project (version control + auditing) |
| **Token Efficiency** | Filesystem I/O avoids re-passing JSON to chat |
| **Scriptability** | Node.js script is IDE-agnostic and platform-independent |
| **Reusability** | Mapping can be re-run if requirements change |

---

## Stage 3-4 Part 2: Control Picking (Script-Based)

### ⚠️ MANDATORY: TWO-STEP PROCESS - BOTH STEPS REQUIRED

**Step 1: Create `control-mapping.json`** in project root (NOT in scripts folder)
```bash
# File: ./control-mapping.json (at workspace root)
# Contains: elements with type_hints for BM25 search
```

**Step 2: ⚡ RUN SCRIPT (REQUIRED - NOT OPTIONAL) ⚡**
```bash
cd <project-root>/skills/syncfusion-winforms-ui-builder/scripts
node controls_search.cjs <absolute-path>/control-mapping.json
```

**CRITICAL: Script MUST execute successfully before proceeding to Stage 4**
- ✅ Script reads control-mapping.json from disk
- ✅ Runs BM25 search on 110 Syncfusion WinForms controls
- ✅ Outputs JSON with `mapped_controls` array
- ✅ Each element gets mapped to a control with score
- ❌ If script fails → STOP and debug before Stage 4

**Step 3: Capture Output in Chat Context**
- ✅ Script outputs control + icon mapping JSON
- ✅ Keep mapping results in conversation context ONLY (no file)
- ✅ Do NOT save script output to file
- ✅ Reference mapping in chat for Stages 4-5
- ✅ Verify output has `mapped_controls` with 3+ controls mapped

### Workflow Benefits
| Aspect | Benefit |
|--------|---------|
| **Token Efficiency** | Create control-mapping.json once, reuse in script without re-passing JSON in chat |
| **Persistence** | `control-mapping.json` stays in project for version control + auditing |
| **Scriptability** | Script reads `control-mapping.json` directly from filesystem (faster) |
| **Clean Context** | Only control mapping output in chat (not redundant control-mapping.json) |
| **Reusability** | Stage 5 code generation can reference `control-mapping.json` if needed |

---

## Stage 3-4 Part 2: Control & Icon Picking (Script-Based)

### Input: Controls-Mapping JSON from Stage 3

Receive the control-mapping.json created above with `icon_hint` fields for control elements and `icon_elements` for icon-only elements.

### Processing: ControlMapper + IconMapper Script

**⚠️ MANDATORY STEPS (DO NOT SKIP):**

1. **Create** `control-mapping.json` at workspace root (e.g., `D:\MyWorkspace\control-mapping.json`) - OUTSIDE project folder
2. **Execute** script with ABSOLUTE PATH (Windows best practice)
3. **Capture** script output in chat context (DO NOT save to file)
4. **Reference** control mapping in subsequent stages

**Execution (REQUIRED) - Windows:**

Use absolute path for reliable execution:
```bash
cd <your-workspace-root>\<skills-dir>\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs <your-workspace-root>\control-mapping.json
```

**Replace placeholders:**
- `<your-workspace-root>` = Your workspace's root directory (e.g., `d:\my-workspace`)
- `<skills-dir>` = Skills directory name (configuration-specific, e.g., `.codestudio\skills`, `.agents\skills`, `.github\skills`, or `skills`)

**Real example (with hidden config directory like .codestudio):**
```bash
cd d:\my-workspace\.codestudio\skills\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs d:\my-workspace\control-mapping.json
```

**Real example (with hidden config directory like .agents):**
```bash
cd d:\my-workspace\.agents\skills\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs d:\my-workspace\control-mapping.json
```

**Real example (with visible skills directory):**
```bash
cd d:\my-workspace\skills\syncfusion-winforms-ui-builder\scripts
node controls_search.cjs d:\my-workspace\control-mapping.json
```

**Path Resolution Rules (for Script):**
- ✅ **Absolute paths REQUIRED** - Full path from C:\ or D:\ or any drive (most reliable for workspace root access)
- ✅ **Workspace root location** - control-mapping.json always at workspace root, OUTSIDE project folder
- ✅ **Fully IDE-agnostic** - Works with ANY skills directory structure (`.codestudio/`, `.agents/`, `.github/`, `skills/`, or custom)
- ✅ **Editor-independent** - Not tied to specific IDE names or conventions
- ✅ **Multi-project support** - Single control-mapping.json can be shared/reused across multiple projects in workspace
- ✅ Script automatically resolves absolute paths from workspace root
- ✅ Script validates path exists before processing and shows full path if error occurs
- ❌ Avoid relative paths like `../control-mapping.json` (can cause "file not found" errors)
- ❌ Never place control-mapping.json inside project folder (defeats workspace-level management)

**Output Destination:**
- ✅ Control mapping JSON printed to terminal
- ✅ **Captured in chat context ONLY** (not saved to file)
- ✅ Used for Stage 4 theming + Stage 5 code generation

This automatically:
- Maps elements → Syncfusion WinForms controls (BM25 search)
- Produces Stage 4 output JSON with control mappings and skills
- **Results stay in conversation, not persisted to disk**
- Targets .NET Standard compatible control namespaces

### How It Works

1. **Control Mapping**: BM25 search on `type_hint` → finds best Syncfusion WinForms control
2. **Fallbacks**: 
   - No control match → Standard System.Windows.Forms control
3. **Output**: Complete mapping with controls for Stage 5

### BM25 Search Algorithm

The ControlMapper uses **BM25 (Best Matching 25)**:

- **Tokenizes** query and control keywords
- **Calculates** term frequency (TF) in each WinForms control
- **Calculates** inverse document frequency (IDF) across all controls
- **Applies** BM25 formula for semantic relevance ranking
- **Returns** ranked results with scores
- **Namespace Resolution**: Maps controls to their Syncfusion.Windows.Forms.* or System.Windows.Forms namespaces

**Quality:** Only controls with score > 0 are matched; unmatched → fall back to standard System.Windows.Forms controls

### Output: Control & Icon Mapping

```json
{
  "control_type": "Login Form",
  "variant": "Standard",
  "mapped_controls": [
    {
      "element_id": "email_input",
      "element_name": "Email Address",
      "control": "TextBoxExt",
      "control_alias": "TextBoxExt",
      "skill": "syncfusion-winforms-textbox",
      "skill_hint": "syncfusion-winforms-textbox",
      "score": 13.24,
      "validation": "✓ VERIFIED in controls.csv"
    },
    {
      "element_id": "password_input",
      "element_name": "Password",
      "control": "TextBoxExt",
      "control_alias": "TextBoxExt",
      "skill": "syncfusion-winforms-textbox",
      "skill_hint": "syncfusion-winforms-textbox",
      "score": 12.87,
      "validation": "✓ VERIFIED in controls.csv"
    },
    {
      "element_id": "remember_me",
      "element_name": "Remember Me",
      "control": "CheckBoxAdv",
      "control_alias": "CheckBoxAdv",
      "skill": "syncfusion-winforms-checkbox",
      "skill_hint": "syncfusion-winforms-checkbox",
      "score": 11.45,
      "validation": "✓ VERIFIED in controls.csv"
    },
    {
      "element_id": "submit_button",
      "element_name": "Submit",
      "control": "ButtonAdv",
      "control_alias": "ButtonAdv",
      "skill": "syncfusion-winforms-button",
      "skill_hint": "syncfusion-winforms-button",
      "score": 10.89,
      "validation": "✓ VERIFIED in controls.csv"
    }
  ]
}
```

### Output Field Definitions

**Control Mapping Output Structure:**

| Field | Type | Description | Purpose |
|-------|------|-------------|---------|
| `element_id` | string | Unique identifier from input (snake_case) | Element reference |
| `element_name` | string | Display name from input | UI labeling |
| `control` | string | **Syncfusion WinForms control name from controls.csv (e.g., TextBoxExt, ButtonAdv, DataGrid)** | Code generation (control instantiation) |
| `control_alias` | string | Original control name from mapping for reference | Control name reference |
| `skill` | string | Skill reference label (e.g., `syncfusion-winforms-button`) | BM25 search result & reference |
| `skill_hint` | string | Reference label for Stage 5 NuGet conversion (e.g., `syncfusion-winforms-button`) | NuGet package lookup |
| `score` | number | BM25 relevance score (0-100+) | Match confidence (40+ = excellent) |
| `validation` | string | **"✓ VERIFIED in controls.csv"** or **"✗ NOT FOUND in controls.csv"** | Verification status |

**Control Mapping Template (for reference):**

```json
{
  "element_id": "email_input",
  "element_name": "Email Address",
  "control": "TextBoxExt",
  "control_alias": "TextBoxExt",
  "skill": "syncfusion-winforms-textbox",
  "skill_hint": "syncfusion-winforms-textbox",
  "score": 13.24,
  "validation": "✓ VERIFIED in controls.csv"
}
```

| Template Field | Description |
|----------------|-------------|
| `element_id` | Unique identifier for the UI element (snake_case) |
| `element_name` | Human-readable name of the element |
| `control` | **Syncfusion WinForms control name from controls.csv** (e.g., TextBoxExt, ButtonAdv, DataGrid, CheckBoxAdv) |
| `control_alias` | Reference to original control name from mapping |
| `skill` | Skill reference label (e.g., `syncfusion-winforms-textbox`) |
| `skill_hint` | Reference label for Stage 5 to convert to official NuGet package names (e.g., `syncfusion-winforms-textbox`) |
| `score` | BM25 ranking score (higher = better match, range 0-100+) |
| `validation` | **"✓ VERIFIED in controls.csv"** when control exists in CSV, or **"✗ NOT FOUND in controls.csv"** when fallback to standard WinForms control |

**Critical: `control` vs `skill` vs `skill_hint`**

- ✅ `control`: **Syncfusion WinForms control name** from controls.csv (e.g., `TextBoxExt`, `DataGrid`, `ButtonAdv`) - used directly in C# code generation
- ✅ `skill`: **Reference label** for skill file lookup (e.g., `syncfusion-winforms-textbox`) - used to find skill documentation
- ✅ `skill_hint`: **Reference label for Stage 5 NuGet conversion** (e.g., `syncfusion-winforms-textbox`) - used to find official Syncfusion NuGet package names
- ⚠️ **These are DIFFERENT values** - `control` is for code generation, `skill` and `skill_hint` are for package lookup
- ✅ Stage 5-7 convert `skill_hint` labels → official NuGet package names (e.g., `syncfusion-winforms-textbox` → `Syncfusion.SfInput.WinForms` from control skill SKILL.md)

**Examples of Conversion:**
- `control: TextBoxExt` + `skill_hint: syncfusion-winforms-textbox` → NuGet: `Syncfusion.Shared.Base` (from `.apm/skills/syncfusion-winforms-textbox/SKILL.md`)
- `control: DataGrid` + `skill_hint: syncfusion-winforms-datagrid` → NuGet: `Syncfusion.SfDataGrid.WinForms` (from `.apm/skills/syncfusion-winforms-datagrid/SKILL.md`)
- `control: ButtonAdv` + `skill_hint: syncfusion-winforms-button` → NuGet: `Syncfusion.Shared.Base` (from control skill file)

---

## Status

✅ **FULLY AUTOMATED** - No user interaction
✅ **Single pass** - control-mapping.json created once, controls mapped immediately
✅ **Token efficient** - No duplication or variant selection overhead
✅ **Data-driven** - BM25 semantic search on 100+ Syncfusion WinForms controls
✅ **Namespace Resolution** - Automatic mapping to Syncfusion.Windows.Forms.* and System.Windows.Forms namespaces
✅ **.NET Standard Compatible** - All controls and mappings target .NET Standard / .NET Framework compatibility
✅ **Ready for Stage 5** - Control + namespace mapping feeds directly to code generation

---

## Architecture

- **Input**: User requirements + control type from Stage 1
- **Processing**: 
  - Control analysis → JSON structure with `type_hint`
  - ControlMapper (BM25 algorithm on  100+ Syncfusion WinForms controls)
  - Namespace resolution for .NET Standard compatibility
- **Output**: 
  - `control-mapping.json` (project root) - layout structure + namespaces for Stage 4 & 5
  - Chat summary table - element count, controls matched, namespaces resolved
   "Controls Selected: [ControlName1], [ControlName2], [ControlName3]"
- **Data sources**: 
  - `scripts/controls.csv` (Syncfusion WinForms controls with namespaces)
  - `scripts/controls_search.cjs` (BM25 mapper)
- **Artifacts**: `control-mapping.json` (persistent artifact, reused by Stage 5)
- **Context**: Control + icon mapping results kept in conversation (no file artifact)
- **Target Framework**: .NET Standard / .NET Framework compatible WinForms controls
- **Namespaces**: Syncfusion.Windows.Forms.*, System.Windows.Forms

---

## Stage 3-4 Icon Integration Summary

| Step | Task | Input | Output | Artifact |
|------|------|-------|--------|----------|
| **Stage 3** | Analyze requirements, create control-mapping.json | User requirements + control type | `control-mapping.json` with elements | ✅ `control-mapping.json` |
| **Stage 4** | Map elements to Syncfusion WinForms controls (script-based) | `control-mapping.json` | Control mapping results | Context only (no file) |
| **Stage 5** | Generate code with controls | `control-mapping.json` + control mapping from context | WinForms `.cs` with controls + styling | ✅ Control files |
