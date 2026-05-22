# Syncfusion WinForms UI Builder

**Syncfusion WinForms UI Builder** is an AI-powered agent skill that transforms your UI requirements into production-ready WinForms components. It leverages Syncfusion's extensive WinForms control library to generate accessible, responsive, and theme-consistent user interfaces.

### Key Features

- **AI Agent Orchestration**: Intelligent workflow handling design thinking, control selection, code generation, and validation
- **Syncfusion Integration**: Access to professional WinForms UI components
- **WCAG 2.1 AA Accessibility**: Built-in accessibility compliance with semantic HTML and ARIA support
- **Responsive Design**: Adaptive layouts that work across different screen resolutions
- **Design System Support**: Syncfusion theming with Office 2019, Visual Studio 2015, and custom theme support
- **C# Support**: Full type coverage for properties, events, and event handlers

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [How It Works](#how-it-works)
- [Usage](#usage)
- [Support](#support)

## Prerequisites

Before using Syncfusion WinForms UI Builder, ensure your environment meets these requirements:

| Requirement | Description |
|-------------|-------------|
| **APM** | [Install APM](https://microsoft.github.io/apm/getting-started/installation/) - Agent Package Manager for skill installation |
| **.NET Framework** | .NET Framework 4.6.2+ or .NET 8.0+ |
| **Visual Studio** | Visual Studio 2022 or later with WinForms development tools |
| **Syncfusion License** | [Commercial](https://www.syncfusion.com/sales/unlimitedlicense), [Free Community](https://www.syncfusion.com/products/communitylicense), or [Free Trial](https://www.syncfusion.com/account/manage-trials/start-trials) |

## Installation

```bash
# Install for GitHub Copilot
apm install syncfusion/winforms-ui-builder -t copilot

# Install for Claude Code
apm install syncfusion/winforms-ui-builder -t claude

# Install for Cursor
apm install syncfusion/winforms-ui-builder -t cursor

# Install for Codex
apm install syncfusion/winforms-ui-builder -t codex

```

## How It Works

The Syncfusion WinForms UI Builder uses an **8-stage AI orchestration workflow** that transforms your requirements into production-ready components.

| Stage | Name | Description |
|-------|------|-------------|
| 1 | Intent Analysis | Parse query, identify control type |
| 2 | Project Detection | Auto-detect project settings |
| 3 | Component Mapping | Select 3+ Syncfusion components |
| 4 | Theming | Lock design system decisions |
| 5 | Code Generation | Generate WinForms components |
| 6 | Dependencies | Install required packages |
| 7 | Validation | WCAG + security + performance |
| 8 | Code Insertion | Insert into project |

## Usage

Invoke the skill through your AI assistant by describing what you want to build:

**Example 1: Generate a Login Form**
```
Create a login form with email, password, and remember me checkbox
```

**Example 2: Generate a Data Table**
```
Build a customer data table with sorting and filtering
```

**Example 3: Generate a Dashboard**

Choose the `syncfusion-winforms-ui-builder` agent in the AI chat panel and invoke the skill through your AI assistant by describing what you want to build:

```
Design a WinForms dashboard with a collapsible left navigation panel containing TreeView items for Modules, Reports, and Settings; a RibbonMenu bar at the top with tabs for Home, Insert, and View; a main content region showing three SfCard tiles for system metrics (CPU Usage, Memory, Disk Space) with progress indicators; and a DataGrid showing recent activity logs with columns for Timestamp, Level, Source, and Message, plus a details panel on the right showing selected log entry information.
```

Generated code follows best practices with accessible UI patterns, consistent Syncfusion theming, strong C# typing, and built-in security measures such as input validation and avoidance of hardcoded secrets.

## Best Practices
- Maintain consistency in file structure, naming, and coding standards.
- Use advanced AI models (e.g., Claude Sonnet 4.6+) for better code quality.
- Review everything before production—replace placeholders and verify logic, security, and compatibility.

## Support

Product support is available through the following media.

- [Support ticket](https://support.syncfusion.com/support/tickets/create) - Guaranteed response in 24 hours | Unlimited tickets | Holiday support
- [Community forum](https://www.syncfusion.com/forums/windowsforms)
- [Request feature or report bug](https://www.syncfusion.com/feedback/winforms)
- [Live chat](https://www.syncfusion.com/support)