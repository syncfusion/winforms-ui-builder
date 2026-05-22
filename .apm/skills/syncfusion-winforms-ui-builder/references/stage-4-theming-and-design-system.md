# Stage 4: Theming & Design System Selection for Windows Forms

**Purpose:** Lock theming and design system decisions before code generation in Stage 5.

---

## ⚠️ CRITICAL: Syncfusion-First Approach

**Before selecting themes, verify that Syncfusion WinForms controls support your required functionality:**

1. **Check Syncfusion WinForms documentation** for the specific control you need
   - [Syncfusion WinForms Controls](https://help.syncfusion.com/windowsforms/overview)
   - [Control Gallery](https://www.syncfusion.com/winforms-ui-controls)

2. **If Syncfusion provides the control** → Use Syncfusion control with appropriate theme
3. **If Syncfusion does NOT support the requirement** → Use standard Microsoft WinForms controls as fallback
4. **NEVER assume Syncfusion provides a control** by inferring from control names or patterns

**Use ONLY officially documented Syncfusion NuGet packages:**
- Verify all package names are EXPLICITLY documented in:
  - The corresponding control skill file (`.codestudio/skills/syncfusion-winforms-<name>/SKILL.md`), OR
  - Official Syncfusion documentation at [Syncfusion WinForms Controls](https://help.syncfusion.com/windowsforms/overview)
- Do NOT append `.WinForms` or other patterns to control names to create package names (this approach is incorrect)
- Example: For DataGrid, use `Syncfusion.SfDataGrid.WinForms` (verified from official docs), NOT `Syncfusion.DataGrid.WinForms` (assumed pattern)
- **REJECT any package name that is not explicitly documented** in control skill files or official Syncfusion resources

---

## Overview

This stage is about **decision-making clarity**, not code generation. You'll:

- **Select a Syncfusion theme** appropriate for your WinForms application
- **Understand design philosophy** differences between Office themes and accessibility profiles
- **Define color architecture** and branding strategy for Windows Forms controls
- **Establish typography and spacing** standards for desktop UI
- **Plan accessibility** and dark mode support (if applicable)
- **Document design decisions** so Stage 5 can generate consistent code

**Key Insight:** Windows Forms theming is fundamentally different from web theming. Instead of CSS frameworks (Tailwind, Bootstrap, Material), you use **SkinManager** to apply predefined Syncfusion themes or custom Theme Studio-based themes. Your theme choice directly impacts all Syncfusion controls in your application and determines your assembly dependencies.

This file provides theming guidance for Syncfusion Windows Forms controls. For implementation details, refer to:
- **[syncfusion-themes.md](syncfusion-themes.md)** — Detailed theme reference, assembly requirements, and theme selection
- **[winforms-dotnet-standards.md](winforms-dotnet-standards.md)** — WCAG 2.2 AA accessibility, .NET security, and code quality standards

**Output:** Theme selection and design system decisions documented and ready for Stage 5 implementation.

---

## Table of Contents

1. [Syncfusion Theme Selection & Available Themes](#1-syncfusion-theme-selection--available-themes)
2. [Built-In Themes (No Extra Assemblies)](#2-built-in-themes-no-extra-assemblies)
3. [Premium Themes (Separate Assemblies)](#3-premium-themes-separate-assemblies)
4. [Color System Architecture for Windows Forms](#4-color-system-architecture-for-windows-forms)
5. [Typography & Spacing Standards](#5-typography--spacing-standards)
6. [Accessibility & WCAG 2.2 Compliance](#6-accessibility--wcag-22-compliance)
7. [Dark Mode Strategy](#7-dark-mode-strategy)
8. [Assembly Loading & Theme Application](#8-assembly-loading--theme-application)
9. [Custom Color Customization Strategy](#9-custom-color-customization-strategy)
10. [Stage 4 Decision Checklist](#10-stage-4-decision-checklist)
11. [What Stage 5 Does With These Decisions](#11-what-stage-5-does-with-these-decisions)

---

## 1. Syncfusion Theme Selection & Available Themes

**Input:** Application branding requirements, accessibility needs, deployment target

**Decision Point:** Your theme choice defines the visual identity and experience of your application. Understand what's available and what aligns with your needs.

### Theme Selection Strategy

Syncfusion provides **legacy built-in themes** (Office2007/2010/2013) and **modern premium themes** (Office2019, HighContrastBlack) that require explicit assembly loading.

**Key Principle:** Use **Office2019Colorful** as your default for new projects (latest technology, modern design). Use **HighContrastBlack** for accessibility-critical applications (WCAG AAA). Legacy built-in themes are supported for backward compatibility only.

### Why Theme Selection Matters

Your Syncfusion theme directly impacts:
- **Visual consistency** across all Syncfusion controls (DataGrid, Button, TextBox, ComboBox, etc.)
- **Accessibility compliance** (built-in themes meet WCAG AA; HighContrastBlack exceeds WCAG AAA)
- **Assembly dependencies** (built-in themes = no extra DLLs; premium themes = explicit assembly loading)
- **Customization complexity** (built-in themes are production-ready; custom themes require testing)
- **User experience** (theme choice affects professional appearance and usability)

**Non-Obvious Pattern:** Don't create custom themes unless absolutely necessary. Built-in themes are designed by Microsoft Office designers and are proven in enterprise applications. Custom themes require extensive testing and maintenance.

**Output:** Theme selection strategy understood

---

## 2. Built-In Themes (No Extra Assemblies) - Legacy/Backward Compatibility

**These themes are immediately available with no separate assembly loading required.**  
**⚠️ Note: Built-in themes are legacy. Use Premium Themes (Office2019, HighContrastBlack) for new projects.**

| Theme | Visual Style | Best For | Accessibility | Status |
|-------|-------------|----------|----------------|--------|
| **Office2007** | Classic Office 2007 styling | Legacy applications | Good contrast | ⚠️ Legacy |
| **Office2007Blue** | Blue accent variant | Backward compatibility | ✅ WCAG AA | ⚠️ Legacy |
| **Office2007Black** | Dark variant | Legacy dark themes | ✅ WCAG AA | ⚠️ Legacy |
| **Office2007Silver** | Neutral silver | Minimal aesthetic | ✅ WCAG AA | ⚠️ Legacy |
| **Office2010** | Office 2010 appearance | Traditional corporate | ✅ WCAG AA | ⚠️ Legacy |
| **Office2010Blue/Black/Silver** | Color variants | Flexible legacy styling | ✅ WCAG AA | ⚠️ Legacy |
| **Office2013** | Modern flat design | Contemporary apps | Excellent | ⚠️ Legacy |
| **Office2013White** | Light professional | Bright UI | ✅ WCAG AA | ⚠️ Legacy |
| **Office2013DarkGray** | Dark theme | Eye-friendly | ✅ WCAG AA | ⚠️ Legacy |
| **Office2013Black** | High contrast dark | Maximum accessibility | ✅ WCAG AAA | ⚠️ Legacy (Use HighContrastBlack instead) |
| **Metro** | Flat, minimal design | Clean aesthetics | Good contrast | ⚠️ Legacy |

### Premium Themes: The New Default Standard

**Premium Themes (Recommended for most projects):**
```csharp
// Program.cs - Latest Office2019 theme
SkinManager.LoadAssembly(typeof(Office2019Theme).Assembly);
SkinManager.ApplicationVisualTheme = "Office2019Colorful";
Application.Run(new MainForm());
```

- ✅ **Latest Technology:** Office2019 is the current standard
- ✅ **Modern Appearance:** Contemporary design aligned with current Microsoft standards
- ✅ **WCAG AA Compliance:** Full accessibility (Office2019)
- ✅ **WCAG AAA Compliance:** Exceptional accessibility (HighContrastBlack)
- ✅ **Customizable:** ThemeStyle support for brand-specific overrides
- ✅ **Enterprise Ready:** Proven in modern enterprise applications
- ✅ **Future-Proof:** Latest supported Syncfusion themes

**Built-In Themes (Legacy - Use Only When):**
- Maintaining backward compatibility with Office2007/2010/2013 applications
- Project explicitly prohibits additional assembly dependencies
- Special legacy system requirement

**Decision Point:** Are you using latest Office2019/HighContrastBlack, or do you have legacy constraints?

### Office2019 & HighContrastBlack: Recommended Premium Themes

**Recommendation: Use Office2019Colorful or HighContrastBlack as your primary choice**

**Why Office2019 is Preferred:**
- ✅ **Latest Design:** Most current Office 2019 aesthetic with vibrant branding
- ✅ **Modern Appearance:** Contemporary UI aligned with current Microsoft standards
- ✅ **Advanced Customization:** Supports ThemeStyle property for brand-specific overrides
- ✅ **WCAG AA Compliance:** Full accessibility built-in
- ✅ **Enhanced Visual Appeal:** Refined appearance compared to Office2013
- ✅ **Future-Proof:** Latest supported theme (Office2019Colorful)
- ✅ **Professional Look:** Enterprise-grade styling

**Why HighContrastBlack is Essential:**
- ✅ **Accessibility Excellence:** WCAG AAA compliance (7:1+ contrast)
- ✅ **Vision Impairment Support:** Maximum readability for users with visual disabilities
- ✅ **Compliance Ready:** Meets government/enterprise accessibility mandates
- ✅ **Color-Blind Friendly:** Designed for maximum distinction
- ✅ **Professional Appearance:** High-contrast without losing professional quality

**Assembly Requirements:**
- **Office2019:** `Syncfusion.Office2019Theme.WinForms` (latest theme)
- **HighContrastBlack:** `Syncfusion.HighContrastTheme.WinForms` (accessibility leader)

**Best Practice Matrix:**
| Project Type | Recommended Theme | Reason |
|--------------|-------------------|--------|
| **New Projects** | Office2019Colorful | Latest technology, best visuals, forward-compatible |
| **Corporate Apps** | Office2019Colorful | Modern appearance, customizable for branding |
| **Accessibility-First** | HighContrastBlack | WCAG AAA compliance, vision impairment support |
| **Government/Regulated** | HighContrastBlack | Exceeds accessibility requirements |
| **Mixed Accessibility Needs** | Both (Office2019 + HighContrastBlack toggle) | Support users with and without visual needs |

**Legacy Themes (Built-In) - Use Only If:**
- Maintaining backward compatibility with existing Office2007/2010/2013 applications
- Project explicitly requires no additional assembly dependencies
- Otherwise, Office2019 is the default preference

**Output:** Premium theme selected (Office2019 or HighContrastBlack) with documented rationale

---

## 3. Premium Themes (Separate Assemblies)

**These themes require explicit assembly loading but offer enhanced styling aligned with latest Office designs.**

### Office2016 Theme
- **Assembly:** `Syncfusion.Office2016Theme.WinForms.dll`
- **Variants:** White, DarkGray, Black, Colorful
- **Best For:** Modern contemporary Office appearance
- **Compatibility:** Only for Sf-prefixed controls (SfDataGrid, SfButton, SfDateTimeEdit, SfNumericTextBox, SfSmithChart)
- **Why:** More refined than Office2013; aligns with Microsoft Office 2016 UI
- **Accessibility:** ✅ WCAG AA compliant

### Office2019 Theme
- **Assembly:** `Syncfusion.Office2019Theme.WinForms.dll`
- **Variants:** Office2019Colorful (primary variant)
- **Best For:** Latest Microsoft Office aesthetic with vibrant branding
- **Why:** Most current Office styling; supports advanced ThemeStyle customization
- **Customization:** Advanced ThemeStyle property for appearance overrides
- **Accessibility:** ✅ WCAG AA compliant

### HighContrast Theme
- **Assembly:** `Syncfusion.HighContrastTheme.WinForms.dll`
- **Variants:** HighContrastBlack
- **Best For:** Accessibility-first applications; compliance requirements
- **Why:** Maximum contrast ratios; WCAG AAA compliance; vision impairment support
- **Customization:** Supports advanced customization for brand-aware high contrast styling
- **Accessibility:** ✅ WCAG AAA (7:1+ contrast)

### Premium Theme Selection Decision Tree

**For New Projects (Default Recommendation):**
→ Use **Office2019Colorful** (separate assembly, latest standard)
- Why: Latest Office design language, modern appearance, supports advanced ThemeStyle customization
- Assembly: Add `Syncfusion.Office2019Theme.WinForms` via NuGet
- Best for: All new WinForms applications unless specific constraints apply

**For Modern Professional Applications:**
→ Use **Office2019Colorful** (separate assembly, contemporary, highly customizable)
- Why: Current Microsoft Office aesthetic with vibrant branding and professional refinement
- Assembly: Add `Syncfusion.Office2019Theme.WinForms` via NuGet
- Supports: Advanced ThemeStyle customization for enterprise branding

**For Dark-Themed Applications:**
→ Use **Office2019Colorful** with dark mode support OR **HighContrastBlack**
- Office2019 + Dark Toggle: Modern appearance with user preference support
- HighContrastBlack: Maximum contrast for dark environments, WCAG AAA

**For Accessibility-First Applications (WCAG AAA):**
→ Use **HighContrastBlack** (separate assembly, WCAG AAA)
- Why: Exceeds accessibility requirements; 7:1+ contrast; vision impairment support
- Assembly: Add `Syncfusion.HighContrastTheme.WinForms` via NuGet
- Best for: Government/enterprise compliance, accessibility mandates
- Recommended with: Office2019 toggle for users without accessibility needs

**For Maximum Brand Customization:**
→ Use **Office2019Colorful** with custom ThemeStyle (separate assembly)
- Why: Supports per-control appearance overrides and brand colors; latest technology
- Best for: Large enterprise applications with strict branding guidelines
- Benefit: Future-proof with latest Office design standards

### Assembly Loading Requirements

**Built-In Themes:** No assembly loading needed
```csharp
SkinManager.ApplicationVisualTheme = "Office2013White";  // Immediate
```

**Premium Themes:** Require explicit NuGet package + assembly loading
```csharp
// Program.cs - Premium theme must be loaded before UI initializes
SkinManager.LoadAssembly(typeof(Office2019Theme).Assembly);
SkinManager.ApplicationVisualTheme = "Office2019Colorful";
Application.Run(new MainForm());
```

**Output:** Theme category selected (built-in vs. premium) with documented rationale

---

## 4. Color System Architecture for Windows Forms

### 4.1 Syncfusion Theme Colors vs Custom Overrides

**Principle:** Use Syncfusion theme as your base; customize only when brand requirements mandate it.

| Approach | When To Use | Pros | Cons |
|----------|-----------|------|------|
| **Theme Only** | Small to medium projects | Simple, no overhead, proven | Limited brand expression |
| **Theme + Overrides** | Medium projects with branding | Balance of consistency and customization | Requires testing across controls |
| **Custom Theme Studio** | Large projects, strict brand | Full control, dedicated styling | Higher setup complexity |

### 4.2 Brand Color Integration Strategy

**If using built-in theme only (Recommended):**
- Syncfusion theme handles all color defaults
- Your brand appears through logo, content, and application layout
- Minimal styling decisions needed
- ✅ Fastest path, lowest risk

**If customizing colors:**
1. Start with Syncfusion theme (e.g., Office2019Colorful)
2. Define your brand colors in code or configuration
3. Use `ThemeStyle` property to override specific control appearances
4. Document all overrides for consistency across forms

**Example Color Customization:**
```csharp
// Define brand colors
Color brandPrimary = System.Drawing.Color.FromArgb(0, 102, 204);
Color brandSuccess = System.Drawing.Color.FromArgb(52, 168, 83);
Color brandWarning = System.Drawing.Color.FromArgb(255, 180, 0);
Color brandError = System.Drawing.Color.FromArgb(216, 48, 32);

// Apply to specific controls
this.treeViewAdv1.ThemeStyle.BorderColor = brandPrimary;
this.sfDataGrid1.Style.HeaderStyle.BackColor = brandPrimary;
this.sfDataGrid1.Style.RecordStyle.BackColor = Color.White;
this.sfDataGrid1.Style.AlternatingRecordStyle.BackColor = Color.FromArgb(240, 248, 255);
```

### 4.3 Semantic Color Roles

Define these color roles for consistency:

| Role | Purpose | Example | Usage |
|------|---------|---------|-------|
| **Primary** | Brand color, main actions | #0066CC | Buttons, highlights, branding |
| **Success** | Positive outcomes | #34A853 | Checkmarks, success messages |
| **Warning** | Cautions, alerts | #FFB400 | Warning icons, alert states |
| **Error** | Errors, destructive actions | #D83020 | Error text, delete buttons |
| **Info** | Informational content | #1F73E7 | Info icons, notifications |
| **Neutral Dark** | Text, dark surfaces | #1F2937 | Body text, backgrounds |
| **Neutral Light** | Light surfaces, borders | #F3F4F6 | Card backgrounds, borders |

### 4.4 Accessibility: Color Contrast Verification

**WCAG 2.2 AA Minimum:** 4.5:1 contrast for normal text

**Your Responsibility:** If you customize colors, verify contrast ratios:
- Text on background: 4.5:1 minimum (AA) or 7:1 (AAA)
- UI controls (borders, backgrounds): 3:1 minimum
- Use WCAG contrast checker tools to verify

**Validation Rule:**
```csharp
// Example: Verify contrast (use online contrast checker tools)
// Text: #1F2937 (dark gray) on background #F3F4F6 (light gray)
// Contrast ratio: 12.6:1 ✓ WCAG AAA
// 
// Text: #757575 (medium gray) on background #FFFFFF (white)
// Contrast ratio: 4.5:1 ✓ WCAG AA (minimum)
```

**Output:** Color system architecture defined with brand colors and semantic roles locked

---

## 5. Typography & Spacing Standards

### 5.1 Typography Hierarchy for Windows Forms

Windows Forms doesn't use CSS typography scales. Use consistent font sizing for readability:

| Element Type | Font | Size | Weight | Usage |
|-------------|------|------|--------|-------|
| **Form Title** | Segoe UI | 14-16px | Bold | Main window title bar, section headers |
| **Section Headers** | Segoe UI | 12-13px | Bold | Group titles, major sections |
| **Body Text** | Segoe UI | 11px | Regular | Labels, content, standard text |
| **Small Text** | Segoe UI | 10px | Regular | Help text, tooltips, status bars |
| **Monospace** | Consolas | 10px | Regular | Code display, technical data |

**Standard:** Use **Segoe UI** as default (Windows native, excellent readability)

**Rule:** Minimum body text is **11px**. Smaller than this strains eyes on desktop.

### 5.2 Spacing Standards

Windows Forms uses logical spacing measured in pixels:

| Spacing Role | Size | Usage | Example |
|-------------|------|-------|---------|
| **Extra Small (XS)** | 4px | Minimal gaps, icon spacing | Icon-to-text distance |
| **Small (S)** | 8px | Control padding, tight spacing | Form control margins |
| **Medium (M)** | 12px | Standard gaps, form field padding | Field padding inside containers |
| **Large (L)** | 16px | Generous spacing, section separation | Section-to-section spacing |
| **Extra Large (XL)** | 24px | Major layout breaks, top-level separation | Form area separation |

**Windows Forms Convention:** Use **4px multiples** (4, 8, 12, 16, 20, 24, 32px) for consistency.

**Margin vs Padding:**
- **Padding:** Internal spacing within controls (button text to edge = 8-12px)
- **Margin:** External spacing between controls (vertical gap = 12-16px)

### 5.3 Control Sizing Standards

**Minimum Target Size (WCAG 2.2 AA):** 24×24 screen pixels
**Comfortable Target Size (Recommended):** 44×44 pixels for touch accessibility

| Control Type | Min Height | Recommended | Padding |
|-------------|-----------|-------------|---------|
| **Button** | 24px | 32-44px | 8-12px horizontal, 4-8px vertical |
| **TextBox** | 24px | 28px | 4-8px padding |
| **CheckBox** | 24px | 28px | Touch target 44×44 |
| **ComboBox** | 24px | 28px | 4-8px padding |
| **DataGrid Header** | 20px | 24px | 4-8px padding |
| **DataGrid Row** | 20px | 24px | 4-8px padding |
| **Icon Button** | 24px | 44px | Larger for touch |

**Example Control Layout:**
```csharp
// Good spacing and sizing
Button okButton = new Button
{
    Text = "OK",
    Width = 80,
    Height = 32,
    Margin = new Padding(8),       // Margin from other controls
    Padding = new Padding(8, 4)    // Internal padding
};

Label label = new Label
{
    Text = "Name:",
    AutoSize = true,
    Margin = new Padding(0, 12, 0, 4)  // 12px above, 4px below
};

TextBox textBox = new TextBox
{
    Width = 200,
    Height = 24,
    Margin = new Padding(0, 0, 0, 12)
};
```

**Output:** Typography and spacing standards locked

---

## 6. Accessibility & WCAG 2.2 Compliance

Windows Forms accessibility is built into Syncfusion controls, but you must ensure proper integration.

### 6.1 Keyboard Navigation

**Requirement:** All functionality must be accessible via keyboard

**What to implement:**
- ✅ Tab order is logical (left-to-right, top-to-bottom)
- ✅ All interactive elements are keyboard-accessible
- ✅ Tab stops set correctly (TabStop = true for interactive controls)
- ✅ Escape key closes dialogs and cancels operations
- ✅ Enter submits forms and activates buttons
- ✅ Arrow keys work for lists, dropdowns, trees

**Example:**
```csharp
// Good tab order
Label nameLabel = new Label { Text = "Name:", TabIndex = -1 };
TextBox nameInput = new TextBox { TabIndex = 0 };  // First tab stop
TextBox emailInput = new TextBox { TabIndex = 1 }; // Second tab stop
Button submitButton = new Button { Text = "Submit", TabIndex = 2 };

// Escape closes dialog
public partial class MyDialog : Form
{
    protected override void OnKeyDown(KeyEventArgs e)
    {
        if (e.KeyCode == Keys.Escape)
        {
            this.Close();
            e.Handled = true;
        }
        base.OnKeyDown(e);
    }
}
```

### 6.2 Focus Management & Visibility

**Requirement:** Users must always see where keyboard focus is

**Never do this:**
```csharp
textBox.HideSelection = true;  // ❌ VIOLATES WCAG: hides focus when unfocused
```

**Instead:**
```csharp
textBox.HideSelection = false;  // ✅ Always show focus indicator
```

### 6.3 Color Contrast

**WCAG 2.2 AA Minimum:** 4.5:1 contrast for normal text

**All Syncfusion themes meet WCAG AA (4.5:1):**
- ✅ Office2013White, Office2013Black: WCAG AA
- ✅ Office2013DarkGray: WCAG AA
- ✅ HighContrastBlack: WCAG AAA (7:1+)

**If you customize colors, verify:**
- Text on background: 4.5:1 minimum (AA) or 7:1 (AAA)
- UI controls (borders, backgrounds): 3:1 minimum
- Use online WCAG contrast checkers to verify

### 6.4 Accessible Names & Descriptions

**For all interactive elements:**

```csharp
// ❌ Missing accessible name
Button button = new Button { Image = icon };

// ✅ Proper accessibility
Button button = new Button
{
    Image = icon,
    AccessibleName = "Menu",
    AccessibleDescription = "Open application menu",
    AccessibleRole = AccessibleRole.PushButton
};

// ✅ Labels for inputs
Label emailLabel = new Label { Text = "Email address:" };
TextBox emailInput = new TextBox
{
    AccessibleName = "Email address",
    AccessibleDescription = "Enter your email for password reset"
};

// ✅ Error messages linked to fields
Label errorLabel = new Label { Text = "Invalid email format" };
emailInput.AccessibleDescription = "Invalid email format. Please enter a valid email address.";
```

### 6.5 Target Size (WCAG 2.2 NEW)

**Minimum:** 24×24 screen pixels  
**Recommended:** 44×44 pixels for touch-friendly interfaces

All Syncfusion built-in controls meet this requirement with default sizing.

### 6.6 Validation & Error Handling

**Requirement:** Errors must be clear, accessible, and linked to fields

```csharp
void ValidateEmail(string email)
{
    if (string.IsNullOrEmpty(email))
    {
        errorMessage.Text = "Email is required";
        errorMessage.ForeColor = Color.Red;
        emailInput.BackColor = Color.FromArgb(255, 230, 230);  // Light red
        emailInput.AccessibleDescription = "Email is required";
        return false;
    }
    
    if (!email.Contains("@"))
    {
        errorMessage.Text = "Please enter a valid email address";
        errorMessage.ForeColor = Color.Red;
        emailInput.BackColor = Color.FromArgb(255, 230, 230);
        emailInput.AccessibleDescription = "Invalid email format";
        return false;
    }
    
    errorMessage.Text = "";
    emailInput.BackColor = Color.White;
    emailInput.AccessibleDescription = "";
    return true;
}
```

### 6.7 Accessibility Testing

**Recommended Tools:**
- **Narrator** (Win + Ctrl + N) — Test with built-in screen reader
- **Inspect.exe** (Windows SDK) — Examine control properties and tab order
- **NVDA** (Free) — Advanced screen reader testing
- **Contrast Analyzer** — Verify color contrast ratios

**Manual Testing Checklist:**
- [ ] Tab through entire form—order is logical
- [ ] All buttons keyboard-accessible (Enter/Space)
- [ ] Dialogs closable with Escape
- [ ] Focus indicators always visible
- [ ] Error messages clear and linked to fields
- [ ] All images/icons have accessible names
- [ ] Color contrast ≥ 4.5:1 for text
- [ ] No information conveyed by color alone

**Output:** Accessibility requirements documented and standards locked

---

## 7. Dark Mode Strategy

### 7.1 Dark Mode Decision

**Decision Point:** Will your application support dark mode?

**If NO (Light Only):**
- Use: Office2013White, Office2007Silver, Metro
- Configuration: Single theme in Program.cs
- Simplest path—no runtime switching needed

**If YES (Dark Mode Support):**
- Use: Office2013Black, Office2013DarkGray, or Office2016DarkGray
- Configuration: Either fixed dark theme OR runtime toggle
- Requires testing with dark theme

### 7.2 Dark Mode Implementation Options

**Option A: Fixed Dark Theme (Application-Wide)**
```csharp
// Program.cs - Dark theme applied at startup
static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        
        // Set dark theme globally (no runtime switching)
        SkinManager.ApplicationVisualTheme = "Office2013Black";
        
        Application.Run(new MainForm());
    }
}
```

**Option B: Runtime Theme Toggle**
```csharp
// Form with theme toggle button
public partial class MainForm : Form
{
    private bool isDarkMode = false;
    
    private void OnThemeToggle()
    {
        if (isDarkMode)
        {
            SkinManager.ApplicationVisualTheme = "Office2013White";
            isDarkMode = false;
        }
        else
        {
            SkinManager.ApplicationVisualTheme = "Office2013Black";
            isDarkMode = true;
        }
    }
    
    private void OnFormLoad(object sender, EventArgs e)
    {
        // Detect system dark mode preference (Windows 10+)
        bool systemDarkMode = IsSystemDarkModeEnabled();
        if (systemDarkMode)
        {
            SkinManager.ApplicationVisualTheme = "Office2013Black";
            isDarkMode = true;
        }
    }
    
    // Helper: Detect Windows dark mode setting
    private bool IsSystemDarkModeEnabled()
    {
        try
        {
            var key = Microsoft.Win32.Registry.CurrentUser.OpenSubKey(
                @"Software\Microsoft\Windows\CurrentVersion\Themes\Personalize");
            
            object? value = key?.GetValue("AppsUseLightTheme");
            return value is int i && i == 0;  // 0 = dark mode
        }
        catch
        {
            return false;
        }
    }
}
```

### 7.3 Dark Mode Design Considerations

**Different from Web Dark Mode:**

| Aspect | Light Mode | Dark Mode |
|--------|-----------|----------|
| **Background** | Light (white/gray) | Dark (dark gray/black) |
| **Text** | Dark text | Light text (usually lighter weight) |
| **Accents** | Vibrant colors | Slightly desaturated colors |
| **Depth** | Use shadows | Use lighter surfaces for depth |
| **Contrast** | Ensure readability | Often naturally higher contrast |

**Syncfusion Themes Already Handle:**
- ✅ Proper color adaptation (Office2013Black, DarkGray)
- ✅ Text color inversion
- ✅ Accent color adjustment
- ✅ Border and surface styling
- ✅ WCAG AA compliance maintained

**Your Responsibility:** Test with actual dark theme to ensure readability

### 7.4 Color Contrast in Dark Mode

**WCAG Requirements Still Apply:**
- Dark mode backgrounds: Light text must have 4.5:1 contrast
- Light text on dark backgrounds: naturally high contrast
- Sync theme handles this—verify with contrast checker

**Example Dark Mode Verification:**
```csharp
// Office2013Black provides:
// Background: #1F1F1F (very dark)
// Text: #FFFFFF (white)
// Contrast ratio: 21:1 ✓ WCAG AAA
```

**If Customizing Colors in Dark Mode:**
```csharp
// Dark mode custom colors
Color darkBackground = Color.FromArgb(31, 31, 31);      // #1F1F1F
Color lightText = Color.FromArgb(255, 255, 255);        // #FFFFFF
Color darkAccent = Color.FromArgb(76, 153, 228);        // Slightly desaturated blue

// Verify: #FFFFFF on #1F1F1F = 21:1 contrast ✓ WCAG AAA
```

**Output:** Dark mode strategy locked (yes/no + implementation approach if yes)

---

## 8. Assembly Loading & Theme Application

### 8.1 Assembly Loading Order (Critical for Premium Themes)

**MANDATORY SEQUENCE:**
1. Load theme assemblies (if using premium themes)
2. Set `ApplicationVisualTheme` 
3. Create and show main form

**INCORRECT Order (Don't Do This):**
```csharp
// ❌ WRONG: Theme applied after form creation
public static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.Run(new MainForm());
        
        // Too late! Theme won't apply
        SkinManager.ApplicationVisualTheme = "Office2019Colorful";
    }
}
```

**CORRECT Order (Built-In Theme):**
```csharp
// ✅ CORRECT: Built-in theme (no assembly loading needed)
public static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        
        SkinManager.ApplicationVisualTheme = "Office2013White";
        
        Application.Run(new MainForm());
    }
}
```

**CORRECT Order (Premium Theme):**
```csharp
// ✅ CORRECT: Premium theme assembly loaded before theme set
public static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        
        // Load premium theme assembly
        SkinManager.LoadAssembly(typeof(Office2019Theme).Assembly);
        
        // Then set the theme
        SkinManager.ApplicationVisualTheme = "Office2019Colorful";
        
        // Now create and show form
        Application.Run(new MainForm());
    }
}
```

### 8.2 Theme Application Scope

| Approach | When To Use | Scope | Code |
|----------|-----------|-------|------|
| **Application-Wide** | Single unified theme | All forms, all controls | `SkinManager.ApplicationVisualTheme` |
| **Form-Level** | Different themes per form | Single form and its controls | Use SkinManager in specific form |
| **Control-Level** | Mixed themes in same form | Individual control only | `control.ThemeName` property |

**Most Common: Application-Wide Theme**

All controls inherit theme globally—simplest and most consistent.

### 8.3 Required Assemblies by Theme

**Built-In Themes:** No extra assemblies required
```xml
<!-- .csproj already has these -->
<PackageReference Include="Syncfusion.Core.WinForms" Version="33.2.3" />
```

**Office2016 Theme:** Requires assembly
```xml
<PackageReference Include="Syncfusion.Office2016Theme.WinForms" Version="33.2.3" />
```

**Office2019 Theme:** Requires assembly
```xml
<PackageReference Include="Syncfusion.Office2019Theme.WinForms" Version="33.2.3" />
```

**HighContrast Theme:** Requires assembly
```xml
<PackageReference Include="Syncfusion.HighContrastTheme.WinForms" Version="33.2.3" />
```

**Output:** Assembly loading sequence documented and verified

---

## 9. Custom Color Customization Strategy

### 9.1 When to Customize vs When to Use As-Is

**Use Office2019Colorful As-Is (Recommended for 85% of projects):**
- ✅ No customization overhead
- ✅ Proven styling across all controls (latest Office design)
- ✅ Professional appearance out-of-the-box
- ✅ Faster development with modern aesthetics
- ✅ Lower maintenance burden
- ✅ Future-proof with latest technology

**Example:**
```csharp
// No customization needed—Office2019 theme is production-ready
SkinManager.LoadAssembly(typeof(Office2019Theme).Assembly);
SkinManager.ApplicationVisualTheme = "Office2019Colorful";
// That's it! All controls automatically styled with latest design.
```

**Alternative: Use HighContrastBlack As-Is (15% of projects - Accessibility-First):**
- ✅ WCAG AAA compliance built-in
- ✅ No customization needed for accessibility
- ✅ Professional high-contrast appearance
- ✅ Ideal for government/regulated industries
- ✅ Supports vision impairment requirements

**Customize Only When (Rare - 5% of large enterprises):**
- Strict brand requirements (specific corporate colors beyond theme)
- Large enterprise application with extensive branding guidelines
- Custom controls that need brand-aligned styling beyond ThemeStyle

### 9.2 Customization Approach

**If you need customization:**

1. Start with built-in theme as base
2. Define brand colors in constants
3. Apply overrides to specific controls using `ThemeStyle`
4. Document all customizations for consistency

**Example:**
```csharp
// Define brand colors as constants
public static class BrandColors
{
    public static readonly Color Primary = Color.FromArgb(0, 102, 204);      // Brand blue
    public static readonly Color Success = Color.FromArgb(52, 168, 83);      // Success green
    public static readonly Color Warning = Color.FromArgb(255, 180, 0);      // Warning orange
    public static readonly Color Error = Color.FromArgb(216, 48, 32);        // Error red
    public static readonly Color DarkText = Color.FromArgb(31, 31, 31);      // Dark text
    public static readonly Color LightBg = Color.FromArgb(243, 244, 246);    // Light background
}

// Apply customizations to controls
public partial class MainForm : Form
{
    public MainForm()
    {
        InitializeComponent();
        ApplyBrandingOverrides();
    }
    
    private void ApplyBrandingOverrides()
    {
        // DataGrid header color
        sfDataGrid1.Style.HeaderStyle.BackColor = BrandColors.Primary;
        sfDataGrid1.Style.HeaderStyle.ForeColor = Color.White;
        
        // Alternating row colors for readability
        sfDataGrid1.Style.RecordStyle.BackColor = Color.White;
        sfDataGrid1.Style.AlternatingRecordStyle.BackColor = BrandColors.LightBg;
        
        // Button appearance (if not using Syncfusion buttons)
        submitButton.BackColor = BrandColors.Primary;
        submitButton.ForeColor = Color.White;
    }
}
```

### 9.3 Testing Customizations

**Verify across all Syncfusion controls:**
- [ ] DataGrid (headers, rows, borders)
- [ ] Input controls (TextBox, ComboBox, CheckBox)
- [ ] Buttons (default, focused, pressed states)
- [ ] Containers (GroupBox, Panel backgrounds)
- [ ] Dialogs and popups
- [ ] Color contrast (WCAG AA: 4.5:1 minimum)
- [ ] Accessibility (focus indicators remain visible)

**Anti-Pattern: Incomplete Customization**

❌ Don't customize DataGrid headers but leave buttons default—visual incoherence

✅ Do systematically customize or leave all as theme default

**Output:** Customization strategy locked (theme as-is vs. selective overrides)

---

## 10. Stage 4 Decision Checklist

**Upon completion, confirm the following decisions are locked:**

### Theme Selection
- ✅ Premium theme selected: **Office2019Colorful** (recommended) or **HighContrastBlack** (accessibility-first)
- ✅ Assembly dependencies confirmed:
  - **Office2019:** `Syncfusion.Office2019Theme.WinForms` (latest standard for new projects)
  - **HighContrastBlack:** `Syncfusion.HighContrastTheme.WinForms` (WCAG AAA accessibility)
- ✅ Rationale documented (why this theme for this application)
- ✅ Theme accessibility level confirmed: **AA (Office2019)** or **AAA (HighContrastBlack)**
- ⚠️ Legacy built-in themes (Office2007/2010/2013) only if backward compatibility required

### Color System Architecture
- ✅ Base theme styling approved (as-is or customized)
- ✅ Brand colors defined (if customizing)
- ✅ Semantic color roles locked (primary, success, warning, error, info)
- ✅ Color contrast verified (WCAG AA minimum, 4.5:1 for text)
- ✅ Brand integration approach documented (theme only vs. selective overrides)

### Typography & Spacing
- ✅ Font family confirmed (Segoe UI standard, custom if needed)
- ✅ Typography sizes locked (Form Title, Section Headers, Body, Small Text)
- ✅ Spacing grid confirmed (4px multiples standard)
- ✅ Control sizing standards documented (minimum 24×24, recommended 44×44)
- ✅ Application layout spacing standards documented

### Accessibility & WCAG 2.2
- ✅ Keyboard navigation plan confirmed
- ✅ Tab order strategy locked
- ✅ Focus indicator visibility confirmed (HideSelection = false)
- ✅ Color contrast verified (4.5:1 minimum for normal text)
- ✅ Accessible names/descriptions strategy for custom controls
- ✅ Target size requirements confirmed (24×24 minimum)
- ✅ Error handling and validation messaging approach documented

### Dark Mode
- ✅ Dark mode support decided (yes/no)
- ✅ Implementation approach locked (if yes):
  - Fixed dark theme OR
  - Runtime theme toggle OR
  - System dark mode detection
- ✅ Color contrast verified for dark theme (if applicable)

### Assembly & Deployment
- ✅ Required assemblies identified:
  - Built-in themes: No extra assemblies
  - Office2016: `Syncfusion.Office2016Theme.WinForms.dll`
  - Office2019: `Syncfusion.Office2019Theme.WinForms.dll`
  - HighContrast: `Syncfusion.HighContrastTheme.WinForms.dll`
- ✅ Assembly loading sequence documented (Program.cs)
- ✅ Theme application scope decided (application-wide / form-level / mixed)

### Custom Customization (if needed)
- ✅ Customization approach locked (theme as-is vs. selective overrides)
- ✅ Brand color constants defined (if customizing)
- ✅ Customization testing plan documented
- ✅ Control coverage for customization identified

### Implementation References
- ✅ **[syncfusion-themes.md](syncfusion-themes.md)** reviewed for detailed theme reference
- ✅ **[winforms-dotnet-standards.md](winforms-dotnet-standards.md)** reviewed for accessibility and standards
- ✅ Ready to proceed to Stage 5 with all theming decisions locked

---

## 11. What Stage 5 Does With These Decisions

Stage 5 (Code Generation) uses your Stage 4 theming decisions to generate:

- **Program.cs setup** with correct `SkinManager` configuration and assembly loading
- **Syncfusion theme application** with chosen theme (built-in or premium)
- **Custom color overrides** if you documented brand customization
- **Color definitions** as constants or configuration
- **Form layouts** respecting your documented spacing standards
- **Accessibility implementation** (accessible names, descriptions, tab order)
- **Dark mode support** (if you chose yes)
- **WCAG 2.2 AA compliance** built into controls (keyboard, focus, contrast)

Stage 5 generates *implementation*, not *decisions*. The decisions you locked in Stage 4 ensure Stage 5 output is consistent and coherent with your WinForms/.NET standards.

---

### For All Windows Forms Projects:
- ✅ Syncfusion theme applied correctly (matches locked selection)
- ✅ Assembly loading sequence verified (built-in or premium)
- ✅ WCAG 2.2 AA accessibility compliance (contrast, keyboard, focus)
- ✅ Proper C# naming conventions and type safety
- ✅ .NET security standards enforced (no hardcoded secrets)
- ✅ Resource management and disposal patterns correct
- ✅ Performance optimizations (BeginUpdate/EndUpdate, async patterns)
- ✅ Build successful, no compilation errors

**Output:** Production-ready WinForms code aligned with Stage 4 theming and .NET standards

---

## References & Further Reading

For implementation guidance:
- **[syncfusion-themes.md](syncfusion-themes.md)** — Complete theme reference, assembly details, advanced customization
- **[winforms-dotnet-standards.md](winforms-dotnet-standards.md)** — WCAG 2.2 AA accessibility, .NET security, performance, code quality standards

- **[stage-1-intent-analysis.md](stage-1-intent-analysis.md)** — Project requirements and intent
- **[stage-2-project-detection.md](stage-2-project-detection.md)** — Project structure detection
- **[stage-3-layout-analysis.md](stage-3-layout-analysis.md)** — Layout and form design

---

**Stage 4 Complete:** Theming decisions locked and ready for Stage 5 code generation.
