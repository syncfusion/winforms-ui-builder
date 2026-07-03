# Stage: Control Skill Extraction

**Purpose:** Extract, structure, and persist all required control data from skill reference files into `skill-extraction.json` before any code is generated. This artifact is the single, verified source of truth for `using` directives, properties, events, and packages used throughout Stage 5.

**Trigger:** Runs immediately after the Stage 5 atomic validation gate clears (Step 2).  
**Input:** `<project-root>/control-mapping.json` + skill reference files per control  
**Output:** `<project-root>/skill-extraction.json` — fully validated artifact

⛔ **Stage 5 code generation does NOT begin until `skill-extraction.json` exists and `validation_status` = `"PASS"`.**

---

## Why This Stage Exists

| Failure Mode Without This Stage | Consequence |
|---|---|
| Extracted data lives only in transient context | Code generation silently falls back to guessed `using` directives and type names |
| No shared artifact across deliverables | Designer.cs, code-behind, and `.resx` files use inconsistent APIs |

`skill-extraction.json` eliminates both: all data is explicit, persistent, and locked before the first file is written.

---

## Input Sources (MANDATORY — Read in This Order)

For each control in `control-mapping.json`, files are read from:
```
<skill-root>/                    ← SKILL.md  (PRIMARY — read first)
<skill-root>/references/         ← all .md reference files (SECONDARY — read after SKILL.md)
```
where `<skill-root>` = `<skills-root>/syncfusion-winforms-<control-name>/`  
and `<skills-root>` is one of: `.codestudio/skills`, `.agent/skills`, `.agents/skills`, `.github/skills`, `skills`

**Read order (mandatory):**

| Priority | File | Purpose |
|---|---|---|
| **0 — PRIMARY** | `<skill-root>/SKILL.md` | Control overview, which references to use, feature structure, expected APIs |
| **1 — MANDATORY** | `<skill-root>/references/getting-started.md` | Core `using` directive, base properties, events, NuGet package |
| **2 — ALL OTHERS** | All remaining `.md` files in `/references/` | Additional properties, events, methods — guided by SKILL.md |

> ⛔ **Do NOT read any reference file before SKILL.md is processed.** SKILL.md determines which reference files are relevant and how to interpret them.

### Reference File Detection (DYNAMIC — NOT Fixed Filenames)

⛔ **Do NOT assume fixed filenames.** Real files may be named differently (e.g., `data-filter.md`, `input-validation.md`, `theme-guide.md`). Use SKILL.md to identify relevant files, then scan all remaining files in `/references/`.

**Step B — After SKILL.md is read, scan ALL files in `/references/` folder:**
- List every `.md` file in the directory
- Use SKILL.md guidance to prioritize which files cover which features
- Read each file regardless of its name
- Categorize by content using keyword detection:

| Category | Keyword Signals in File Content | What to Extract |
|---|---|---|
| **Core Setup** | `using`, `namespace`, `NuGet`, `PackageReference`, `assembly`, `InitializeComponent` | `using` directive, package name, base setup |
| **Filtering** | `filter`, `search`, `query`, `predicate` | Filter properties, filter events |
| **Validation** | `validation`, `error`, `invalid`, `required`, `rule` | Validation properties, error events |
| **Styling** | `style`, `theme`, `appearance`, `BackColor`, `ForeColor`, `SkinManager`, `Font` | Style properties, theme API calls |
| **Other** | Any file not matching above | Read fully — extract any properties, events, or methods found in code examples |

**Rules:**
- ✅ Always read SKILL.md before any reference file
- ✅ Use SKILL.md to identify which reference files are relevant
- ✅ Read every `.md` file in `/references/` — no file is skipped based on name
- ✅ Categorize by content, not filename
- ❌ Do NOT read reference files without SKILL.md guidance
- ❌ Do NOT treat `getting-started.md` as the only source — all reference files are potential sources

---

## Extraction Workflow (Execute in Order)

---

### Step 0: Read SKILL.md (PRIMARY — MANDATORY BEFORE ANY OTHER FILE)

For each control, read `<skill-root>/SKILL.md` first — before `getting-started.md` or any reference file.

```
LOCATE: <skill-root>/SKILL.md
  ❌ NOT found → HALT: "Skill definition missing for <control-name> — cannot proceed with extraction"

READ and extract:
  ✅ Control overview and purpose
  ✅ Which reference files cover which features (use as a guide for Step B scanning)
  ✅ Feature-level documentation structure
  ✅ High-level expected APIs (used to cross-validate Step 3 extraction)

RECORD: skill_md_read = true in skill-extraction.json entry
```

