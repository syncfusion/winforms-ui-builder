# Stage 1: Intent Analysis

**Purpose:** Parse and validate the user's natural language request, identify control type and modifiers, resolve ambiguities for WinForms and .NET development.

**AI Should:**
- Read the user's raw query carefully
- Identify primary intent: `generate_control`, `generate_form`, or `modify_existing`
- Extract control type (e.g., "login form" → Form/LoginForm, "data grid" → SfDataGrid, "button group" → GroupBox/Buttons)
- Extract modifiers (e.g., "dark theme" → styling:dark, "with validation" → feature:validation, "async operation" → feature:async)
- Identify target namespace/directory if specified (e.g., "in the auth folder" → targetDir:Auth/)

**Ambiguity Resolution:**
If the request is unclear, ask ONE clarifying question. Examples:

| Ambiguous Input | Clarifying Question |
|---|---|
| "Build me a form" | "What kind of form? (login, registration, contact, data-entry, multi-step)" |
| "Add a control" | "What control would you like? (DataGridView, ListView, TextBox, Button, ComboBox, etc.)" |
| "Make it better" | "Which form/control and what aspect? (accessibility, validation, theming, layout)" |

**Output to User:**
One-line confirmation:
```
✓ Understood: Generating a dark-themed LoginForm with validation and async operations.
Starting project detection...
```

**Reference:** For control type catalog and WinForms best practices, see LAYOUT-DESIGN.md

**Status:** This stage requires NO user interaction for confirmation. AI decides intent based on pure reasoning and .NET/WinForms conventions.
