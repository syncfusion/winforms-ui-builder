# Syncfusion WinForms Theming Resources

**⚠️ MANDATORY:** After selecting your WinForms theme, you MUST consult **Skill: syncfusion-winforms-theming** for detailed implementation guidance before proceeding to Stage 5.

## Modern Theming with SkinManager (Recommended)

Syncfusion recommends using the **SkinManager** for WinForms projects. This allows for application-wide theme application and unified styling of all Syncfusion controls from a single entry point.

### Implementation Checklist:
1. **Reference Assembly:** Ensure `Syncfusion.Shared.WinForms` is installed via NuGet.
2. **Theme Assembly:** For premium themes, install the assembly for your specific theme (e.g., `Syncfusion.Office2019Theme.WinForms`).
3. **Application Entry:** Call `SkinManager.LoadAssembly()` and set `SkinManager.ApplicationVisualTheme` in `Program.cs` — before `Application.Run()`.
4. **Scope:** For form-level or control-level theming, set the `ThemeName` property on the individual `SkinManager` instance or control after `InitializeComponent()`.

## Quick Reference: Syncfusion WinForms Themes

| Theme Name | Assembly / NuGet Package | Design System |
|---|---|---|
| **Office2007Blue** | Built into `Syncfusion.Shared.WinForms` | Classic Office 2007, blue accent |
| **Office2007Black** | Built into `Syncfusion.Shared.WinForms` | Classic Office 2007, dark variant |
| **Office2007Silver** | Built into `Syncfusion.Shared.WinForms` | Classic Office 2007, neutral silver |
| **Office2010Blue** | Built into `Syncfusion.Shared.WinForms` | Office 2010, blue accent |
| **Office2010Black** | Built into `Syncfusion.Shared.WinForms` | Office 2010, dark variant |
| **Office2010Silver** | Built into `Syncfusion.Shared.WinForms` | Office 2010, neutral silver |
| **Office2013White** | Built into `Syncfusion.Shared.WinForms` | Modern flat, bright professional |
| **Office2013DarkGray** | Built into `Syncfusion.Shared.WinForms` | Modern flat, eye-friendly dark |
| **Office2013Black** | Built into `Syncfusion.Shared.WinForms` | High contrast dark, WCAG AAA |
| **Metro** | Built into `Syncfusion.Shared.WinForms` | Flat minimal, clean aesthetics |
| **Office2016Colorful** | `Syncfusion.Office2016Theme.WinForms` *(Sf controls only)* | Contemporary Office 2016 |
| **Office2016DarkGray** | `Syncfusion.Office2016Theme.WinForms` *(Sf controls only)* | Contemporary Office 2016, dark |
| **Office2016Black** | `Syncfusion.Office2016Theme.WinForms` *(Sf controls only)* | Contemporary Office 2016, high contrast |
| **Office2019Colorful** | `Syncfusion.Office2019Theme.WinForms` | Latest Office aesthetic, vibrant |
| **HighContrastBlack** | `Syncfusion.HighContrastTheme.WinForms` | WCAG AAA, accessibility-first |

**Note:** Built-in themes require no extra assembly beyond `Syncfusion.Shared.WinForms`. Premium themes require `LoadAssembly()` in `Program.cs` before any form is created.

## By Use Case:

### Applying a Theme (SkinManager)
- Call `SkinManager.LoadAssembly(typeof(Office2019Theme).Assembly)` in `Program.cs` for premium themes.
- Set `SkinManager.ApplicationVisualTheme = "Office2019Colorful"` before `Application.Run()`.
- Refer to **Skill: syncfusion-winforms-theming** → **getting-started.md**.

### Implementing Dark Mode
- Use `Office2013Black`, `Office2016DarkGray`, or `HighContrastBlack` for dark or high-contrast appearances.
- Toggle themes at runtime by reassigning `SkinManager.ApplicationVisualTheme` and calling `Refresh()` on open forms.
- Refer to **Skill: syncfusion-winforms-theming** → **theme-management.md**.

### Customizing Colors (ThemeStyle)
- Each Syncfusion control exposes a `ThemeStyle` property for per-control appearance overrides.
- Define brand colors as `System.Drawing.Color` values and assign them to `ThemeStyle` sub-properties (e.g., `BorderColor`, `BackColor`, `HeaderStyle.BackColor`).
- Refer to **Skill: syncfusion-winforms-theming** → **advanced-customization.md**.

### Using Icons
- Refer to **Icon Library** for:
  - Setting up `ImageList` components and assigning them to controls via `ImageList` and `ImageIndex` properties
  - Icon sizing conventions for toolbar buttons, tree nodes, and grid rows
  - Icon state variations (normal, disabled, highlighted) using `ImageList` index mapping

### Advanced Theming
- Refer to **Advanced Features** for:
  - Form-level vs control-level `ThemeName` scoping for mixed-theme layouts
  - Font customization across WinForms controls via `Font` property and `AutoScaleMode`
  - Per-monitor DPI-aware scaling using `AutoScaleMode.Dpi` and high-DPI manifest settings
