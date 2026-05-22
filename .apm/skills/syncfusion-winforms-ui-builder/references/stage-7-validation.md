# Stage 7: Validation

**Purpose:** Validate generated code against .NET Standard and WinForms best practices. Binary pass/fail result.

**AI Should:**

1. **Validate Accessibility (WCAG 2.1 AA, UI Automation):**
   - All controls have associated `Label` or `AccessibleName` property?
   - `AutomationProperties` set for UI Automation support?
   - `TabIndex` properly configured, `TabStop` enabled for interactive controls?
   - Focus indicator visible on interactive elements?
   - Keyboard navigation supported? (no traps, Tab order logical)
   - Color contrast ≥ 4.5:1 for text and UI components?
   - Interactive controls ≥ 44x44 DIP (Device Independent Pixels)?

2. **Check Security:**
   - Input validation implemented before processing/binding?
   - SQL queries use parameterized queries (not string concatenation)?
   - No hardcoded secrets/API keys/connection strings?
   - Syncfusion license key in config or environment variable, not hardcoded?
   - No dynamic code execution from user input?

3. **Verify Performance & Responsiveness:**
   - Large lists use virtual scrolling or pagination?
   - Data binding efficiently implemented, no unnecessary refreshes?
   - Long-running operations execute on background thread (not UI thread)?
   - No blocking operations on UI thread?

4. **Check DPI Awareness & Scaling:**
   - Controls use DIP (Device Independent Pixels) for sizing?
   - Font sizes responsive to system DPI settings?
   - Layouts adapt to window resizing without breaking?
   - High-DPI support for assets (≥96 DPI)?

5. **Verify Code Quality:**
   - Follows C# naming conventions (PascalCase types, camelCase members)?
   - Public classes/methods have XML documentation comments?
   - Error handling with specific exception types and user-friendly messages?
   - Code follows SOLID principles?

**Validation Result:**

Binary: **PASS ✓** or **FAIL ✗**

**If PASS:**
```
✓ Validation Result: PASS

All standards met:
  ✓ WCAG 2.1 AA accessibility (UI Automation)
  ✓ .NET security standards
  ✓ Performance optimization
  ✓ DPI awareness & scaling
  ✓ Code quality

Auto-fixes applied: 3
  ✓ Added AccessibleName to controls
  ✓ Fixed TabIndex ordering
  ✓ Added error handling in event handlers

Ready to proceed to dependencies...
```

**If FAIL:**
```
✗ Validation Result: FAIL

Blocking issues found:
  ✗ Controls missing AccessibleName or Label (accessibility)
  ✗ SQL query not parameterized (security)

Auto-fixes applied:
  ✓ Added AccessibleName to 2 controls
  ✓ Parameterized SQL query

Warnings (auto-fixed):
  ✓ Missing error handling in Form_Load event
  ✓ Virtual scrolling recommended for DataGridView with 1000+ rows

Remaining issues: 0
Result: PASS (after fixes)
```

**User Interaction:** ⭐ **USER DECISION #2**

If result is PASS:
```
Ready to generate dependencies?
[Proceed] [Review] [Stop]
```

If result is FAIL (with blocking issues not auto-fixable):
```
Validation failed with 2 critical issues (not auto-fixable):
  - DataGridView columns need manual sort/filter implementation
  - Custom event handling required for business logic

Override and proceed anyway?
[Override & Proceed] [Request Manual Fixes] [Stop]
```

**Status:** ⭐ **USER DECISION #2** - User confirms validation result or overrides.

**Reference:** See validation-rules.md for complete .NET Standard and WinForms validation rules. For detailed standards, see winforms-dotnet-standards.md.