⛔ **If reference files are read before SKILL.md → HALT: "Reference files read without SKILL.md guidance — re-run Step 0 first"**

---

### Step 1: Collect Controls from `control-mapping.json`

1. Read `<project-root>/control-mapping.json`
2. Collect every unique `control` value from `mapped_controls[]`
3. Skip entries where `control == "NATIVE_WINFORMS"` — record in `native_winforms_controls[]`, no extraction needed
4. Build extraction list (e.g., `["SfDataGrid", "SfComboBox", "ButtonAdv"]`)

⛔ `control-mapping.json` missing or invalid → HALT. Return to Stage 3.

---

### Step 2: `using` Directive Extraction (CRITICAL)

For each control, resolve the C# `using` directive using this **strict priority order**:

```
1. Read getting-started.md → search for using directives in C# code examples
   IF exact "using Syncfusion.<Namespace>;" found in code examples → USE IT (highest priority)

2. Read getting-started.md → check for assembly-qualified type references
   IF a full type name (e.g., Syncfusion.WinForms.DataGrid.SfDataGrid) is declared → DERIVE the using from it

3. Scan ALL other .md files in /references/ folder
   IF a more specific using directive is used in any file's C# examples → USE the more specific one

RESULT RULES:
  ✅ Use the using directive exactly as written in the skill file
  ✅ If both a short namespace and a fully-qualified type appear → prefer the explicit using directive (more reliable)
  ❌ Do NOT guess, infer, or construct a namespace from the control name
  ❌ Do NOT hardcode any namespace not found in a reference file
  ❌ If no using directive found in any file in /references/ → HALT: "Namespace undefined for <control-name>"
```

**No default `using` directive fallback exists.** Every namespace must be sourced directly from a skill file reference.

---

### Step 3: Property & Event Extraction (MANDATORY)

Extract from **all** `.md` files found in `/references/` — determined by the dynamic scan, not by fixed filenames.

**Process per file:**
1. Locate all C# and Designer.cs code examples in the file
2. Extract every property name assigned on the control in those examples
3. Extract every event name wired in those examples
4. Extract every method called on the control instance
5. Cross-reference: if a property appears in a Designer.cs or code-behind example but is not listed in a documented API table → **still include it** (usage in examples is evidence of validity)
6. Record which file each item was found in (`source` field)

