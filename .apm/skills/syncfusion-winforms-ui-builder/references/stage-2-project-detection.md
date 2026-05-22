# Stage 2: Project Detection

**Purpose:** Auto-detect WinForms project structure, framework, language, and configuration to ensure generated code integrates seamlessly.

---

## ⚠️ CRITICAL: Project Folder and Solution File Structure

**MANDATORY:** Projects must be organized in a dedicated project folder with a solution file (.sln), not scattered in the workspace root.

### Project Organization Rules (REQUIRED)

**Correct Structure:**
```
D:\Workspace\                              ← workspace root
├── .codestudio/                           ← skills directory (config)
├── InventoryApp/                          ← PROJECT FOLDER (dedicated)
│   ├── InventoryApp.sln                   ← Solution file (MANDATORY)
│   ├── InventoryApp.csproj                ← Project file
│   ├── Program.cs
│   ├── Forms/
│   │   └── InventoryManagementForm.cs
│   ├── bin/
│   └── obj/
└── control-mapping.json                   ← Workspace-level artifact
```

**INCORRECT Structure (Anti-Pattern - DO NOT USE):**
```
D:\Workspace\                              ← ❌ Project files scattered here
├── InventoryApp.csproj                    ← ❌ Should be in dedicated folder
├── Program.cs                             ← ❌ Should be in dedicated folder
├── Form1.cs                               ← ❌ Should be in dedicated folder
└── control-mapping.json
```

---


## Pre-Detection Check: Project Existence Validation

**Step 1: Scan for Existing WinForms Project**
- Search for `.csproj` file in workspace
- Search for `Program.cs` and `Form1.cs` (WinForms entry points)
- **Result:** Project found → Proceed to auto-detect project settings
- **Result:** Project NOT found → Proceed to Step 2

**Step 2: If NO Project Found - Create New WinForms Template**

**MANDATORY: Create project in dedicated folder with .sln file**

1. **Determine Project Name** 
   - Ask user for desired project name (e.g., "InventoryApp", "SalesDashboard")
   - Validate: Name contains only alphanumeric characters and no spaces
   
2. **Create Dedicated Project Folder**
   - Execute: `mkdir <ProjectName>`
   - Navigate into: `cd <ProjectName>`
   
3. **Create WinForms Project in Dedicated Folder**
   - Execute: `dotnet new winforms -n <ProjectName>`
   - **Expected:** `.csproj`, `Program.cs`, `Form1.cs` created in dedicated folder
   
4. **Create Solution File (.sln) - MANDATORY**
   - Execute: `dotnet new sln -n <ProjectName>`
   - Add project: `dotnet sln <ProjectName>.sln add <ProjectName>.csproj`
   - Verify: `dotnet sln list` shows the project
   
5. **Verify Build**
   - Execute: `dotnet build`
   - Should complete without errors
   
6. **Proceed to Auto-Detection**
   - Continue with project settings detection

---

## Project Auto-Detection

**Once project exists, proceed with auto-detection:**

1. **Project Structure Validation**
   - Verify `.sln` file exists alongside `.csproj`
   - Verify project files are in dedicated folder (NOT in workspace root)
   - If validation fails → Use "Project Creation" section to fix structure before proceeding
   
2. **Framework Type**
   - Scan for: `.csproj`, `.sln`, `packages.config`, `Directory.Build.props`
   - Detect: WinForms (.NET Framework), WinForms (.NET Core/8+), WinForms (.NET Standard)
   
3. **Language Preference**
   - Check for: `.csproj` → C# language version (e.g., `<LangVersion>latest</LangVersion>`)
   - Check for: `.cs` files in the project
   - Default: C# (modern version if available in .csproj)
   
4. **UI Styling Strategy**
   - Check for: Theme files in Resources folder
   - Check for: Custom color/font schemes in app.config or settings
   - Default: System.Drawing for colors and styling
   
5. **Forms/Controls Directory**
   - Common paths: `Forms/`, `Controls/`, `UI/`, root folder
   - Find existing UserControl and Form patterns
   
6. **Formatting Rules**
   - Read `.editorconfig` for indent, line length, charset preferences
   - Read `.csproj` for code analysis rules
   - Apply same C# naming conventions and formatting to generated code
   
7. **Syncfusion License & Package Versioning**
   - Check: Is `SYNCFUSION_LICENSE_KEY` in app.config or environment variables?
   - Check: Does project already call `Syncfusion.Licensing.SyncfusionLicenseProvider.RegisterLicense()`?
   - Prompt: If missing, ask user for license key
   
8. **Syncfusion Package Version Detection**
   - **Scan `.csproj` and `packages.config` for existing Syncfusion NuGet packages:**
     - If `Syncfusion.*.WinForms` exists: Extract version (e.g., `33.2.3`)
     - Use SAME version for all new Syncfusion packages → Prevents version conflicts
   - **If NO existing Syncfusion packages found:**
     - Use latest stable version for all new packages
   - **Document version decision:** Log detected version in stage output
   
**User Interaction:**
Ask user to confirm or override detected settings:
```
===== DETECTION RESULTS =====
✓ Framework: WinForms (.NET 8)
✓ Language: C# 11
✓ UI Styling: System.Drawing
✓ Forms Dir: Forms/
✓ Formatting: 4 spaces, PascalCase for classes
✓ Syncfusion Version: 33.2.3 (detected from .csproj)
  OR
✓ Syncfusion Version: Latest (no existing packages)

[Confirm] [Override] [Cancel]
```

**Status:** User decides whether to accept detected settings or override them.
- If confirmed: Stage 7 (Dependencies) will use detected version for ALL new Syncfusion packages
- If overridden: User can specify custom version or latest for Syncfusion NuGet packages
