# Stage 4: Theming & Design System Selection

**Purpose:** Lock all design system decisions before Stage 5 code generation. This stage is decision-making only — no code is generated here.

**Input:** Stage 2 (project config, .NET version) + Stage 3 (control-mapping.json)
**Output:** All decisions below locked and ready for Stage 5 implementation.

---

## Table of Contents

1. [WinForms Application Philosophy](#1-winforms-application-philosophy)
2. [Syncfusion WinForms Theme Alignment](#2-syncfusion-winforms-theme-alignment)
3. [Color System Architecture](#3-color-system-architecture)
4. [Spacing & Typography Systems](#4-spacing--typography-systems)
5. [Responsive Strategy (DPI-Aware Scaling)](#5-responsive-strategy-dpi-aware-scaling)
6. [Accessibility Standards](#6-accessibility-standards)
7. [SkinManager Token Architecture](#7-skinmanager-token-architecture)
8. [Syncfusion WinForms Control Integration](#8-syncfusion-winforms-control-integration)
9. [Event-Driven Integration Bridge](#9-event-driven-integration-bridge-critical)
10. [Load Your Application Reference (MANDATORY)](#10-load-your-application-reference-mandatory)
11. [Stage 4 Decision Checklist](#11-stage-4-decision-checklist)
12. [What Stage 5 Does With These Decisions](#12-what-stage-5-does-with-these-decisions)

---

## ⛔ ERROR HANDLING: Theme & Assembly Issues

**Common errors in Stage 4–5:**
- ❌ Theme not applying: controls render without Syncfusion styling
- ❌ `TypeInitializationException` or `FileNotFoundException` for theme assembly
- ❌ Build fails: "Type not found" for Syncfusion theme types

**Root cause:** Theme assembly not loaded before form creation, or incorrect assembly loading order.

**Mandatory fixes:**
1. ✅ Load theme assembly BEFORE `Application.Run()` in `Program.cs`
2. ✅ Set `SkinManager.ApplicationVisualTheme` AFTER loading the assembly and BEFORE `Application.Run()`
3. ❌ NEVER set `SkinManager.ApplicationVisualTheme` inside a Form constructor or `Load` event — too late
4. ⛔ **MANDATORY: Verify the theme NuGet package is installed before referencing its assembly type (e.g., `typeof(Office2019Theme)`)**
5. ✅ Register the Syncfusion license key before any Syncfusion API call
6. ⛔ If build fails: Check skill file for correct `SkinManager` API pattern before modifying code

---

## 1. WinForms Application Philosophy

Confirm the application type detected in Stage 2. This choice drives all downstream theme, color, and layout decisions.

| Application Type | Design Priority | Syncfusion Theme Pairing |
|---|---|---|
| **Enterprise** | Data density, task efficiency | Office2019Colorful or HighContrastBlack |
| **Internal/LOB** | Domain workflows, expert users | Office2019Colorful or Office2016Colorful |
| **Data Entry** | Speed, clarity, keyboard efficiency | Office2019Colorful or Office2013White |
| **Accessibility-First** | WCAG AAA compliance, vision support | HighContrastBlack |

**Rules:**
- ✅ Confirm or override Stage 2's detected type — document the reason if overriding
- ❌ Do not mix application philosophies (e.g., minimal aesthetics in a dense data-entry form)
- ✅ Proceed to Section 10 after confirming application type

---

## 2. Syncfusion WinForms Theme Alignment

Select one Syncfusion theme. This choice is **locked** — it determines the NuGet package installed in Stage 6 and the `SkinManager` call generated in Stage 5.

### Theme Selection

| Scenario | Theme |
|---|---|
| Modern Office appearance (new projects — default) | `Office2019Colorful` |
| Accessibility-first / WCAG AAA compliance | `HighContrastBlack` |
| Contemporary Office 2016 appearance | `Office2016Colorful` |
| Follows OS / legacy enterprise standards | `Office2013White` / `Office2013DarkGray` |
| Maximum dark mode contrast | `Office2013Black` |

### Platform Compatibility

| Platform | Supported Themes | Min Syncfusion |
|---|---|---|
| .NET Framework 4.6.2 / 4.7.2 | All themes | 2023.3 |
| .NET 8.0+ | All themes | 2024.1 |
| .NET 6.0 – 7.0 | All themes | 2023.3 |

### Theme NuGet Packages

| Theme | NuGet Package |
|---|---|
| `Office2019Colorful` | `Syncfusion.Office2019Theme.WinForms` |
| `HighContrastBlack` | `Syncfusion.HighContrastTheme.WinForms` |
| `Office2016Colorful` | `Syncfusion.Office2016Theme.WinForms` |
| Built-in (`Office2013White`, `Office2013Black`, etc.) | No extra package — included in `Syncfusion.Core.WinForms` |

**Version rule:** Theme NuGet package version MUST match all other Syncfusion packages detected in Stage 2.

---

## 3. Color System Architecture

Define a semantic color palette. When using Syncfusion themes, colors are provided by `SkinManager` — do NOT hardcode control colors directly. Custom colors are only needed for app-level branding elements not covered by the Syncfusion theme.

**Required color roles (custom overrides only — when NOT relying solely on Syncfusion theme colors):**
- **Primary** — brand color, CTAs, key actions
- **Semantic** — success (`#4CAF50`), warning (`#FFC107`), error (`#F44336`), info (`#2196F3`)
- **Neutral scale** — text, backgrounds, borders
- **Surface** — panels, containers (optional)

**Rules:**
- ✅ Define brand colors as `static readonly Color` constants in a `BrandColors` class
- ✅ Verify contrast ≥ 4.5:1 for all text and UI controls (WCAG AA)
- ❌ Do NOT hardcode `Color.FromArgb(...)` directly on controls — always reference a named constant
- ❌ Do NOT override `BackColor` / `ForeColor` on individual controls unless a documented brand requirement mandates it
- ❌ Do NOT deviate from the selected theme's visual language without documenting the reason

**Dark mode decision (make now):**
- Light only → no action needed; use `Office2019Colorful` or `Office2013White`
- Dark support → use a dark theme variant (`Office2013Black`, `Office2013DarkGray`); apply via `SkinManager` at startup or via a runtime toggle

---

## 4. Spacing & Typography Systems

### Spacing (DPI-Aware)

Use a 4px base grid. WinForms layout uses logical pixel values; set `AutoScaleMode = AutoScaleMode.Dpi` on all Forms to ensure correct scaling across DPI settings.

| Token | Value | Usage |
|---|---|---|
| `SpaceXSmall` | 4px | Icon-to-text gaps, minimal internal padding |
| `SpaceSmall` | 8px | Control padding, tight grouping |
| `SpaceMedium` | 12px | Standard field padding, form field gaps |
| `SpaceLarge` | 16px | Section-to-section spacing |
| `SpaceXLarge` | 24px | Major layout breaks, form area separation |

- ❌ Never hardcode pixel values directly on controls — reference spacing constants

### Typography

Use a consistent size scale. Set fonts explicitly on Forms and controls; do not rely on ambient inheritance.

| Token | Font | Size | Usage |
|---|---|---|---|
| `FontSizeSmall` | Segoe UI | 9pt | Tooltips, status bar, help text |
| `FontSizeBody` | Segoe UI | 10pt | Labels, inputs, standard content |
| `FontSizeLarge` | Segoe UI | 11pt | Subheadings, group labels |
| `FontSizeHeading` | Segoe UI | 13pt | Section headers |
| `FontSizeTitle` | Segoe UI | 16pt | Form title, main window header |

- Minimum body font: **9pt** (96 DPI baseline)
- Use **Segoe UI** as the default — it is the Windows native UI font

---

## 5. Responsive Strategy (DPI-Aware Scaling)

**Design approach:** Anchor- and dock-based fluid layouts — no fixed absolute coordinates. Forms can be resized by the user.

| Window Category | Min Size | Use Case |
|---|---|---|
| Small | 400×300 | Dialogs, utilities, pickers |
| Medium | 800×600 | Standard data-entry / LOB forms |
| Large | 1024×768+ | Dashboards, multi-panel layouts |

**Layout strategy:**
- Use `TableLayoutPanel` for grid-style multi-column layouts with `*` percent-based columns
- Use `FlowLayoutPanel` for single-axis dynamic content
- Use `Anchor` (`Top | Left | Right | Bottom`) on key controls for proportional resize
- Use `Dock = Fill` for primary content areas (e.g., data grids)
- ❌ Do not use hardcoded pixel coordinates (`Location = new Point(x, y)`) for layout — use anchoring and docking

**DPI awareness:**
- Set `AutoScaleMode = AutoScaleMode.Dpi` on every Form
- Set `AutoScaleDimensions = new SizeF(96F, 96F)` as the design-time baseline
- Enable per-monitor DPI awareness in `app.manifest` (supports multi-monitor setups)
- Set `MinimumSize` on Forms only where minimum usability requires it

---

## 6. Accessibility Standards

### Keyboard Navigation

- Minimum click/touch target: **24×24 pixels** (WCAG 2.2 AA); **44×44 pixels** recommended for touch-accessible interfaces
- Minimum spacing between interactive targets: **8px**
- Color contrast: **≥ 4.5:1** for text and UI controls (WCAG 2.1 AA)
- Set `AccessibleName` and `AccessibleDescription` on all interactive Syncfusion controls and icon-only buttons
- Keyboard navigation: logical `TabIndex` order (left-to-right, top-to-bottom); `HideSelection = false` on all input controls; Escape closes dialogs; Enter activates default buttons

### Focus Visibility

- ❌ Never set `HideSelection = true` — this hides the focus indicator and violates WCAG
- ✅ Ensure `TabStop = true` for all interactive controls
- ✅ Set `AcceptButton` and `CancelButton` on every `Form` that has OK/Cancel actions

### Error & Validation Messaging

- Errors must be surfaced as visible `Label` text near the offending field — do not rely on colour alone
- Update `AccessibleDescription` on the input control to reflect the current error state
- Use `ErrorProvider` for field-level inline validation indicators

---

## 7. SkinManager Token Architecture

### Theme Application Approach

**When using Syncfusion themes:**
❌ Do NOT set `BackColor` / `ForeColor` directly on themed Syncfusion controls
❌ Do NOT override control colors in the Form constructor before the theme is applied

**Instead:**
✅ Theme colors, borders, and control styling are provided by `SkinManager` at startup
✅ If custom brand colors are needed: define them in a `static BrandColors` class and apply via `ThemeStyle` properties after theme load
✅ Keep custom color overrides completely separate from `SkinManager` theme application

### Semantic Naming Convention (Mandatory)

Use role-based names, not value-based names:

| ❌ Descriptive (avoid) | ✅ Semantic (use) |
|---|---|
| `Color204Blue` | `BrandColors.Primary` |
| `Padding16` | `Spacing.Large` |
| `Font14px` | `Typography.HeadingSize` |

### Resource Hierarchy

1. **Primitives** — base `Color` and `int` spacing / font-size constants
2. **Semantic** — role-based constants composed from primitives (`BrandColors.Primary`, `Spacing.Medium`)
3. **Control-level** — sparingly, only for control-specific overrides applied via `ThemeStyle`

---

## 8. Syncfusion WinForms Control Integration

### Theme Application via SkinManager (MANDATORY)

✅ Apply Syncfusion themes at startup using `SkinManager`.
❌ Do NOT set themes after `Application.Run()` has been called.

**Program.cs — startup sequence:**
```csharp
using Syncfusion.Licensing;
using Syncfusion.Windows.Forms;

static class Program
{
    [STAThread]
    static void Main()
    {
        SyncfusionLicenseProvider.RegisterLicense(
            Environment.GetEnvironmentVariable("SYNCFUSION_LICENSE_KEY"));

        // For built-in themes (Office2013White, Office2013Black, etc.):
        // SkinManager.ApplicationVisualTheme = "Office2013White";

        // For premium themes — load assembly first, then set theme:
        SkinManager.LoadAssembly(typeof(Office2019Theme).Assembly);
        SkinManager.ApplicationVisualTheme = "Office2019Colorful";

        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new MainForm());
    }
}
```

**For further customization:** Refer to `skills/syncfusion-winforms-theming/SKILL.md` for `ThemeStyle` overrides, brand color application, and runtime theme switching.

### Custom Resource Coordination

- ✅ If custom brand colors are needed (not covered by the Syncfusion theme): define them in `BrandColors` constants and apply via `control.ThemeStyle` properties
- ✅ Reference named constants in all color assignments — never hardcode `Color.FromArgb(...)` inline on controls
- ❌ Do not set `control.BackColor = Color.Blue` directly — use a `BrandColors` constant and apply via `ThemeStyle`
- ❌ Do not apply custom colors before the `SkinManager` theme is loaded — theme load will overwrite them

### Runtime Issue Prevention

| Issue | Prevention |
|---|---|
| Theme NuGet package not installed | Stage 6 MUST install `Syncfusion.<ThemeName>.WinForms` matching locked theme |
| `typeof(Office2019Theme)` not found | Stage 6 MUST include the corresponding theme NuGet package |
| Version mismatch between theme and control packages | All Syncfusion packages use the version detected in Stage 2 |
| Theme not visible at runtime | Confirm `SkinManager` calls precede `Application.Run()` in `Program.cs` |

---

## 9. Event-Driven Integration Bridge (CRITICAL)

Every interactive UI element defined in Stage 4 must have a corresponding event handler or data-binding connection declared here. Stage 5 uses this mapping to wire all interactions — unhandled controls will not function.

### Mandatory Wiring Rules

| UI Element | Required Connection | Example |
|---|---|---|
| Input field (`SfTextBoxExt`, `TextBox`) | `TextChanged` handler or `DataBindings` | `nameInput.DataBindings.Add("Text", model, "Name")` |
| Action button (`ButtonAdv`, `Button`) | `Click` event handler in code-behind or Presenter | `submitButton.Click += OnSubmitClick;` |
| Selection control (`ComboBoxAdv`, `SfDataGrid`) | `SelectedIndexChanged` / `SelectionChanged` handler | `grid.SelectionChanged += OnRowSelected;` |
| Toggle / checkbox | `CheckedChanged` handler | `rememberMeCheck.CheckedChanged += OnRememberMeChanged;` |
| Navigation (form transition) | Handled in event handler — open target Form | `new DashboardForm().Show(); this.Hide();` |
| Error / status display | Set `Label.Text` and `AccessibleDescription` in handler | `errorLabel.Text = "Email is required";` |

**Validation gate:**
```
FOR EACH interactive UI element in control-mapping.json:
  IF no Click, TextChanged, or DataBindings connection declared
  → FAIL: "No event wiring on '<elementId>' — UI interaction will not work
           Fix: add event handler or DataBindings call in Stage 5 code generation"

FOR EACH navigation flow (e.g., Login → Dashboard):
  IF navigation is not triggered via an event handler that opens the target Form
  → FAIL: "Broken UI workflow: '<flow>' not wired via event handler
           Fix: implement form transition in event handler; show target Form on execute"
```

### Common Scenario Checklist

| Scenario | Wiring Requirement |
|---|---|
| Login button → authenticate → navigate | `Click` handler validates credentials; opens `DashboardForm` on success |
| Form field → validate on change | `TextChanged` or `Validating` event updates error label and `AccessibleDescription` |
| Grid row selection → detail view | `SelectionChanged` handler populates detail panel or opens detail Form |
| Cancel / close button | `Click` handler calls `this.Close()` |

❌ Do NOT leave any Button, input, or navigation trigger without a declared event handler or DataBindings connection.

---

## 10. Load Your Application Reference (MANDATORY)

Based on the application type confirmed in Section 1, load the corresponding skill reference before proceeding to Stage 5. Reference files are located at any of:

```
<skills-root>/syncfusion-winforms-ui-builder/references/
<skills-root>/syncfusion-winforms-theming/SKILL.md
```

Where `<skills-root>` is one of: `.codestudio/skills`, `.agent/skills`, `.agents/skills`, `.github/skills`, `skills`

| Application Type | Theme Focus | Key Reference |
|---|---|---|
| **Enterprise** | Office2019Colorful — data density, professional | `references/syncfusion-themes.md` + `references/winforms-dotnet-standards.md` |
| **Internal/LOB** | Office2019Colorful — expert workflows, productivity | `references/syncfusion-themes.md` + `references/winforms-dotnet-standards.md` |
| **Data Entry** | Office2013White or Office2019Colorful — speed and clarity | `references/syncfusion-themes.md` |
| **Accessibility-First** | HighContrastBlack — WCAG AAA, vision support | `references/syncfusion-themes.md` + `references/winforms-dotnet-standards.md` |

⛔ **You cannot proceed to Stage 5 without loading your application reference.**

---

## 11. Stage 4 Decision Checklist

Confirm all items are locked before proceeding.

### Application & Theme
- ✅ Application type confirmed (Enterprise / LOB / Data Entry / Accessibility-First)
- ✅ Syncfusion WinForms theme selected and documented
- ✅ Theme NuGet package name recorded (e.g., `Syncfusion.Office2019Theme.WinForms`)
- ✅ Application reference file loaded (Section 10)

### Color System
- ✅ Primary, semantic, and neutral palette defined
- ✅ Custom colors (if any) defined in `BrandColors` constants — NOT hardcoded inline on controls
- ✅ Dark mode decision made (light only / use matching dark theme name)
- ✅ WCAG contrast ≥ 4.5:1 verified

### Spacing & Typography
- ✅ 4px base spacing grid defined (constants or inline values consistent with grid)
- ✅ Typography size scale defined (Segoe UI, 9pt–16pt range)
- ✅ Minimum body font ≥ 9pt; forms use `AutoScaleMode = AutoScaleMode.Dpi`

### Responsive Design
- ✅ Fluid layout strategy confirmed (`TableLayoutPanel` / `Anchor` / `Dock`)
- ✅ Per-monitor DPI awareness enabled in `app.manifest`
- ✅ `AutoScaleMode = AutoScaleMode.Dpi` set on all Forms
- ✅ `MinimumSize` set on Forms only where required

### Accessibility
- ✅ `TabIndex` order documented (left-to-right, top-to-bottom)
- ✅ `HideSelection = false` confirmed on all input controls
- ✅ `AcceptButton` and `CancelButton` set on all applicable Forms
- ✅ Touch/click targets ≥ 24×24 pixels (44×44 recommended)
- ✅ `AccessibleName` and `AccessibleDescription` plan confirmed for all interactive controls

### SkinManager Token Architecture
- ✅ Semantic naming applied (role-based constants, not value-based)
- ✅ Custom colors (if any) in `BrandColors` class — applied via `ThemeStyle` after theme load
- ✅ Syncfusion theme applied via `SkinManager` — NOT via direct control color assignment

### Event-Driven Integration
- ✅ Every interactive control (button, input, grid) has a declared event handler or DataBindings connection
- ✅ Every navigation flow (e.g., Login → Dashboard) wired via a `Click` event handler
- ✅ Error/status display updated via handler to both `Label.Text` and `AccessibleDescription`
- ✅ No UI control left without a declared wiring connection

### Syncfusion Integration
- ✅ `SkinManager.LoadAssembly(...)` called before `SkinManager.ApplicationVisualTheme` (premium themes)
- ✅ `SkinManager.ApplicationVisualTheme` set before `Application.Run()` in `Program.cs`
- ✅ All Syncfusion package versions consistent with Stage 2 detection

---

## 12. What Stage 5 Does With These Decisions

Stage 5 generates implementation — not decisions. It uses Stage 4 output to produce:

| Stage 4 Decision | Stage 5 Output |
|---|---|
| Locked theme + SkinManager pattern | `SkinManager.LoadAssembly()` + `SkinManager.ApplicationVisualTheme` in `Program.cs` |
| Custom color resources (if any) | `static BrandColors` class; `ThemeStyle` overrides applied post-theme-load |
| Semantic resource naming | All color/spacing assignments reference named constants, never hardcoded values |
| Responsive layout strategy | `TableLayoutPanel` with percent sizing; `Anchor` / `Dock` on controls; no hardcoded coordinates |
| Accessibility standards | `AccessibleName`, `AccessibleDescription`, `TabIndex` order, `HideSelection = false` on all interactive controls |
| **Event-driven integration map (Section 9)** | Every button → `Click` handler; every input → `TextChanged` / `DataBindings`; every navigation → Form transition in handler |
| Application reference (Section 10) | Code patterns aligned to application type |

✅ All Stage 5 code is consistent with and traceable to decisions locked here.

**Output:** Production-ready WinForms code aligned with Stage 4 design decisions
