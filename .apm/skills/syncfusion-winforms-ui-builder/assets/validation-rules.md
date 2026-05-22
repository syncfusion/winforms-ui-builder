# Validation Rules Reference

**Purpose:** Comprehensive checklist for Stage 7 validation. Used to validate components against .NET Standard and WinForms best practices.

## Binary Validation Result

Each control receives a **PASS ✓** or **FAIL ✗** result against these rules.

---

## Control Mapping Accuracy (Stage 3-5 Validation) - Blocking

**Critical: Prevents incorrect control selection from propagating to code generation.**

| Rule | Check | Pass/Fail |
|------|-------|-----------|
| **Stage 3 Mapping Verification** | All mapped controls have `validation: "✓ VERIFIED in controls.csv"` (not fallback to native WinForms) | |
| **BM25 Score Adequacy** | All control mappings have BM25 score **40+** (excellent match), or **20-40** with documented rationale | |
| **Control-Mapping.json Accuracy** | Element `type_hint` values contain keywords from controls.csv for mapped controls | |
| **User Requirement Alignment** | Mapped controls fulfill all requirements from Stage 1 intent analysis | |
| **No Unintended Fallbacks** | Common controls (Button, TextBox, DataGrid) are NOT falling back to native WinForms (except when intentional) | |
| **Mapping Consistency** | Identical element types across sections mapped to same Syncfusion control (unless rationale documented) | |

**Examples of validation failures:**
- ❌ Button element maps to TextBox (type mismatch)
- ❌ "data_grid" element with type_hint "grid" maps to fallback ListView with score 5 (weak match)
- ❌ BM25 score 15 with no documented rationale (below 20 threshold)
- ❌ Email input maps to ComboBox instead of TextBox (incorrect control class)

**Action if blocking rule fails:**
- ✗ STOP code generation
- ✓ Return to Stage 3: Update control-mapping.json type_hints
- ✓ Re-run `controls_search.cjs` script
- ✓ Verify all mappings show "✓ VERIFIED in controls.csv"
- ✓ Verify all scores 40+ or document rationale for 20-40 scores
- ✓ Only then proceed to Stage 5

---

## Accessibility (WCAG 2.1 AA, UI Automation) - Blocking

| Rule | Check | Pass/Fail |
|------|-------|-----------|
| **Control Labels** | All controls have associated `Label` or `AccessibleName` | |
| **UI Automation** | Controls have `AutomationProperties` set for accessibility | |
| **Tab Order** | `TabIndex` values properly set, `TabStop` enabled for interactive controls | |
| **Focus Indicator** | All interactive controls have visible focus state | |
| **Keyboard Navigation** | All functionality accessible via keyboard, no traps | |
| **High Contrast** | Colors meet contrast requirements for accessibility | |
| **Control Size** | Interactive controls ≥ 44x44 DIP (Device Independent Pixels) | |

---

## Security - Blocking

| Rule | Check | Pass/Fail |
|------|-------|-----------|
| **Input Validation** | All user input validated before processing/binding | |
| **SQL Injection Prevention** | Parameterized queries used for database operations | |
| **No Secrets** | No hardcoded API keys, connection strings, or credentials | |
| **Syncfusion License** | License key in configuration file or environment variable, not hardcoded | |
| **Code Injection** | No dynamic code execution from user input | |

---

## Performance - Warning (Auto-fixable)

| Rule | Check | Status |
|------|-------|--------|
| **Virtual Scrolling** | Large lists use virtual scrolling or pagination | ⚠️ Can suggest |
| **Data Binding** | Binding operations efficiently implemented, no unnecessary refreshes | ⚠️ Can auto-fix |
| **Assembly Size** | No unnecessary dependencies or bloated libraries referenced | ⚠️ Can warn |
| **UI Responsiveness** | Long-running operations run on background thread | ⚠️ Can auto-fix |

---

## DPI Awareness & Scaling - Warning (Auto-fixable)

| Rule | Check | Status |
|------|-------|--------|
| **DPI Scaling** | Controls use DIP (Device Independent Pixels) for sizing | ⚠️ Can auto-fix |
| **Font Scaling** | Font sizes responsive to system DPI settings | ⚠️ Can auto-fix |
| **Layout Flexibility** | Layouts adapt to window resizing without breaking | ⚠️ Can auto-fix |
| **High-DPI Support** | Assets support high-DPI displays (>96 DPI) | ⚠️ Can suggest |

---

## Code Quality - Warning

| Rule | Check | Status |
|------|-------|--------|
| **.NET Naming** | Follows C# naming conventions (PascalCase types, camelCase members) | ⚠️ Can auto-fix |
| **XML Documentation** | Public classes and methods have XML doc comments | ⚠️ Can add |
| **Error Handling** | Try-catch blocks with specific exception types, user-friendly messages | ⚠️ Can auto-fix |
| **SOLID Principles** | Code follows Single Responsibility, Open/Closed, etc. | ⚠️ Can suggest |

---

## Validation Logic (Stage 7)

### Step 1: Check Blocking Rules
If ANY blocking rule fails → **FAIL ✗**
- Inaccessible controls (missing labels, no UI Automation)
- SQL injection vulnerability (non-parameterized queries)
- Hardcoded secrets or connection strings
- Focus/keyboard navigation trap

**Action:** Auto-fix if possible. If not auto-fixable, ask user to override or request fixes.

### Step 2: Check Auto-Fixable Warnings
Apply auto-fixes:
- Missing accessibility names → Add AutomationProperties
- Poor keyboard navigation → Fix TabIndex and TabStop
- Missing control size → Increase to minimum 44x44 DIP
- No error handling → Add try-catch blocks
- Inefficient binding → Optimize data binding operations

### Step 3: Check Non-Auto-Fixable Warnings
Report to user:
- Missing XML documentation comments
- Could benefit from virtual scrolling for large lists
- No background thread usage for long operations

**Action:** Warn but allow proceeding.

### Step 4: Output Result

**If all blocking rules pass + warnings auto-fixed:**
```
✓ VALIDATION PASS

All standards met:
  ✓ WCAG 2.1 AA accessibility (UI Automation)
  ✓ Security checks (.NET Standard)
  ✓ Performance optimizations
  ✓ DPI awareness & scaling
  
Auto-fixes applied: 3
  - Added AccessibleName to inputs
  - Fixed TabIndex ordering
  - Added error handling in event handlers

Ready to proceed to Stage 8...
```

**If blocking rule fails (not auto-fixable):**
```
✗ VALIDATION FAIL

Critical issues:
  ✗ Controls missing labels/AccessibleName
  ✗ SQL queries not parameterized

Auto-fixes NOT available for these issues.

Options:
  [Override & Proceed] [Request Manual Fixes] [Cancel]
```

---

## Override Behavior

If user overrides failed validation:
```
⚠️  Proceeding with known accessibility/security issues:
  - 2 controls missing UI Automation labels
  - Potential SQL injection in query builder

Code will be generated but flagged as non-compliant.
User assumes responsibility for fixing before production.
```

---

All validation rules ensure generated code follows .NET Standard best practices, accessibility standards, and is production-ready.
