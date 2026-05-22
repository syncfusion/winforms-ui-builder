# Stage 6: Dependencies

**Purpose:** Detect required NuGet packages, resolve version conflicts, prepare dotnet add package command.

---

## ⚠️ CRITICAL: Convert Skill Names to Officially Documented Syncfusion NuGet Packages

**Stage 3 generates control mappings using REFERENCE LABELS** (e.g., `syncfusion-winforms-datagrid` from `skill_hint` field).
**Stage 6 MUST map these reference labels to OFFICIAL Syncfusion NuGet package names explicitly documented in control skill files or controls.csv.**

**DO NOT assume package names follow patterns** (e.g., ❌ appending `.WinForms` to control names).

**Verification Process (REQUIRED):**
1. **Extract all Syncfusion skill names from Stage 3 output:**
   - Get `skill_hint` values from Stage 3 control mapping results
   - Example: `syncfusion-winforms-button`, `syncfusion-winforms-datagrid`, `syncfusion-winforms-chart`
   - Source: From controls_search.cjs output (kept in chat from Stage 3)

2. **For EACH skill name, locate the mapping source:**
   - **FIRST**: Check `controls.csv` in `/scripts/` directory:
     - Skill Name (column) → NuGet Package column mapping
     - Example: `syncfusion-winforms-datagrid` → `Syncfusion.SfDataGrid.WinForms`
   - **SECOND**: Look for corresponding control skill file:
     - `.codestudio/skills/syncfusion-winforms-<controlname>/SKILL.md`
     - `.agent/skills/syncfusion-winforms-<controlname>/SKILL.md`
     - `.agents/skills/syncfusion-winforms-<controlname>/SKILL.md`
     - `.github/skills/syncfusion-winforms-<controlname>/SKILL.md`
     - `skills/syncfusion-winforms-<controlname>/SKILL.md`
   - **THIRD**: Consult official Syncfusion documentation if neither above exists:
     - [Syncfusion WinForms Controls](https://help.syncfusion.com/windowsforms/introduction/overview)

3. **Verify the NuGet package name is EXPLICITLY documented in at least one source:**
   - The control's SKILL.md file under `NuGet Package` or getting-started section, OR
   - `controls.csv` skill-to-package mapping, OR
   - Official Syncfusion WinForms documentation

4. **Verify the package version is compatible with the project's target framework** (from Stage 2)

5. **REJECT any package name that is not explicitly documented** - do not infer or assume

**Mapping Examples (Verified Against controls.csv and Official Syncfusion Documentation):**

| Skill Reference (skill_hint from Stage 3) | Source | Correct Official NuGet Package | Notes | DO NOT USE |
|---|---|---|---|---|
| `syncfusion-winforms-datagrid` | controls.csv | `Syncfusion.SfDataGrid.WinForms` | For advanced data grid functionality | ❌ `DataGrid.WinForms` ❌ `Syncfusion.DataGrid.WinForms` |
| `syncfusion-winforms-chart` | controls.csv | `Syncfusion.Chart.Windows` | For chart visualization | ❌ `Chart.WinForms` |
| `syncfusion-winforms-button` | controls.csv | `Syncfusion.Core.WinForms` | Core Syncfusion assemblies | ❌ `Button.WinForms` |
| `syncfusion-winforms-textbox` | controls.csv | `Syncfusion.Core.WinForms` or `Syncfusion.Shared.Base` | Text input controls | ❌ `TextBox.WinForms` |
| `syncfusion-winforms-ribbon` | controls.csv | `Syncfusion.Tools.Windows` | For ribbon bar UI | ❌ `Ribbon.WinForms` |

**How to Use This Table:**
1. Get `skill_hint` value from Stage 3 control mapping output (e.g., `syncfusion-winforms-datagrid`)
2. Find matching row in table above
3. Use the "Correct Official NuGet Package" name in `dotnet add package` command
4. **NEVER use the "DO NOT USE" variations** — these are incorrect patterns

**Example Workflow:**
```
Stage 3 output: "skill_hint": "syncfusion-winforms-button"
           ↓
Table lookup: syncfusion-winforms-button → Syncfusion.Core.WinForms
           ↓
Stage 6 command: dotnet add package Syncfusion.Core.WinForms
```

---

**AI Should:**

1. **Map Skill Names to Officially Documented NuGet Packages:**
   - Extract Syncfusion skill names from generated code comments or imports
   - For EACH skill name, consult:
     - First: The corresponding control skill file (if available at `.codestudio/skills/syncfusion-winforms-<name>/SKILL.md`)
     - Second: Official Syncfusion WinForms documentation
   - **ONLY use package names that are EXPLICITLY documented** in control skill files or official Syncfusion resources
   - Verify package names are officially supported and match documented NuGet references exactly
   - Document mapping and source (skill file or official documentation) for audit trail
   - **REJECT** any inferred or assumed package names

2. **Check Project's .csproj File:**
   - What NuGet packages already installed?
   - What versions are currently in use?
   - What is the target framework?
   - Any version conflicts with .NET Framework or .NET Core compatibility?

3. **Resolve Conflicts:**
   - If Syncfusion package already installed:
     - Is version compatible with target framework?
     - Suggest upgrade if needed for .NET compatibility
   - If framework version conflicts:
     - Recommend target framework version (e.g., net8.0)
     - Check compatibility with all dependencies
   - Verify Syncfusion license registration is included

4. **Prepare Installation Command:**
   - Generate dotnet add package command using VERIFIED official package names
   - List packages to add with specific versions
   - List packages to upgrade (if needed)
   - Include package restoration command

**Example Output:**

```
✓ Dependency Analysis

Target Framework:
  .NET 8.0 (recommended for WinForms)

New Packages to Install:
  - Syncfusion.Core.WinForms (Latest)
  - Syncfusion.SfInput.WinForms (Latest)
  - Syncfusion.Shared.Base (Latest)
  - Syncfusion.Tools.Windows (Latest)

Existing Packages:
  ✓ Syncfusion.Licensing (compatible)
  ✓ System.ComponentModel.Composition (compatible)
  ✓ System.Windows.Forms (framework built-in)

Conflicts: None

Installation Commands (Windows):
$ dotnet add package Syncfusion.Core.WinForms
$ dotnet add package Syncfusion.SfInput.WinForms
$ dotnet add package Syncfusion.Shared.Base
$ dotnet add package Syncfusion.Tools.Windows

Or restore entire project:
$ dotnet restore
```

**User Interaction:**
User confirms NuGet installation or updates project file manually:
```
Ready to install dependencies?
[Install] [Show Command] [Edit .csproj] [Skip]
```

**Status:** AI detects and prepares. User decides whether to install now or update project file manually.
