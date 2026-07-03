# Stage 2: Project Detection & Setup

**Purpose:** Detect or create the WinForms project, enforce standardized structure, read skill files to identify dependencies, install only validated packages, and confirm a clean build before Stage 3 begins.

---

## Step 1: Detect Existing Project

Scan workspace root and subdirectories for:
- `.sln` file
- `<ProjectName>.csproj`
- `Program.cs` and `Form1.cs`

| Result | Action |
|---|---|
| Solution + project found | → Proceed to Step 3 (Auto-Detection) |
| Not found | → Proceed to Step 2 (Create Project) |

---

## Step 2: Create New WinForms Project (if missing)

### 2a. Collect Project Name
- Ask user for a project name (alphanumeric only; auto-replace spaces with underscores)

### 2b. Scaffold Project (Primary Commands)

**MANDATORY: Projects must be organized in a dedicated project folder with a solution file (.sln), not scattered in the workspace root.**

Navigate to the desired workspace root and run:
```bash
dotnet new winforms -n <ProjectName>
```

This scaffolds the complete WinForms template inside a dedicated project folder:
```
<ProjectName>/
├── <ProjectName>.csproj
├── Program.cs
├── Form1.cs
├── Form1.Designer.cs
└── Form1.resx
```

### 2c. Verify Created Files
- `<ProjectName>.csproj` exist at projectRoot
- `Program.cs`, `Form1.cs` exist

⛔ **If any file is missing → report dotnet CLI error and halt.**

---

## Step 3: Auto-Detect Project Settings

| Setting | How to Detect | Default |
|---|---|---|
| Framework | Read `<TargetFramework>` in `.csproj` | `net8.0-windows` |
| Language | Read `<LangVersion>` in `.csproj` + presence of `.cs` files | C# (latest) |
| Styling strategy | Scan for theme/resource files in `Resources/` or `app.config` | System.Drawing |
| Control directory | Scan for `Forms/`, `Controls/`, `UI/` | `Forms/` |
| Code style | Read `.editorconfig` | Default C# naming conventions |
| Syncfusion version | Scan `.csproj` / `packages.config` for `Syncfusion.*.WinForms` package version | Latest stable (Step 5) |

---

## Step 4: Validate WinForms Compatibility

- Confirm `<UseWindowsForms>true</UseWindowsForms>` in `.csproj`
- Confirm target framework is `net462` or `net8.0-windows` (or newer)
- Confirm `Program.cs` has a `Main()` method calling `Application.Run()` — flag for Stage 5 to generate if missing
- ⛔ **BLOCK** if project targets a non-Windows framework (WinForms is Windows-only)

---

## Step 5: Apply Folder Structure

Create any missing folders inside `<ProjectName>/`:

```
<ProjectName>/
├── Models/           # Data models and DTOs
├── Forms/            # Top-level Form classes
├── Controls/         # Reusable UserControls
├── Services/         # Business logic and backend services
├── Repositories/     # Data access interfaces + in-memory implementations
└── Resources/        # Icons, color/spacing constants, theme assets
```

Verify / update `Program.cs`:
- `Main()` entry point must call `Application.Run(new <StartupForm>())`
- ❌ Do NOT hardcode Syncfusion visual styles per-control — themes are applied globally via `SkinManager`
- ✅ Only custom constants (colors, fonts, spacing) go in `Resources/`
- Syncfusion theme is bootstrapped in `Program.cs` `Main()`, before `Application.Run()`:
  ```csharp
  Syncfusion.WinForms.Controls.SkinManager.SetVisualStyle(SkinManager.VisualStyles.Office2019Colorful);
  ```
- Refer to `references/stage-4-theming-and-design-system.md` for the full resource structure

---

## Step 6: Read Skill Files & Detect Dependencies (MANDATORY — Before Any Package Installation)

⛔ **CRITICAL RULE: Do NOT assume, guess, or hardcode package names. All dependencies MUST come from skill files.**

### 6a. Identify Required Skill Files
- Read `control-mapping.json` (from Stage 3 if already available) or infer from Stage 1 intent
- For each mapped control, locate the skill folder:
  `<skills-root>/syncfusion-winforms-<control-name>/`
  where `<skills-root>` is one of: `.codestudio/skills`, `.agent/skills`, `.agents/skills`, `.github/skills`, `skills`

### 6b. Extract from Each Skill File (Mandatory Pre-Installation Step)
**Do NOT proceed without completing all sub-steps:**
- Read: `<skill-folder>/SKILL.md` or `<skill-folder>/references/getting-started.md`
- Extract **exactly** as documented:
  - ✅ Official NuGet package name (copy verbatim, e.g., `Syncfusion.SfDataGrid.WinForms`, NOT inferred names)
  - ✅ Minimum or pinned version (if skill file specifies)
  - ✅ Any dependent packages (listed under Prerequisites)
