# WinForms .NET Standards & Compliance Reference

**Version:** 2.0.0  
**Last Updated:** May 5, 2026  
**Purpose:** WCAG 2.2 AA accessibility, .NET security, performance, and code quality standards enforced during Stage 5 validation for WinForms applications

---

## Table of Contents

1. [Accessibility Standards (WCAG 2.2 AA)](#accessibility-standards-wcag-22-aa)
2. [Security Standards](#security-standards)
3. [Performance Standards](#performance-standards)
4. [Code Quality Standards](#code-quality-standards)
5. [Validation Checklist](#validation-checklist)
6. [Auto-Fix Rules](#auto-fix-rules)

---

## Accessibility Standards (WCAG 2.2 AA)

### WCAG Principles: POUR

| Principle | Description |
|-----------|-------------|
| **P**erceivable | Content can be perceived through different senses (text alternatives, contrast, clear structure) |
| **O**perable | Interface can be operated by all users (keyboard, no traps, sufficient target size) |
| **U**nderstandable | Content and interface are understandable (clear language, predictable, error guidance) |
| **R**obust | Content works with assistive technologies (semantic HTML, proper ARIA, screen reader compatible) |

---

### Perceivable: Content Must Be Perceivable

#### 1.1 Text Alternatives

**Image controls require descriptive alt text:**

```csharp
// ❌ Missing alt text
pictureBox.Image = Image.FromFile("chart.png");

// ✅ Descriptive alt text using AccessibleDescription
pictureBox.AccessibleDescription = "Bar chart showing 40% increase in Q3 sales";
pictureBox.AccessibleName = "Sales Chart";
pictureBox.Image = Image.FromFile("chart.png");

// ✅ Decorative image (no description needed)
pictureBox.AccessibleDescription = "";
pictureBox.Image = Image.FromFile("decorative-border.png");

// ✅ Complex image with tooltip
pictureBox.AccessibleDescription = "2024 market trends infographic";
toolTip.SetToolTip(pictureBox, "2024 market trends - 40% growth in Q1, 25% in Q2, 35% in Q3");
pictureBox.Image = Image.FromFile("infographic.png");
```

**Icon buttons and image-only buttons need accessible names:**

```csharp
// ❌ No accessible name
Button btnMenu = new Button { Image = menuIcon };

// ✅ Using AccessibleName and AccessibleDescription
Button btnMenu = new Button
{
    Image = menuIcon,
    AccessibleName = "Menu",
    AccessibleDescription = "Open application menu",
    AccessibleRole = AccessibleRole.PushButton,
    Text = "" // Empty text since icon is used
};

// ✅ Using tooltip for additional accessibility context
Button btnRefresh = new Button
{
    Image = refreshIcon,
    AccessibleName = "Refresh",
    AccessibleDescription = "Reload data from server"
};
toolTip.SetToolTip(btnRefresh, "Refresh - Reload data from server");
```

**Validation Rules:**
- [ ] All informative images have descriptive alt text
- [ ] Decorative images have empty alt (`alt=""`)
- [ ] Icon-only buttons have `aria-label` or hidden text
- [ ] Complex images have `aria-describedby` with detailed description

---

#### 1.2 Media Alternatives

```csharp
// ❌ Video without captions
var mediaPlayer = new AxWindowsMediaPlayer();
mediaPlayer.URL = "video.mp4";
this.Controls.Add(mediaPlayer);

// ✅ Video with subtitle/caption file reference
var mediaPlayer = new AxWindowsMediaPlayer();
mediaPlayer.URL = "video.mp4";
mediaPlayer.AccessibleDescription = "Training video with captions available";
// Load subtitle file (SRT or VTT format)
LoadSubtitles("video_captions.srt");
this.Controls.Add(mediaPlayer);

// ✅ Audio player with transcript
Panel audioPanel = new Panel { Height = 100 };
var axWmp = new AxWindowsMediaPlayer();
axWmp.URL = "podcast.mp3";
audioPanel.Controls.Add(axWmp);

Label transcriptLabel = new Label
{
    Text = "Transcript:",
    AutoSize = false,
    Height = 20
};
TextBox transcriptBox = new TextBox
{
    Multiline = true,
    ReadOnly = true,
    Text = "Full transcript text here...",
    Dock = DockStyle.Fill
};
audioPanel.Controls.Add(transcriptLabel);
audioPanel.Controls.Add(transcriptBox);
```

**Validation Rules:**
- [ ] Videos have captions
- [ ] Videos have audio descriptions
- [ ] Auto-playing audio can be paused/stopped

---

#### 1.4 Color Contrast

**Minimum Ratios (WCAG 2.2 AA):**

| Text Type | Minimum Ratio |
|-----------|---------------|
| Normal text (< 11pt / < 9pt bold) | 4.5:1 |
| Large text (≥ 11pt / ≥ 9pt bold) | 3:1 |
| UI components & focus indicators | 3:1 |

```csharp
// ✓ GOOD - High contrast button
Button button = new Button
{
    ForeColor = Color.Black,        // Dark text
    BackColor = Color.White,        // Light background
    // Contrast ratio: 21:1 ✓
};

// ✗ BAD - Low contrast text (fails)
Label hint = new Label
{
    ForeColor = Color.FromArgb(204, 204, 204),  // Light gray
    BackColor = Color.White,                     // Light background
    // Contrast ratio: 1.9:1 FAILS ✗
};

// ✓ GOOD - Readable text with high contrast
Label label = new Label
{
    ForeColor = Color.Black,
    BackColor = Color.White,
    AutoSize = true,
    Font = new Font("Segoe UI", 9f)
};

// ✓ GOOD - Focus indicator with sufficient contrast
TextBox textBox = new TextBox();
textBox.GotFocus += (s, e) => 
{
    textBox.BorderStyle = BorderStyle.Fixed3D;
    textBox.BackColor = Color.LightBlue;  // Indicates focus
    textBox.BorderStyle = BorderStyle.FixedSingle; // 2px equivalent
};
```

**Don't rely on color alone to convey information:**

```csharp
// ❌ Only red border indicates error
TextBox emailInput = new TextBox();
emailInput.BorderStyle = BorderStyle.FixedSingle;
// On invalid input:
emailInput.BackColor = Color.Red;  // Only color - not accessible!

// ✅ Color + icon + error message
TextBox emailInput = new TextBox
{
    BorderStyle = BorderStyle.FixedSingle
};
PictureBox errorIcon = new PictureBox
{
    Image = Properties.Resources.error_icon,
    SizeMode = PictureBoxSizeMode.AutoSize
};
Label errorMessage = new Label
{
    Text = "Please enter a valid email address",
    ForeColor = Color.Red,
    AutoSize = true
};

// On validation error:
emailInput.BackColor = Color.FromArgb(255, 230, 230);  // Light red
errorIcon.Visible = true;
errorMessage.Visible = true;
```

**Validation Rules:**
- [ ] All text has contrast ≥ 4.5:1 (normal) or 3:1 (large)
- [ ] Focus indicators have contrast ≥ 3:1
- [ ] Placeholder text has minimum 4.5:1 contrast
- [ ] Icons conveying information have 3:1 contrast
- [ ] Information not conveyed by color alone

---

### Operable: Users Must Be Able to Operate the Interface

#### 2.1 Keyboard Accessibility

**All functionality must be accessible via keyboard—no mouse-only actions:**

```csharp
// ❌ BAD - Only mouse events
Button button = new Button();
button.Click += (s, e) => PerformAction();
// Keyboard users cannot activate this button

// ✅ GOOD - Keyboard and mouse support
Button button = new Button
{
    TabIndex = 0,  // Include in tab order
    Text = "Click Me"
};
button.Click += (s, e) => PerformAction();
button.KeyDown += (s, e) =>
{
    if (e.KeyCode == Keys.Enter || e.KeyCode == Keys.Space)
    {
        e.Handled = true;
        PerformAction();
    }
};

// ✅ GOOD - MenuItem has built-in keyboard support
ToolStripMenuItem editMenu = new ToolStripMenuItem("&Edit");
editMenu.ShortcutKeys = Keys.Alt | Keys.E;
editMenu.Click += (s, e) => PerformEdit();
```

**No keyboard traps—users must be able to Tab out of every control:**

```csharp
// ✓ GOOD - Form allows escape to close
public partial class MyForm : Form
{
    public MyForm()
    {
        InitializeComponent();
        this.KeyPreview = true;
    }

    protected override void OnKeyDown(KeyEventArgs e)
    {
        if (e.KeyCode == Keys.Escape)
        {
            this.Close();  // Allow exit
            e.Handled = true;
        }
        base.OnKeyDown(e);
    }
}

// ✗ BAD - Prevents Tab key (keyboard trap)
Panel panel = new Panel();
panel.KeyDown += (s, e) =>
{
    if (e.KeyCode == Keys.Tab)
    {
        e.Handled = true;  // TRAP! Can't leave with Tab
    }
};
```

**Validation Rules:**
- [ ] All interactive elements in tab order (no negative tabindex)
- [ ] Tab order is logical (left-to-right, top-to-bottom)
- [ ] Can Tab into and out of every control
- [ ] Escape key closes modals and dropdowns
- [ ] Enter key submits forms and activates buttons
- [ ] Arrow keys work for lists, dropdowns, tabs

---

#### 2.4 Focus Management

**Users must see where keyboard focus is—focus indicators must be visible:**

```csharp
// ❌ NEVER hide the focus rectangle (accessibility violation)
TextBox textBox = new TextBox();
textBox.HideSelection = true;  // ✗ Focus becomes invisible when unfocused

// ✅ GOOD - Always show focus indicators
TextBox textBox = new TextBox
{
    HideSelection = false,  // Keep focus visible
    TabIndex = 0
};

// ✅ GOOD - Custom focus indicator
Panel customControl = new Panel
{
    BorderStyle = BorderStyle.FixedSingle
};
customControl.GotFocus += (s, e) =>
{
    customControl.BorderStyle = BorderStyle.Fixed3D;
    customControl.BackColor = Color.FromArgb(200, 220, 255);
};
customControl.LostFocus += (s, e) =>
{
    customControl.BorderStyle = BorderStyle.FixedSingle;
    customControl.BackColor = Color.White;
};

// ✅ GOOD - Set TabStop to ensure element can receive focus
Button button = new Button
{
    TabStop = true,  // Include in tab order
    Text = "Submit",
    TabIndex = 0
};
```

**Manage focus when opening modals or dialogs:**

```csharp
// ✓ GOOD - Focus first input when dialog opens
public partial class MyDialog : Form
{
    private TextBox firstInput;

    public MyDialog()
    {
        InitializeComponent();
    }

    protected override void OnLoad(EventArgs e)
    {
        base.OnLoad(e);
        // Focus first input when dialog opens
        firstInput.Focus();
    }

    protected override void OnKeyDown(KeyEventArgs e)
    {
        // Escape closes dialog
        if (e.KeyCode == Keys.Escape)
        {
            this.DialogResult = DialogResult.Cancel;
            this.Close();
            e.Handled = true;
        }
        base.OnKeyDown(e);
    }
}

// ✅ GOOD - Restore focus when dialog closes
Form parentForm = new Form();
MyDialog dialog = new MyDialog();
dialog.ShowDialog(parentForm);
parentForm.Focus();  // Return focus to parent
```

**Validation Rules:**
- [ ] Focus indicators always visible (never outline: none)
- [ ] Focus outline ≥ 2px thick, ≥ 3:1 contrast
- [ ] Focus order logical (top-to-bottom, left-to-right)
- [ ] Focus managed when modal/dialog opens
- [ ] Focused element not obscured by sticky headers/footers

---

#### 2.5 Target Size (NEW in 2.2)

**Interactive targets must be at least 24 × 24 screen pixels:**

```csharp
// ✓ GOOD - Minimum target size (24x24)
Button button = new Button
{
    Width = 24,
    Height = 24,
    Text = "⋮"  // Menu icon
};

// ✓ GOOD - Comfortable target size (recommended 44×44 for touch/accessibility)
Button okButton = new Button
{
    Width = 44,
    Height = 44,
    Text = "OK",
    Font = new Font("Segoe UI", 10f)
};

// ✓ GOOD - Button group with proper spacing
FlowLayoutPanel buttonPanel = new FlowLayoutPanel
{
    AutoSize = true,
    Margin = new Padding(5)
};
for (int i = 0; i < 3; i++)
{
    Button btn = new Button
    {
        Width = 44,
        Height = 44,
        Text = (i + 1).ToString(),
        Margin = new Padding(5)  // Spacing to prevent accidental clicks
    };
    buttonPanel.Controls.Add(btn);
}

// ✓ GOOD - Checkbox with larger target area
CheckBox checkbox = new CheckBox
{
    Width = 44,
    Height = 24,
    Text = "Remember me",
    AutoSize = false
};

// ✓ GOOD - Small icon button with adequate spacing
Panel iconPanel = new Panel
{
    Width = 44,
    Height = 44,
    Margin = new Padding(5)
};
PictureBox icon = new PictureBox
{
    Image = Properties.Resources.minimize_icon,
    Dock = DockStyle.Fill,
    SizeMode = PictureBoxSizeMode.CenterImage
};
iconPanel.Controls.Add(icon);
```

**Exceptions:** Text within paragraphs, system-controlled elements (window chrome), spacing where overlapping circles don't intersect.

**Validation Rules:**
- [ ] All buttons ≥ 24×24 screen pixels
- [ ] All clickable icons ≥ 24×24 screen pixels
- [ ] All form inputs (TextBox, CheckBox) ≥ 24×24 screen pixels
- [ ] Adequate spacing (5-10px minimum) between interactive elements to prevent accidental activation

---

#### 2.5 Dragging Movements Alternative (NEW in 2.2)

**Any action triggered by dragging must offer a keyboard alternative:**

```csharp
// ❌ Drag-only reorder (fails accessibility)
ListBox itemList = new ListBox
{
    AllowDrop = true,
    SelectionMode = SelectionMode.One
};
itemList.DragDrop += (s, e) =>
{
    // Reorder items on drag
    // Keyboard users cannot reorder!
};

// ✅ Drag + button alternatives
Panel listContainer = new Panel();
ListBox itemList = new ListBox
{
    AllowDrop = true,
    SelectionMode = SelectionMode.One,
    Dock = DockStyle.Fill
};

FlowLayoutPanel buttonPanel = new FlowLayoutPanel
{
    Dock = DockStyle.Bottom,
    Height = 40,
    AutoSize = false
};

Button moveUpBtn = new Button
{
    Text = "↑ Move Up",
    Width = 44,
    Height = 24
};
moveUpBtn.Click += (s, e) =>
{
    if (itemList.SelectedIndex > 0)
    {
        object selectedItem = itemList.SelectedItem;
        itemList.Items.RemoveAt(itemList.SelectedIndex);
        itemList.Items.Insert(itemList.SelectedIndex - 1, selectedItem);
    }
};

Button moveDownBtn = new Button
{
    Text = "↓ Move Down",
    Width = 44,
    Height = 24
};
moveDownBtn.Click += (s, e) =>
{
    if (itemList.SelectedIndex < itemList.Items.Count - 1)
    {
        object selectedItem = itemList.SelectedItem;
        itemList.Items.RemoveAt(itemList.SelectedIndex);
        itemList.Items.Insert(itemList.SelectedIndex + 1, selectedItem);
    }
};

buttonPanel.Controls.AddRange(new Control[] { moveUpBtn, moveDownBtn });
listContainer.Controls.AddRange(new Control[] { itemList, buttonPanel });

// ✅ Slider with keyboard support
TrackBar slider = new TrackBar
{
    Minimum = 0,
    Maximum = 100,
    Value = 50,
    TabStop = true
};
// TrackBar already supports Arrow keys by default

// ✅ Alternative: Use spinners for numeric input
NumericUpDown numericInput = new NumericUpDown
{
    Minimum = 0,
    Maximum = 100,
    Value = 50
    // Arrows and +/- buttons provide keyboard/mouse alternatives
};
```

**Applies to:** Sliders, list reordering, drag-to-select, panel resizing, color pickers, and all drag-based interactions.

---

### Understandable: Content Must Be Understandable

#### 3.1 Language & Structure

```csharp
// ✅ Set application language/culture
System.Globalization.CultureInfo.CurrentUICulture = 
    new System.Globalization.CultureInfo("en-US");

// ✅ For multilingual applications, support language selection
public void SetApplicationLanguage(string languageCode)
{
    System.Threading.Thread.CurrentThread.CurrentUICulture = 
        new System.Globalization.CultureInfo(languageCode);
    // Reload UI strings from resources
}

// ✅ Logical UI hierarchy (similar to heading hierarchy)
Form mainForm = new Form { Text = "Main Application" };

GroupBox section1 = new GroupBox { Text = "Primary Section" };
GroupBox section1Sub = new GroupBox { Text = "Subsection" };

Label instruction = new Label { Text = "Detailed instruction text" };

// ✅ Use GroupBox for logical grouping (like semantic sections)
GroupBox contactInfo = new GroupBox
{
    Text = "Contact Information",
    TabIndex = 0
};

// Proper tab order reflects logical hierarchy
TextBox name = new TextBox { TabIndex = 1, AccessibleName = "Name" };
TextBox email = new TextBox { TabIndex = 2, AccessibleName = "Email" };
TextBox phone = new TextBox { TabIndex = 3, AccessibleName = "Phone" };

// ❌ BAD: Illogical tab order (disrupts hierarchy)
TextBox field1 = new TextBox { TabIndex = 5 };
TextBox field2 = new TextBox { TabIndex = 1 };  // Wrong order!
TextBox field3 = new TextBox { TabIndex = 3 };
```

**Validation Rules:**
- [ ] Page language specified in `<html lang="...">`
- [ ] Language changes marked with `lang` attribute
- [ ] Headings use proper hierarchy (h1-h6, no skipping)
- [ ] Headings are descriptive

---

#### 3.2 Predictable Behavior

**Users must be able to predict what happens when they interact:**

```csharp
// ✓ GOOD - Buttons trigger actions, clear purpose
Button navigateButton = new Button
{
    Text = "Open Report",
    AccessibleName = "Open Report",
    AccessibleDescription = "Opens the sales report in a new window"
};
navigateButton.Click += (s, e) =>
{
    ReportForm report = new ReportForm();
    report.Show();
};

Button submitButton = new Button
{
    Text = "Save",
    AccessibleDescription = "Saves changes to database"
};

// ❌ BAD - Unexpected behavior
Button confusedButton = new Button
{
    Text = "Click Me"  // Vague purpose
};
confusedButton.Click += (s, e) =>
{
    // Opens modal? Submits form? Deletes data? User doesn't know!
};

// ✓ GOOD - Focus doesn't trigger changes (read-only highlighting OK)
TextBox textBox = new TextBox();
textBox.GotFocus += (s, e) =>
{
    textBox.BackColor = Color.FromArgb(200, 220, 255);  // OK: visual highlight
};
textBox.LostFocus += (s, e) =>
{
    textBox.BackColor = Color.White;
};

// ❌ BAD - Focus triggers data change
ComboBox dropdown = new ComboBox();
dropdown.GotFocus += (s, e) =>
{
    LoadDataFromDatabase();  // UNEXPECTED! User just navigated with Tab
    SubmitForm();  // WRONG! User didn't intend this
};

// ✓ GOOD - Change event is intentional
ComboBox filter = new ComboBox
{
    AccessibleName = "Sort by"
};
filter.SelectedIndexChanged += (s, e) =>
{
    SortListBox(filter.SelectedItem.ToString());  // OK: user selected something
};
```

**Consistent help (NEW in 2.2)—help mechanisms appear in same relative order:**

```csharp
// ✓ GOOD - Consistent help menu on every form
public partial class MainForm : Form
{
    private void CreateHelpMenu()
    {
        MenuStrip menuStrip = new MenuStrip();
        
        ToolStripMenuItem helpMenu = new ToolStripMenuItem("&Help");
        helpMenu.DropDownItems.Add(
            new ToolStripMenuItem("&Contact Us", null, (s, e) => OpenContactForm())
        );
        helpMenu.DropDownItems.Add(
            new ToolStripMenuItem("&FAQ", null, (s, e) => OpenFAQForm())
        );
        helpMenu.DropDownItems.Add(
            new ToolStripMenuItem("&Help Topics", null, (s, e) => OpenHelpTopics())
        );
        
        menuStrip.Items.Add(helpMenu);
        this.MainMenuStrip = menuStrip;
        this.Controls.Add(menuStrip);
    }
}
```

---

#### 3.3 Forms & Error Handling

**Every input needs an associated label:**

```csharp
// ❌ No label
TextBox emailInput = new TextBox
{
    PlaceholderText = "Email"  // Placeholder is NOT a label
};

// ✅ Explicit label + input
Label emailLabel = new Label
{
    Text = "Email address:",
    AutoSize = true
};
TextBox emailInput = new TextBox
{
    AccessibleName = "Email address",
    AutoCompleteMode = AutoCompleteMode.SuggestAppend,
    AutoCompleteSource = AutoCompleteSource.AllUrl
};

// ✅ With instruction text
Label passwordLabel = new Label { Text = "Password:" };
TextBox passwordInput = new TextBox
{
    AccessibleName = "Password",
    PasswordChar = '•'
};
Label requirements = new Label
{
    Text = "At least 8 characters with one number",
    ForeColor = Color.Gray,
    AutoSize = true
};
passwordInput.AccessibleDescription = "At least 8 characters with one number";

// ✅ Required field indicator
Label nameLabel = new Label { Text = "Name: *" };  // Asterisk indicates required
TextBox nameInput = new TextBox
{
    AccessibleName = "Name (required)",
    AccessibleDescription = "Your full name is required"
};
```

**Error messages must be clear and linked to fields:**

```csharp
// ❌ Unclear error
MessageBox.Show("Error");  // What went wrong?

// ✅ Clear, contextual error messages
void ValidateEmail(string email)
{
    if (string.IsNullOrEmpty(email))
    {
        emailInput.BackColor = Color.FromArgb(255, 230, 230);
        errorMessage.Text = "Email is required";
        errorMessage.ForeColor = Color.Red;
        errorIcon.Visible = true;
        return false;
    }
    
    if (!email.Contains("@"))
    {
        emailInput.BackColor = Color.FromArgb(255, 230, 230);
        errorMessage.Text = "Please enter a valid email address (example: name@domain.com)";
        errorMessage.ForeColor = Color.Red;
        errorIcon.Visible = true;
        return false;
    }
    
    emailInput.BackColor = Color.White;
    errorMessage.Text = "";
    errorIcon.Visible = false;
    return true;
}

// ✅ Error linked to field with accessible description
TextBox emailField = new TextBox
{
    AccessibleDescription = "Please enter a valid email address (example: name@domain.com)"
};
```

**Don't force users to re-enter information (NEW in 2.2):**

```csharp
// ✅ Auto-populate shipping from billing
void OnSameAsBillingChecked(object sender, EventArgs e)
{
    if (sameAddressCheckbox.Checked)
    {
        // Auto-fill shipping with billing data
        shippingStreetInput.Text = billingStreetInput.Text;
        shippingCityInput.Text = billingCityInput.Text;
        shippingStateInput.Text = billingStateInput.Text;
        shippingZipInput.Text = billingZipInput.Text;
    }
    else
    {
        shippingStreetInput.Clear();
        shippingCityInput.Clear();
        shippingStateInput.Clear();
        shippingZipInput.Clear();
    }
}

sameAddressCheckbox.CheckedChanged += OnSameAsBillingChecked;
```

**Login must not rely solely on cognitive tests (NEW in 2.2):**

```csharp
// ❌ Cognitive test only
Button solveButton = new Button { Text = "Solve puzzle to login" };

// ✅ Multiple authentication options
Panel loginPanel = new Panel();
Button passwordButton = new Button
{
    Text = "Sign in with password",
    Width = 150
};
Button emailLinkButton = new Button
{
    Text = "Email me a login link",
    Width = 150
};
Button passkeyButton = new Button
{
    Text = "Sign in with passkey",
    Width = 150
};

// All three options available
```

**Validation Rules:**
- [ ] Every input has associated label
- [ ] Required fields marked with * or text
- [ ] Error messages clear and specific
- [ ] Errors linked via `aria-describedby`
- [ ] Error messages include how to fix
- [ ] Validation happens at appropriate times (blur, submit)
- [ ] Information not re-requested (auto-fill where possible)
- [ ] Login not purely cognitive (offer alternatives)

---

### Robust: Content Must Work with Assistive Technologies

#### 4.1 Native WinForms Controls

**Prefer native WinForms controls—they have accessibility built in:**

```csharp
// ❌ Non-semantic custom control
Panel customButton = new Panel
{
    BackColor = Color.Blue,
    Height = 40,
    Width = 100
};
Label buttonText = new Label { Text = "Submit", Dock = DockStyle.Fill };
customButton.Controls.Add(buttonText);
// Difficult to make accessible, no keyboard support by default

// ✅ Native button (automatic: keyboard, focus, role)
Button submitButton = new Button
{
    Text = "Submit",
    Width = 100,
    Height = 40
    // Keyboard accessible by default
};

// ❌ Custom checkbox control
Panel customCheckbox = new Panel { Width = 30, Height = 30 };
bool isChecked = false;
customCheckbox.Click += (s, e) => isChecked = !isChecked;

// ✅ Native checkbox (simple, accessible)
CheckBox nativeCheckbox = new CheckBox
{
    Text = "I agree to the terms",
    AutoSize = true,
    TabStop = true
};

// ✗ Non-semantic form (just panels)
Panel form = new Panel();
Panel emailField = new Panel();
Panel passwordField = new Panel();

// ✓ Semantic form using GroupBox and native controls
GroupBox form = new GroupBox { Text = "Login Form" };
Label emailLabel = new Label { Text = "Email:" };
TextBox emailInput = new TextBox { AccessibleName = "Email" };
Label passwordLabel = new Label { Text = "Password:" };
TextBox passwordInput = new TextBox 
{ 
    AccessibleName = "Password",
    PasswordChar = '•'
};
Button submitButton = new Button { Text = "Submit" };
```

**Use AccessibleRole and AccessibleName for custom controls only:**

```csharp
// ✓ GOOD - AccessibleName for custom tab control
TabControl tabControl = new TabControl
{
    AccessibleName = "Product Information Tabs"
};
TabPage descPage = new TabPage { Text = "Description" };
TabPage reviewsPage = new TabPage { Text = "Reviews" };
tabControl.TabPages.AddRange(new[] { descPage, reviewsPage });

// ✓ GOOD - AccessibleName for icon-only buttons
Button closeButton = new Button
{
    Text = "×",
    AccessibleName = "Close",
    AccessibleDescription = "Close this dialog",
    AccessibleRole = AccessibleRole.PushButton,
    Width = 24,
    Height = 24
};

// ✓ GOOD - AccessibleDescription for error messages
TextBox emailInput = new TextBox
{
    AccessibleName = "Email"
};
Label errorLabel = new Label
{
    Text = "Error: Invalid email format",
    ForeColor = Color.Red
};
emailInput.AccessibleDescription = "Error: Invalid email format. Please enter a valid email address.";
```

**Validation Rules:**
- [ ] Buttons are `<button>` elements
- [ ] Links are `<a>` elements
- [ ] Forms use `<form>` element
- [ ] Inputs have associated labels
- [ ] No `role`, `aria-*` when native HTML works
- [ ] Icon buttons have `aria-label`
- [ ] Custom components have proper roles
- [ ] Error messages have `aria-describedby`

---

## Testing Accessibility

**Automated tools for WinForms:**
```powershell
# Accessibility Insights for Windows
# https://accessibilityinsights.io/
# Analyze WinForms app for accessibility issues

# NVDA (free screen reader)
# https://www.nvaccess.org/

# Windows Narrator (built-in)
# Win + Ctrl + N

# Inspect.exe (Windows SDK)
# C:\Program Files (x86)\Windows Kits\10\bin\x64\inspect.exe
# Examine control properties and accessibility tree
```

**Manual testing—test with assistive technologies:**
- [ ] **Keyboard navigation:** Tab through entire application, all features accessible
- [ ] **Screen reader (NVDA):** Listen to control names, descriptions, state changes
- [ ] **High Contrast Mode:** Windows Settings > Ease of Access > Display > High Contrast
- [ ] **Windows Magnifier:** Win + Plus key, test at 200%+ magnification
- [ ] **Narrator:** Win + Ctrl + N, verify all content is announced
- [ ] **Tab order:** Verify logical left-to-right, top-to-bottom flow with Inspect.exe
- [ ] **Color contrast:** Use Contrast Analyzer tool or browser extensions
- [ ] **Focus indicators:** Verify focus rectangles are always visible

---

## Security Standards

### 2.1 Input Validation

**Requirement:** Prevent XSS and injection attacks

**What to Check:**

```tsx
// ✗ BAD - Unsanitized user input
const userBio = "<img src=x onerror='alert(1)'>";
<div dangerouslySetInnerHTML={{ __html: userBio }} />

// ✓ GOOD - User input rendered as text (safe)
<div>{userBio}</div>

// ✓ GOOD - Sanitize if HTML is truly needed
import DOMPurify from 'dompurify';
const cleanHTML = DOMPurify.sanitize(userBio);
<div dangerouslySetInnerHTML={{ __html: cleanHTML }} />
```

**Validation Rules:**
- [ ] No `dangerouslySetInnerHTML` with user input
- [ ] No `eval()` or `Function()` constructors
- [ ] No inline event handlers (onClick="code()")
- [ ] User input sanitized before display
- [ ] URL validation before navigation

---

### 2.2 Secrets & Environment Variables

**Requirement:** Never expose API keys or secrets in code

**What to Check:**

```csharp
// ✗ BAD - Hardcoded secret
const string API_KEY = "sk_live_12345abcde";
using (HttpClient client = new HttpClient())
{
    client.DefaultRequestHeaders.Add("Authorization", $"Bearer {API_KEY}");
    // EXPOSED: Secret visible in source code!
}

// ✓ GOOD - From secure configuration
string apiKey = ConfigurationManager.AppSettings["ApiKey"];
using (HttpClient client = new HttpClient())
{
    client.DefaultRequestHeaders.Add("Authorization", $"Bearer {apiKey}");
}

// ✓ GOOD - From environment variables
string databaseConnection = Environment.GetEnvironmentVariable("DB_CONNECTION", 
    EnvironmentVariableTarget.User);

// ✓ GOOD - From secure user configuration (encrypted)
Configuration config = ConfigurationManager.OpenExeConfiguration(
    ConfigurationUserLevel.PerUserRoamingAndLocal);
string apiKey = config.AppSettings.Settings["ApiKey"]?.Value;

// ✓ GOOD - From Azure Key Vault (.NET)
using (var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential()))
{
    KeyVaultSecret secret = client.GetSecret("syncfusion-license-key");
    string licenseKey = secret.Value;
}
```

**Validation Rules:**
- [ ] No hardcoded API keys in source code
- [ ] No hardcoded database connection strings
- [ ] Secrets stored in secure configuration (app.config, environment, Key Vault)
- [ ] Configuration files with secrets in .gitignore
- [ ] Database credentials from secure stores, not variables
- [ ] Syncfusion license keys from secure configuration, not hardcoded
- [ ] Connection strings use Windows authentication where possible

---

### 2.3 Dependency Security

**Requirement:** Use trustworthy, well-maintained NuGet packages

**What to Check:**

```xml
<!-- .csproj file -->
<ItemGroup>
  <PackageReference Include="Syncfusion.Core.WinForms" Version="33.2.3" />
  <PackageReference Include="Syncfusion.SfInput.WinForms" Version="33.2.3" />
</ItemGroup>
```

**Validation Rules:**
- [ ] All packages from official NuGet.org registry
- [ ] No typosquatted package names (verify publisher)
- [ ] Syncfusion packages only from Syncfusion organization
- [ ] Run `dotnet list package --outdated` regularly
- [ ] No deprecated packages
- [ ] Check package advisories: https://github.com/advisories
- [ ] Verify package publisher and download count
- [ ] Use only signed packages
- [ ] Keep dependencies up-to-date with security patches

---

## Performance Standards

### 3.1 Rendering Optimization

**Requirement:** Prevent unnecessary UI redraws and improve responsiveness

**What to Check:**

```csharp
// ✗ BAD - Redraw on every property change
private void UpdateUI()
{
    for (int i = 0; i < 1000; i++)
    {
        listBox.Items.Add($"Item {i}");  // 1000 redraws!
    }
}

// ✓ GOOD - Suspend layout during batch updates
private void UpdateUI()
{
    listBox.BeginUpdate();
    for (int i = 0; i < 1000; i++)
    {
        listBox.Items.Add($"Item {i}");
    }
    listBox.EndUpdate();  // Single redraw
}

// ✓ GOOD - Use async/await to prevent blocking UI
private async void LoadDataAsync()
{
    this.Enabled = false;  // Prevent interaction during load
    
    var data = await Task.Run(() => FetchDataFromDatabase());
    
    PopulateDataGrid(data);
    this.Enabled = true;
}

// ✗ BAD - Blocking UI thread
private void LoadData()
{
    var data = FetchDataFromDatabase();  // UI freezes
    PopulateDataGrid(data);
}

// ✓ GOOD - Cache computed values
private Dictionary<string, int> _categoryCache = new();

private int GetCategoryCount(string category)
{
    if (_categoryCache.ContainsKey(category))
    {
        return _categoryCache[category];
    }
    
    int count = CalculateCategoryCount(category);  // Expensive operation
    _categoryCache[category] = count;
    return count;
}
```

**Validation Rules:**
- [ ] Use `BeginUpdate()` and `EndUpdate()` for batch operations
- [ ] Long operations use async/await (no blocking UI thread)
- [ ] Expensive computations cached, not recalculated
- [ ] No infinite loops in timers or event handlers
- [ ] Timers properly stopped/disposed
- [ ] DataGrid virtual scrolling enabled for large datasets
- [ ] Double-buffering enabled for custom controls (`DoubleBuffered = true`)

---

### 3.2 Assembly Size & Resources

**Requirement:** Keep application size reasonable and load resources efficiently

**Validation Rules:**
- [ ] Main assembly < 2MB (uncompressed)
- [ ] No duplicate NuGet references
- [ ] Remove unused `using` statements
- [ ] No large images embedded as resources (use external files)
- [ ] Use resource linking for shared assets
- [ ] Image files optimized (PNG/JPEG compression)
- [ ] Consider splitting large applications into modules
- [ ] Avoid loading entire datasets into memory

---

## Code Quality Standards

### 4.1 C# Type Safety & Null Handling

**Requirement:** Full type safety, no unhandled nulls

**What to Check:**

```csharp
// ✗ BAD - Nullable reference without null check
private void ProcessData(object sender, EventArgs e)
{
    string result = textBox.Text;  // Could be null
    int length = result.Length;     // NullReferenceException!
}

// ✓ GOOD - Proper null checking
private void ProcessData(object sender, EventArgs e)
{
    string? result = textBox?.Text;  // Nullable reference
    if (result != null)
    {
        int length = result.Length;  // Safe
    }
}

// ✓ GOOD - Explicit type safety
private void ConfigureButton(Control control)
{
    if (control is Button button)  // Type check
    {
        button.Click += OnButtonClick;
    }
}

// ✓ GOOD - Generic constraints ensure type safety
public class DataProvider<T> where T : class
{
    public T? GetData(int id)
    {
        // Type-safe return
    }
}
```

**Validation Rules:**
- [ ] Nullable reference types enabled (`#nullable enable`)
- [ ] No unchecked null dereferences
- [ ] Event handlers properly typed
- [ ] Generic types constrained where needed
- [ ] Return types explicitly defined (not inferred)
- [ ] Use `ArgumentNullException` for invalid null inputs
- [ ] Null-conditional operators (`?.`) used appropriately

---

### 4.2 Code Hygiene

**Requirement:** Clean, maintainable C# code

**What to Check:**

```csharp
// ✗ BAD - Poor naming, console output, magic numbers
public class Form1 : Form
{
    private void Button_Click(object sender, EventArgs e)
    {
        int x = 42;  // Magic number, unclear purpose
        Console.WriteLine("Debug: button clicked");
        Debug.Print("x = " + x);  // Debug output left in code
    }
}

// ✓ GOOD - Clear naming, proper logging, constants
public class MainForm : Form
{
    private const int DEFAULT_TIMEOUT_MS = 5000;
    private static readonly ILogger _logger = LoggerFactory.Create(b => b.AddConsole()).CreateLogger<MainForm>();
    
    private void OnOkButtonClick(object sender, EventArgs e)
    {
        int timeoutMs = DEFAULT_TIMEOUT_MS;
        _logger.LogInformation("OK button clicked with timeout: {TimeoutMs}ms", timeoutMs);
    }
}

// ✗ BAD - Unused variables, commented code
private void LoadData()
{
    string apiUrl = "http://api.example.com/data";  // Unused variable
    // var data = FetchData(apiUrl);  // Commented code
    // ProcessData(data);
}

// ✓ GOOD - Clean, no dead code
private async Task LoadDataAsync()
{
    const string ApiUrl = "http://api.example.com/data";
    var data = await FetchDataAsync(ApiUrl);
    await ProcessDataAsync(data);
}
```

**Validation Rules:**
- [ ] No `var` for UI controls (use explicit types: `Button`, `TextBox`)
- [ ] Use `var` only when type is obvious from right side
- [ ] No `Console.WriteLine` or `Debug.Print` in production
- [ ] Use proper logging framework (`ILogger`, `log4net`)
- [ ] No unused variables or imports
- [ ] No commented-out code blocks (use version control)
- [ ] Consistent indentation (4 spaces, IDE default)
- [ ] Meaningful variable and method names
- [ ] No magic numbers (use named constants)
- [ ] Proper access modifiers (`private`, `public`)
- [ ] Use `readonly` for fields that don't change
- [ ] Use `const` for compile-time constants

---

## Validation Checklist

**WCAG 2.2 AA Accessibility Checklist—run for every WinForms control:**

```
PERCEIVABLE
  ✓ All images have AccessibleName and AccessibleDescription
  ✓ Icon buttons have AccessibleName (not empty)
  ✓ Media has captions and descriptions (if applicable)
  ✓ Color contrast ≥ 4.5:1 for text (or 3:1 for large/14pt+)
  ✓ Color contrast ≥ 3:1 for UI components
  ✓ Information not conveyed by color alone (use icons/text)
  ✓ Auto-playing audio disabled
  ✓ Focus indicators visible (never HideSelection=true for focused controls)

OPERABLE
  ✓ All functionality accessible via keyboard (Tab, Enter, Space, Arrows)
  ✓ Tab order logical (left-to-right, top-to-bottom)
  ✓ No keyboard traps (can Tab out of all controls)
  ✓ Focus indicators always visible
  ✓ Focus outline ≥ 2px equivalent, ≥ 3:1 contrast
  ✓ Focused element not obscured by other UI
  ✓ Interactive targets ≥ 24×24 screen pixels
  ✓ Drag operations have button/keyboard alternatives
  ✓ Escape key closes dialogs/popups
  ✓ Enter submits forms, Space activates buttons
  ✓ Arrow keys work for lists/dropdowns/tabs/sliders
  ✓ Context menus accessible via keyboard

UNDERSTANDABLE
  ✓ Application culture/language set appropriately
  ✓ Language changes marked in content (if multilingual)
  ✓ UI hierarchy logical (GroupBox organization, tab order)
  ✓ Group labels descriptive
  ✓ Every form field has associated label
  ✓ Required fields marked (* or "required" text)
  ✓ Error messages clear and specific
  ✓ Error messages linked via AccessibleDescription
  ✓ Form validation at appropriate times (after focus loss, on submit)
  ✓ Information not re-requested (pre-fill where possible)
  ✓ Button text describes action clearly
  ✓ Navigation consistent across forms
  ✓ Help mechanisms consistently placed
  ✓ Login not purely cognitive test (NEW in 2.2)

ROBUST
  ✓ Native WinForms controls used (Button, TextBox, CheckBox, etc.)
  ✓ Custom controls only when native doesn't fit
  ✓ Icon buttons have AccessibleName
  ✓ Error fields have appropriate state indication
  ✓ Error messages visible and accessible
  ✓ Custom controls have AccessibleRole set
  ✓ Proper AccessibleRole and AccessibleName for custom controls

SECURITY
  ✓ No dynamic code execution (eval, Reflection for untrusted input)
  ✓ User input validated before use
  ✓ SQL queries use parameterized queries (no string concatenation)
  ✓ No hardcoded API keys or connection strings
  ✓ Secrets loaded from configuration/environment
  ✓ All NuGet packages from official NuGet.org registry
  ✓ Sensitive data not logged

PERFORMANCE
  ✓ Use BeginUpdate/EndUpdate for batch control updates
  ✓ Long operations use async/await (no UI thread blocking)
  ✓ Expensive computations cached (not recalculated)
  ✓ No infinite loops in event handlers or timers
  ✓ Timers properly disposed
  ✓ DataGrid virtual scrolling enabled for large datasets
  ✓ Double-buffering enabled for custom controls
  ✓ Main assembly < 2MB uncompressed

CODE QUALITY
  ✓ Proper null handling (#nullable enable or explicit null checks)
  ✓ Explicit types for UI controls (not var for Button, TextBox, etc.)
  ✓ Event handlers properly typed
  ✓ Return types explicitly defined
  ✓ No var for UI control declarations
  ✓ No Console.WriteLine or Debug.Print in production
  ✓ Proper logging framework used (ILogger, log4net, etc.)
  ✓ No unused variables or imports
  ✓ No commented-out code blocks
  ✓ Consistent indentation (4 spaces)
  ✓ Meaningful variable and method names
  ✓ Named constants for magic numbers
  ✓ Proper access modifiers (private, public, internal)
  ✓ Use readonly for unchanging fields
  ✓ Use const for compile-time constants
```

---

## Auto-Fix Rules

**Stage 5 automatically fixes these issues:**

| Issue | Auto-Fix |
|-------|----------|
| Missing AccessibleName on icon button | Add based on icon context |
| Missing AccessibleDescription on error field | Add description text |
| Missing TabStop on interactive control | Set TabStop = true |
| Missing TabIndex on control | Add logical tab order |
| Var declarations for controls | Convert to explicit type (Button, TextBox, etc.) |
| Console.WriteLine statements | Remove or convert to logger call |
| Unused using statements | Remove from imports |
| Low color contrast detected | Suggest higher contrast colors |
| Missing focus handling | Add focus management code |
| Unlabeled form inputs | Add Label control with correct association |
| Missing button accessibility text | Add AccessibleName/AccessibleDescription |
| Control too small (< 24×24) | Suggest minimum size |
| HideSelection = true | Change to false for accessibility |
| Control-mapping.json missing or invalid | Prompt for Stage 3 re-execution |
| Weak BM25 scores (< 20) in mappings | Warn about potential incorrect control selection |

---

**End of WinForms .NET Standards Reference**  
Updated for **WCAG 2.2 AA** with focus on WinForms/.NET implementation  
Includes NEW criteria: Focus not obscured (2.4.11), Target size (2.5.8), Dragging alternatives (2.5.7), Redundant entry (3.3.7), Accessible authentication (3.3.8)
