# Stage 5: Code Generation

**Purpose:** Generate production-ready WinForms code (C#, Designer, Resource files) with accessibility and .NET standards compliance.

---

## ⚠️ CRITICAL: Verify Stage 3 Control Mappings

**Before generating any code, VALIDATE the control mappings from Stage 3:**

### Control Mapping Validation Checklist

1. **Verify mapping accuracy**:
   - For EACH mapped control, check that `validation` field is **"✓ VERIFIED in controls.csv"**
   - If any control shows **"✗ NOT FOUND in controls.csv"**, it means Stage 3 script could NOT find a Syncfusion match and fell back to native WinForms
   - Such fallbacks should be intentional and documented (e.g., use native TextBox if Syncfusion TextBoxExt unavailable)

2. **Review BM25 scores**:
   - Scores **40+**: Excellent mapping confidence → Use as-is
   - Scores **20-40**: Good mapping but verify context is correct
   - Scores **<20**: Weak mapping → Consider revising `type_hint` in control-mapping.json and re-running Stage 3 script
   - Scores **0**: No match → Verify fallback to native WinForms is acceptable for this element

3. **Cross-check control type against use case**:
   - Example: If element is `"email_input"` with `type_hint: "text input email"`, verify mapped control is TextBoxExt or similar text input, NOT a dropdown or other control type
   - Example: If element is `"data_grid"` with `type_hint: "grid data table sorting filtering"`, verify mapped control is DataGrid or GridControl, NOT ListView or TreeView
   - Mismatch indicates incorrect mapping → Go back to Stage 3, update control-mapping.json, and re-run script

4. **Check for unintended fallbacks**:
   - If a common control like Button, TextBox, or ComboBox shows "✗ NOT FOUND in controls.csv" (fallback to native WinForms), this indicates:
     - Type_hint keywords do NOT match controls.csv keywords, OR
     - control-mapping.json uses unrecognized keywords
   - Solution: Revise type_hint to include keywords from controls.csv (e.g., for button use `"button cta action click primary"`)

5. **Validate against user requirements**:
   - Re-read original user request and Stage 1 intent analysis
   - Ensure mapped controls fulfill all requirements mentioned
   - Example: User asked for "sortable data table" → Verify mapped control is SfDataGrid (supports sorting), NOT a static DataGrid

**If any validation failures found:**
- ❌ DO NOT proceed to code generation
- ✅ Go back to Stage 3: Update control-mapping.json with better type_hints
- ✅ Re-run `controls_search.cjs` script
- ✅ Verify all mappings now have `validation: "✓ VERIFIED in controls.csv"`
- ✅ Re-run BM25 scores to confirm all scores are adequate (40+ preferred)
- ✅ Only then proceed to Stage 5 code generation

---

## CRITICAL: Read control Skills BEFORE Code Generation

**THIS STEP IS NOT OPTIONAL - Must be completed before writing any code**

### Step 1: Identify All Control Skills from Stage 3

From Stage 3 output, extract ALL Syncfusion WinForms control skills used (e.g., `syncfusion-winforms-grid`, `syncfusion-winforms-buttons`, `syncfusion-winforms-chart`, etc.)

**CRITICAL: Namespace Discovery Protocol**
If the exact using statement or assembly for a control is not known:
1.  **Search for Skill**: Look for the control's `SKILL.md` using skill file discovery paths:
   - `.codestudio/skills/<skill-name>/SKILL.md`
   - `.agent/skills/<skill-name>/SKILL.md`
   - `.agents/skills/<skill-name>/SKILL.md`
   - `.github/skills/<skill-name>/SKILL.md`
   - `skills/<skill-name>/SKILL.md`
2.  **Refer to Documentation**: Read the `getting-started.md` or `SKILL.md` within discovered skill folders to extract the authoritative `using` statement and assembly references.
3.  **Mandatory Mapping**: Use the unique namespace mapping discovered in the skill to avoid "Control does not exist" errors.

### Step 2: Read Getting-Started for EACH control Skill

**For every single control skill identified in Step 1:**

1. **Read:** Skill file using discovery paths (in order of preference):
   - `.codestudio/skills/<skill-name>/references/getting-started.md` → First check here
   - `.agent/skills/<skill-name>/references/getting-started.md` → Fallback
   - `.agents/skills/<skill-name>/references/getting-started.md` → Fallback
   - `.github/skills/<skill-name>/references/getting-started.md` → Fallback
   - `skills/<skill-name>/references/getting-started.md` → Fallback
   - This is the ONLY authoritative source for namespace declarations, assembly references, properties, and setup
   - Do NOT generate C# code without reading this first
   
2. **Extract and document:**
   - NuGet package name (e.g., `Syncfusion.Grid.Windows`)
   - Exact using statement (e.g., `using Syncfusion.Windows.Forms.Grid;`)
   - Assembly name (e.g., `Syncfusion.Grid.Windows.dll`)
   - **Theme requirements (CRITICAL)** - which Syncfusion theme to load
   - Required initialization or setup code
   - Base class or inheritance requirements
   - License key requirement if applicable
   - Supported .NET Framework or .NET Core version

### Step 3: If Complex Features Needed

- Read: Skill file using discovery paths - `<skill-name>/SKILL.md` for complete API documentation:
  - `.codestudio/skills/<skill-name>/SKILL.md`
  - `.agent/skills/<skill-name>/SKILL.md`
  - `.agents/skills/<skill-name>/SKILL.md`
  - `.github/skills/<skill-name>/SKILL.md`
  - `skills/<skill-name>/SKILL.md`
- Read feature-specific guides: `<skill-name>/references/filtering.md`, `sorting.md`, `validation.md`, `styling.md`, etc.

### Step 4: Read the Syncfusion themes guide to install the overall Syncfusion WinForms theme package:
1. **Read:** Syncfusion themes documentation using discovery paths:
   - `.codestudio/skills/syncfusion-winforms-ui-builder/references/syncfusion-themes.md`
   - `.agent/skills/syncfusion-winforms-ui-builder/references/syncfusion-themes.md`
   - `.agents/skills/syncfusion-winforms-ui-builder/references/syncfusion-themes.md`
   - `.github/skills/syncfusion-winforms-ui-builder/references/syncfusion-themes.md`
   - `skills/syncfusion-winforms-ui-builder/references/syncfusion-themes.md`
   - Supported themes: Office2019Colorful, Office2019Black, HighContrastBlack, Office2013, etc.
   - Load theme via SkinManager in Program.cs
   - Verify NuGet package versions are compatible with Syncfusion.Licensing

### Step 5: NOW Generate Code Using Extracted Information

Only after completing Steps 1-4, generate the `.cs` files using the exact namespaces and assemblies extracted from control skills.

**Common Mistake to Avoid:**
❌ Generate code, then try to add using statements later → Results in missing namespace, broken compilation
✅ Read getting-started FIRST, extract namespaces, THEN generate code with all using statements included

**Why This Order Is Critical:**
- Control skills contain the authoritative setup syntax
- Using statements are essential and cause compilation errors if missing
- Reading first ensures: correct namespaces, correct theme resources, no missing dependencies, compilation-free code
- Compatibility with Syncfusion WinForms control versions

---

## Code Generation Process

**After reading all control skills, AI Should:**

1. **Generate .cs (C#) class file**:
   - Derived from `Form`, `UserControl`, or control container
   - Proper Syncfusion namespaces and using statements
   - Control declarations with proper initialization
   - Event handlers following .NET naming conventions (OnClick, OnValueChanged, etc.)
   - Error handling and validation with try-catch blocks
   - Accessibility attributes (tab order, labels, mnemonics)
   - XML documentation comments explaining usage

2. **Generate Designer file** (.Designer.cs):
   - Auto-generated by Visual Studio designer (InitializeComponent method)
   - Control layout and positioning
   - Property assignments for size, location, font, colors
   - Event wiring

3. **Generate resource files** (.resx):
   - Localization strings if applicable
   - Image and icon resources
   - Font and styling resources

4. **Reference code standards** from:
   - winforms-dotnet-standards.md (accessibility + security + .NET best practices)
   - Control skill SKILL.md and feature-specific guides (if available): validation.md, styling.md, data-binding.md, filtering.md, etc.
   - Official Syncfusion documentation for controls without skill files

**Code Generation Standards:**

- **Control Namespaces:** Use exact using statements from the control skill SKILL.md or official Syncfusion documentation
- **Assembly References:** Include ONLY officially documented Syncfusion assemblies explicitly mentioned in skill files or official Syncfusion resources
- **Naming Conventions:** Follow C# PascalCase for classes, camelCase for variables, UPPERCASE_SNAKE for constants
- **Accessibility:** Labels on forms, tab order (TabIndex), keyboard shortcuts (mnemonics), accessible color contrast
- **Type Safety:** Strong typing, no use of `object` without casting, proper nullable handling
- **Error Handling:** Try-catch blocks, user-friendly error messages, logging where appropriate
- **Layout:** Use TableLayoutPanel or FlowLayoutPanel for responsive design, proper DPI awareness
- **Performance:** Dispose of resources properly, use DoubleBuffered for flicker-free rendering, lazy load where needed
- **Security:** Input validation, parameterized queries for databases, no embedded passwords or keys
- **Comments:** XML documentation on public members, explain complex logic with // comments

### Media (MANDATORY)

- **Placeholder Images:** Embed images as resources in the project or reference from local file paths
  - Recommended: Add images to a `Resources` folder in the project
  - Use `Image.FromFile()` or load from assembly resources
  - Always specify dimensions (width x height) when creating PictureBox controls
  - Use relevant images for context-appropriate UI

### Icon Handling (MANDATORY)

**Step 1: Attempt to find icon from Control Mapping**
- **Always run ControlMapper script first** to get semantic icon mappings (BM25 search against Syncfusion WinForms icons)
- **Use Syncfusion built-in icons if available:** Syncfusion provides icon font or ImageList integration
- **Never leave empty space** - either icon or emoji, not blank

**Step 2: If Icon Not Found or Score Too Low**
- ❌ DO NOT skip or use empty space
- ✅ Update the element's `type_hint` in `control-mapping.json`
- ✅ Run the ControlMapper script again: `node controls_search.cjs ../control-mapping.json`
- ✅ Re-check the script output for improved control match

**Step 3: If Still Not Found**
- ✅ Use Unicode character or emoji: Set button text to `"📧"` or relevant Unicode symbol
- Add a text label alongside for clarity if needed
- Document why icon wasn't found in code comments
- Maintain visual consistency with other components

**Example**:
```json
// Before: type_hint was too vague
"type_hint": "email"

// After: updated with more keywords
"type_hint": "email envelope mail message send notification"
```

### Button & Icon Styling with Syncfusion (MANDATORY)

**Principle:** Let Syncfusion own button dimensions and styling. Use container panels only for layout around buttons.

❌ **INCORRECT** - Overriding Syncfusion SfButton properties randomly:
```csharp
sfButton1.Width = 150;
sfButton1.Height = 40;
sfButton1.ForeColor = Color.Blue;
sfButton1.Font = new Font("Arial", 14, FontStyle.Bold);
```

✅ **CORRECT** - Set Syncfusion SfButton properties as designed, use containers for layout:
```csharp
using Syncfusion.WinForms.Buttons;

// Let Syncfusion SfButton define its own appearance
SfButton sfButton1 = new SfButton();
sfButton1.Text = "▶ Play";
sfButton1.Size = new Size(96, 28);
sfButton1.Font = new Font("Segoe UI Semibold", 9F);
sfButton1.UseVisualStyleBackColor = true;

// Use FlowLayoutPanel for button positioning
FlowLayoutPanel buttonPanel = new FlowLayoutPanel();
buttonPanel.Controls.Add(sfButton1);
buttonPanel.Controls.Add(infoButton);
buttonPanel.Spacing = 12;
```

**Why This Works:**
- Syncfusion defines sizing + alignment internally
- Control styling respects the design system
- No property conflict or override issues
- Consistent appearance across all Syncfusion components
- DPI scaling handled automatically by WinForms

---

### Control Reuse Across UI (Same Control, Multiple Places)

**Principle:** One Syncfusion control type can be reused throughout your UI with customizations. For example, a `SfButton` can serve as the Login button, Forgot Password link, and Sign Up button—each customized via properties.

**Example - SfButton Used in Multiple Places:**
```csharp
// LoginForm.cs
using Syncfusion.WinForms.Buttons;
using System.Drawing;
using System.Windows.Forms;

public partial class LoginForm : Form
{
    private SfButton loginButton;
    private LinkLabel forgotPasswordButton;
    private SfButton signUpButton;

    private void InitializeComponent()
    {
        // Primary button - main CTA
        loginButton = new SfButton();
        loginButton.Text = "Login";
        loginButton.BackColor = Color.FromArgb(13, 110, 253);
        loginButton.ForeColor = Color.White;
        loginButton.Size = new Size(100, 40);
        loginButton.Margin = new Padding(0, 0, 12, 0);
        
        // Link label - secondary action
        forgotPasswordButton = new LinkLabel();
        forgotPasswordButton.Text = "Forgot Password?";
        forgotPasswordButton.LinkColor = Color.Gray;
        forgotPasswordButton.AutoSize = true;
        
        // Secondary button - tertiary action
        signUpButton = new SfButton();
        signUpButton.Text = "Sign Up Here";
        signUpButton.BackColor = Color.White;
        signUpButton.ForeColor = Color.Gray;
        signUpButton.Size = new Size(100, 40);
        signUpButton.FlatStyle = FlatStyle.Flat;
        signUpButton.FlatAppearance.BorderColor = Color.LightGray;
        signUpButton.FlatAppearance.BorderSize = 1;
    }
}
```

**Why This Pattern Works:**
- Reuse single control type with different property configurations
- Maintain consistency across the application
- Easy to update styling for all instances of a type
- Each instance maintains its own state and event handlers

---

### Reading Control Skills BEFORE Using Components (MANDATORY)

**CRITICAL:** Do NOT assume control properties or APIs.

**Required Process:**
1. **Identify all mapped components** from Stage 3 output using controls.csv
   - E.g., GridControl, ChartComponent, TabComponent, TextBoxExt, SfButton, etc.

2. **For EACH control**, read the control skill:
   - Location: `.codestudio/skills/<control-skill>/references/getting-started.md`
   - Extract: namespaces (e.g., `Syncfusion.Windows.Forms.Grid`, `Syncfusion.WinForms.Buttons`), assembly references, required properties, setup code
   - Read: feature-specific guides (data-binding.md, validation.md, styling.md, etc.)

3. **DO NOT generate code without reading** control skill documentation
   - Don't assume property names or API structure
   - Don't guess at event handler names
   - Don't skip required setup or initialization

**Example - Reading GridControl Control Skill:**
```
Before generating code:
1. Read: .codestudio/skills/syncfusion-winforms-grid-control/references/getting-started.md
   → Extract: using Syncfusion.Windows.Forms.Grid;
   → Extract: Required assemblies: Syncfusion.Grid.Windows.dll, Syncfusion.Grid.Base.dll, Syncfusion.Shared.Base.dll
   → Read: required properties, RowCount, ColCount, cell access patterns

2. Read: .codestudio/skills/syncfusion-winforms-grid-control/references/data-population.md
   → Understand: Populating cells, grid data structures, binding patterns

3. Read: .codestudio/skills/syncfusion-winforms-grid-control/references/cell-style-architecture.md
   → Understand: GridStyleInfo, cell formatting, colors, fonts

4. NOW generate code with correct namespaces, properties, and methods
```

**What Control Skills Contain:**
- ✅ Authoritative using statements and namespaces (e.g., `using Syncfusion.Windows.Forms.Grid;`)
- ✅ Required assembly references (CRITICAL - don't skip, e.g., `Syncfusion.Grid.Windows.dll`)
- ✅ Complete API documentation with correct class names (GridControl, SfButton, TextBoxExt, etc.)
- ✅ Feature-specific patterns (data binding, filtering, validation)
- ✅ Best practices and performance considerations
- ✅ Accessibility requirements (tab order, mnemonics, keyboard navigation)
- ✅ Theme and styling customization options via properties

**Common Mistakes to Avoid:**
- ❌ Guessing property names → Read skill documentation
- ❌ Missing or incorrect assembly references → Extract from getting-started.md
- ❌ Wrong class names or control types → Use exact names from controls.csv
- ❌ Incomplete initialization → Follow skill's recommended setup code

---

**Example Output Files:**

```
Forms/
  ├── LoginForm.cs              (Main form code-behind)
  ├── LoginForm.Designer.cs     (Auto-generated designer file)
  └── LoginForm.resx            (Resources)
```

---

## Output File Structure

### Simple UI:
```
Forms/LoginForm/
  ├── LoginForm.cs              (Main form code-behind)
  ├── LoginForm.Designer.cs     (Auto-generated designer file)
  ├── LoginForm.resx            (Resources and strings)
  └── Resources/
      └── Images/               (Image files)
```

### Complex UI (Multiple Sections - using UserControls):
```
Forms/Dashboard/
├── DashboardForm.cs            (Main form coordinator)
├── DashboardForm.Designer.cs   (Auto-generated)
├── DashboardForm.resx          (Resources)
├── Controls/
│   ├── HeaderControl/
│   │   ├── HeaderControl.cs
│   │   ├── HeaderControl.Designer.cs
│   │   └── HeaderControl.resx
│   ├── SidebarControl/
│   │   ├── SidebarControl.cs
│   │   ├── SidebarControl.Designer.cs
│   │   └── SidebarControl.resx
│   ├── MainContentControl/
│   │   ├── MainContentControl.cs
│   │   ├── MainContentControl.Designer.cs
│   │   └── MainContentControl.resx
│   └── FooterControl/
│       ├── FooterControl.cs
│       ├── FooterControl.Designer.cs
│       └── FooterControl.resx
└── Resources/
    └── Images/
```

**WinForms Structure Rules:**
- Each section gets its own folder with `.cs`, `.Designer.cs`, and `.resx` files
- For complex UIs, use `UserControl` instead of `Form` for sections
- Parent Form (`DashboardForm`) hosts UserControls via `Controls.Add()`
- Syncfusion theme loaded in `Program.cs` via `SkinManager` or `ThemeProvider`
- Do NOT collapse multiple distinct sections into single file
- Each UserControl/Form is independent and can be tested separately

---

## Control Integration & File Mapping

**Generated files MUST be wired to display in the app:**

1. **Main Form Class** (`Program.cs`):
   ```csharp
   using Syncfusion.Licensing;
   using System.Windows.Forms;
   
   static class Program
   {
       [STAThread]
       static void Main()
       {
           // Register Syncfusion license (if required)
           // SyncfusionLicenseProvider.RegisterLicense("YOUR_LICENSE_KEY");
           
           Application.EnableVisualStyles();
           Application.SetCompatibleTextRenderingDefault(false);
           Application.Run(new LoginForm());
       }
   }
   ```

2. **Form Registration** (if using MDI or form manager):
   ```csharp
   public partial class MainForm : Form
   {
       private void ShowLoginForm()
       {
           LoginForm loginForm = new LoginForm();
           loginForm.MdiParent = this;
           loginForm.Show();
       }
   }
   ```

3. **Ensure Resources are loaded**:
   - If using Designer: Resources automatically generated in .Designer.cs
   - If embedding images: Add to project Resources folder
   - If using Syncfusion theme: Reference in app configuration or load in Program.cs before Application.Run()

4. **Theme Application in Program.cs** (if using Syncfusion theme):
   ```csharp
   using Syncfusion.WinForms.Themes;
   using Syncfusion.Licensing;
   
   static class Program
   {
       [STAThread]
       static void Main()
       {
           // Register license first
           SyncfusionLicenseProvider.RegisterLicense("YOUR_LICENSE_KEY");
           
           // Load Syncfusion theme
           ThemeProvider.SyncTheme = "Office2019Colorful";
           
           Application.EnableVisualStyles();
           Application.SetCompatibleTextRenderingDefault(false);
           Application.Run(new MainForm());
       }
   }
   ```

**Without this mapping, form won't display in the application.**

**User Interaction:** 
Optional review of generated code. No blocking confirmation.

**Status:** AI generates without user decision. User can review/adjust if needed.