- If skill file is missing or package name is ambiguous → **HALT and report error**

### 6c. Validate & Resolve Version Strategy

| Scenario | Version Rule |
|---|---|
| Existing Syncfusion packages in `.csproj` / `packages.config` | Compare to latest stable; suggest upgrade if gap > 2 minor versions; lock user's choice for ALL packages |
| No existing Syncfusion packages | Query latest stable: `dotnet package search Syncfusion.Shared.Base --exact`; use that version for ALL packages |

- ❌ Do NOT install outdated versions
- ❌ Do NOT install packages not listed in a skill file
- ✅ All Syncfusion packages must use the **same resolved version**
- Log resolved version as: `resolved_syncfusion_version: "<version>"`

> **Note:** All required NuGet packages, including dependent/prerequisite packages, must be cross-verified and installed in accordance with the official [Syncfusion Windows Forms Control Dependencies documentation](https://help.syncfusion.com/windowsforms/control-dependencies). Skill files take precedence for control-specific package names and versions, but the dependency documentation must be used to confirm no required dependent assembly is missing before proceeding to Step 7.

---

## Step 7: Install Validated Packages

Install only packages extracted from skill files in Step 6, using the resolved version:

```bash
# Core packages (always required — extracted from skill files)
dotnet add package Syncfusion.Shared.Base       --version <resolved_version>
dotnet add package Syncfusion.Licensing         --version <resolved_version>

# Control-specific packages (from skill files only — examples)
dotnet add package Syncfusion.SfDataGrid.WinForms  --version <resolved_version>
dotnet add package Syncfusion.Tools.Windows        --version <resolved_version>

# Theme package (deferred — installed after Stage 4 confirms theme/visual style)
# dotnet add package Syncfusion.<ThemeName>.WinForms --version <resolved_version>
```

Syncfusion license handling:
- Check `Program.cs` and `SYNCFUSION_LICENSE_KEY` env var
- Missing → prompt: *"Get a free Community License at https://www.syncfusion.com/account/manage-trials"*
- Provided → inject into `app.config` (or env var) + add `SyncfusionLicenseProvider.RegisterLicense()` to `Program.cs` `Main()` before `Application.Run()`
- Skipped → warn that a watermark will appear on Syncfusion controls

Run restore and confirm no errors:
```bash
dotnet restore
```

⛔ **If restore fails → report error and halt.**

---

## Step 8: Validate Build

```bash
dotnet build
```

- **PASS (0 errors)** → Lock settings and proceed to Stage 3
- **FAIL** → Report errors, attempt auto-fix (missing references, namespace issues), retry once; halt if still failing

---

## Validation Checklist

| # | Check | Rule |
|---|---|---|
| 1 | `projectRoot` structure correct | `.sln` and `.csproj` both exist at projectRoot; `.sln` links the `.csproj` |
| 2 | Solution structure correct | `dotnet sln` links `.csproj`; no orphaned projects |
| 3 | WinForms compatibility confirmed | `<UseWindowsForms>true</UseWindowsForms>` + Windows target framework |
| 4 | Folder structure created | `Models/`, `Forms/`, `Controls/`, `Services/`, `Repositories/`, `Resources/` |
| 5 | Skill files read before install | No package installed without reading its `getting-started.md` |
| 6 | Only skill-validated packages installed | Zero assumed or hardcoded package names |
| 7 | Single Syncfusion version locked | All packages use identical `resolved_syncfusion_version` |
| 8 | Latest stable version used | No outdated or pinned-to-old versions |
| 9 | Syncfusion theme applied globally | Set via `SkinManager` in `Program.cs`, not hardcoded per-control |
| 10 | Build passes | `dotnet build` exits with 0 errors |

> **Resource integrity checks** (resource file references, color constants, designer-binding consistency, event handler wiring) are flagged here and auto-fixed by Stage 5 during code generation.

---

## User Confirmation

Present locked settings before proceeding to Stage 3:

```
✓ Solution:            <ProjectName>/<ProjectName>.sln
✓ Project:             <ProjectName>/<ProjectName>.csproj
✓ Framework:           .NET 8 (net8.0-windows)
✓ Language:            C# (latest)
✓ Control Directory:   Forms/
✓ Skill Files Read:    syncfusion-winforms-datagrid, syncfusion-winforms-textinputlayout, ...
✓ Syncfusion Version:  <resolved_version> (latest stable)
✓ Program.cs:          Main() with Application.Run() present
✓ Folder Structure:    Models/ Forms/ Controls/ Services/ Repositories/ Resources/
✓ Build:               PASS

[Confirm] [Override] [Cancel]
```

- **Confirm** → Stage 3 proceeds with all settings locked
- **Override** → User specifies custom values (version, directory, theme)
- **Cancel** → Halt; do not proceed