**Rules:**
- ✅ Include properties, events, and methods found in any file in `/references/`
- ✅ Copy names character-for-character (casing matters in C# identifiers)
- ✅ Mark which file each item was sourced from (aids debugging)
- ❌ Do NOT assume a property exists because it sounds logical
- ❌ Do NOT include items only seen in third-party blog posts or non-skill sources
- ❌ Do NOT stop after `getting-started.md` — scan every file in the folder

**Blocking validation for Step 3:**

| Condition | Action |
|---|---|
| Reference files exist in `/references/` but were not all scanned | ⛔ HALT: `"Incomplete reference analysis — not all files in /references/ were read"` |
| Extraction sourced only from `getting-started.md` when other files exist | ⛔ HALT: `"Partial extraction not allowed — re-scan all files in /references/"` |
| Properties/events included without evidence from a code example | ⛔ HALT: `"Unverified API extraction — must be sourced from actual usage examples"` |

---

### Step 4: Validate Extracted Data (BLOCKING)

Run these checks immediately after extracting each control — before writing to the JSON file:

| Condition | Halt Message |
|---|---|
| Control not in `control-mapping.json` | `"Control <name> not in mapping — extraction not requested"` |
| Skill folder not found | `"Skill folder missing for <control-name> — cannot extract"` |
| `getting-started.md` not found | `"Reference file missing for <control-name> — cannot extract"` |
| `using` directive not found in any reference file | `"Namespace undefined for <control-name> — do not guess"` |
| `valid_properties` empty after reading all files | `"No properties found for <control-name> — check all reference files"` |
| NuGet package name not listed | `"Package undefined for <control-name> — do not assume"` |

---

### Step 5: Write `skill-extraction.json`

Save to `<project-root>/skill-extraction.json` after all controls pass Step 4.

#### Full Schema

```json
{
  "generated_at": "<ISO-8601 timestamp>",
  "resolved_syncfusion_version": "<version from Stage 2>",
  "controls": [
    {
      "control": "SfDataGrid",
      "skill_md_read": true,
      "using_directive": "using Syncfusion.WinForms.DataGrid;",
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
      "setup_instructions": "Set DataSource before InitializeComponent completes. Apply SkinManager theme after form load, not inside InitializeComponent.",
      "advanced_features_read": ["filtering.md", "styling.md"],
      "sources_read": [
        ".codestudio/skills/syncfusion-winforms-sfdatagrid/references/getting-started.md",
        ".codestudio/skills/syncfusion-winforms-sfdatagrid/references/filtering.md",
        ".codestudio/skills/syncfusion-winforms-sfdatagrid/references/styling.md"
      ]
    },
    {
      "control": "SfComboBox",
      "skill_md_read": true,
      "using_directive": "using Syncfusion.WinForms.ListView;",
      "namespace_source": "getting-started.md",
      "nuget_package": "Syncfusion.SfListView.WinForms",
      "nuget_version": "Latest",
      "valid_properties": [
        { "name": "DataSource",           "source": "getting-started.md" },
        { "name": "DisplayMember",        "source": "getting-started.md" },
        { "name": "ValueMember",          "source": "getting-started.md" },
        { "name": "DropDownStyle",        "source": "advanced-features.md" },
        { "name": "AutoCompleteMode",     "source": "advanced-features.md" }
      ],
      "valid_events": [
        { "name": "SelectedIndexChanged", "source": "getting-started.md" }
      ],
      "valid_methods": [],
      "setup_instructions": "Assign DataSource after InitializeComponent. Wire SelectedIndexChanged for value tracking.",
      "advanced_features_read": ["advanced-features.md"],
      "sources_read": [
        ".codestudio/skills/syncfusion-winforms-sfcombobox/references/getting-started.md",
        ".codestudio/skills/syncfusion-winforms-sfcombobox/references/advanced-features.md"
      ]
    },
    {
      "control": "ButtonAdv",
      "skill_md_read": true,
      "using_directive": "using Syncfusion.WinForms.Controls;",
      "namespace_source": "getting-started.md",
      "nuget_package": "Syncfusion.Shared.WinForms",
      "nuget_version": "Latest",
      "valid_properties": [
        { "name": "Text",            "source": "getting-started.md" },
        { "name": "ImageList",       "source": "getting-started.md" },
        { "name": "ImageIndex",      "source": "getting-started.md" },
        { "name": "UseVisualStyle",  "source": "styling-and-themes.md" },
        { "name": "ButtonStyle",     "source": "styling-and-themes.md" }
      ],
      "valid_events": [
        { "name": "Click",           "source": "getting-started.md" }
      ],
      "valid_methods": [],
      "setup_instructions": "Set Text for button label. Apply theme via SkinManager before form is shown, not inside InitializeComponent.",
      "advanced_features_read": ["styling-and-themes.md"],
      "sources_read": [
        ".codestudio/skills/syncfusion-winforms-buttonadv/references/getting-started.md",
        ".codestudio/skills/syncfusion-winforms-buttonadv/references/styling-and-themes.md"
      ]
    }
  ],
  "native_winforms_controls": [
    {
      "control": "NATIVE_WINFORMS",
      "element_id": "status_label",
      "equivalent_native": "System.Windows.Forms.Label",
      "fallback_reason": "No Syncfusion label control available"
    }
  ],
  "extraction_status": "COMPLETE",
  "validation_status": "PASS"
}
```

#### File Rules
- One entry per unique control — no duplicates
- `valid_properties`, `valid_events`, `valid_methods` use objects with `name` + `source` — never bare strings
- `sources_read[]` lists every `.md` file actually opened for this control — filenames are dynamic, not predetermined
- `advanced_features_read[]` records which non-`getting-started.md` files contributed data, identified by their actual filename (not assumed category name)
- `NATIVE_WINFORMS` entries go in `native_winforms_controls[]` only — no `using` directive extraction

---

### Step 6: Validate `skill-extraction.json` (MANDATORY)

Run all checks before Stage 5 proceeds:

| # | Check | Fail Condition |
|---|---|---|
| 0 | SKILL.md read for every control | Any entry where `skill_md_read` ≠ `true` |
| 1 | All controls extracted | Any `control-mapping.json` entry missing from `controls[]` |
| 2 | No missing `using` directives | Any entry with empty or null `using_directive` |
| 3 | `using` directive has a `namespace_source` | Any entry where `namespace_source` is missing or null |
| 4 | No missing packages | Any entry with empty or null `nuget_package` |
| 5 | Properties sourced | Any property object missing a `source` field |
| 6 | Version consistent | Any `nuget_version` not matching `resolved_syncfusion_version` |
| 7 | File is valid JSON | Any parse error (trailing comma, missing bracket, etc.) |
| 8 | `extraction_status` = `"COMPLETE"` | Status missing or set to partial/error |

**On any failure:**
- Report every failing check with control name + field
- ⛔ HALT — do not allow Stage 5 to begin
- Prompt: *"Re-read skill files for `<control-name>` and re-run Stage — Control Skill Extraction for that control only"*

**On full pass:**
- Set `validation_status: "PASS"` in the file
- Output: *"`skill-extraction.json` validated — all [N] controls verified. Proceeding to Stage 5 code generation."*

---

## Usage in Stage 5 (CRITICAL)

Stage 5 must read `skill-extraction.json` before generating any file. It must never derive `using` directives, properties, or events from memory or training data.

### Lookup Pattern (mandatory for every deliverable)

```
READ <project-root>/skill-extraction.json
  ❌ File not found → HALT: "skill-extraction.json missing — run Stage — Control Skill Extraction first"
  ❌ validation_status ≠ "PASS" → HALT: "Extraction not validated — fix and re-run Stage — Control Skill Extraction"

FOR EACH control used in this deliverable:
  FIND controls[] entry where control == "<ControlName>"
  ❌ Not found → HALT: "No extraction entry for <ControlName> — re-run Stage — Control Skill Extraction"

  USE using_directive        → add exact using statement to Designer.cs and code-behind
  USE valid_properties[].name → only these property names are valid in Designer.cs assignments
  USE valid_events[].name    → only these event names are valid in Designer.cs wiring and code-behind handlers
  USE nuget_package          → pass to Stage 6 for dotnet add package
```

### Per-Deliverable Field Map

| Deliverable | Fields Used |
|---|---|
| Designer.cs | `using_directive` → `using` statement; `valid_properties[].name` → property assignments; `valid_events[].name` → event subscriptions |
| Code-behind | `valid_events[].name` → handler signatures; `setup_instructions` → constructor and `InitializeComponent()` ordering |
| `.resx` | `valid_properties[].name` → resource-backed property references (image lists, icons, strings) |
| Stage 6 (NuGet) | `nuget_package` + `nuget_version` → `dotnet add package` command |

---

## Blocking Conditions Summary

| Condition | Action |
|---|---|
| `SKILL.md` not read before reference files | ⛔ HALT — re-run Step 0 first |
| `skill-extraction.json` missing | ⛔ HALT Stage 5 — run Stage — Control Skill Extraction first |
| `validation_status` ≠ `"PASS"` | ⛔ HALT Stage 5 — fix and re-validate |
| Control needed in Designer.cs has no entry | ⛔ HALT — re-extract that control |
| `using` directive used in code not in entry | ⛔ HALT — directive is unverified |
| Property assigned in Designer.cs not in `valid_properties` | ⛔ HALT — property is unverified |
| Event wired in Designer.cs not in `valid_events` | ⛔ HALT — event is unverified |
| `namespace_source` missing | ⛔ HALT — `using` directive origin cannot be traced |

---

## Optional: Validation Script Enhancement

If a `controls_search.cjs` script is used (Stage 3), update it to:
- Scan the entire `/references/` directory dynamically (not a fixed file list)
- Categorize files by content keywords (see Input Sources table above)
- Validate extracted APIs against all scanned files — not just `getting-started.md`
- Report which file each extracted property/event came from

---

## Stage — Control Skill Extraction Output Summary

| Artifact | Location | Consumed By |
|---|---|---|
| `skill-extraction.json` | `<project-root>/` | Stage 5 (all deliverables), Stage 6 (NuGet install) |

✅ All `using` directives traced to a specific reference file — no guesses  
✅ Properties and events sourced from all `.md` files in `/references/` — not fixed filenames  
✅ File categorization is content-based — real filenames may differ from expected patterns  
✅ Single artifact shared across Designer.cs, code-behind, `.resx`, and NuGet steps  
✅ Re-runnable per control — one failed extraction does not require a full restart
