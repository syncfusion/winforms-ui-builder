# Stage 1: Intent Analysis

**Purpose:** Parse and validate the user's natural language request, identify WinForms control type and requirements, resolve ambiguities.

**AI Should:**
- Read the user's raw query carefully
- Identify primary intent: `generate_form`, `generate_usercontrol`, `generate_dialog`, or `modify_existing`
- Extract control type (e.g., "login form" → Form/LoginForm, "product grid" → UserControl/ProductGrid, "confirmation dialog" → Form/ConfirmDialog)
- Extract architecture requirements (e.g., "with presenter" → MVP:enabled, "code-behind only" → MVP:minimal, "with data binding" → BindingSource:enabled)
- Extract Syncfusion controls needed (e.g., "data grid" → SfDataGrid, "chart" → SfChart, "scheduler" → SfScheduler)
- Extract styling/theme (e.g., "dark theme" → Theme:Dark, "Office style" → Theme:Office2019)
- Identify target directory if specified (e.g., "in the Forms folder" → targetDir:Forms/)
- Identify required features (e.g., "with validation", "async/await", "data binding")

**Ambiguity Resolution:**
If the request is unclear, ask ONE clarifying question. Examples:

| Ambiguous Input | Clarifying Question |
|---|---|
| "Build me a form" | "What kind of form? (login, registration, contact, data entry, multi-step)" |
| "Add a control" | "What WinForms control? (Form, UserControl, custom control, or Syncfusion control like SfDataGrid, SfChart)" |
| "Make it better" | "Which control and what aspect? (accessibility, architecture, styling, performance)" |
| "Create a grid" | "Display local data or remote? Single-select or multi-select? With filtering/sorting?" |

**Output to User:**
One-line confirmation:
```
✓ Understood: Generating a dark-themed login form with "Remember Me" support.
Starting project detection...
```

**WinForms-Specific Intent Examples:**

| User Request | Intent | controls | Architecture | Theme |
|---|---|---|---|---|
| "Create a login form" | generate_form | Form + UserControl | Code-behind | Default |
| "Add a customer data grid" | generate_usercontrol | UserControl + SfDataGrid | BindingSource + Event Handlers | Office2019 |
| "Build a product details dialog" | generate_dialog | Form/Dialog + Validation | ErrorProvider | Material |
| "Make a settings panel" | generate_usercontrol | UserControl + Properties | Event-driven | Current theme |

**Reference:** For control type catalog, see stage-3-layout-analysis.md

**Status:** This stage requires NO user interaction for confirmation. AI decides intent based on pure reasoning.
