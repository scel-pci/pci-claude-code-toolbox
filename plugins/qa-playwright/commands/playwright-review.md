---
name: playwright-review
description: Review Playwright tests against best practices and project standards
argument-hint: "<test-file-or-pattern>"
disable-model-invocation: true
---

# Playwright Test Review

Review Playwright tests against best practices and project standards.

Use the `playwright-engineer` agent to perform the review.

## First Step

If `{argument}` is empty, ask the user which test file(s) to review before proceeding.

## Review Checklist

Evaluate tests against Playwright best practices:

### Selectors

| Rating | Pattern | Why |
|--------|---------|-----|
| Preferred | `getByRole()`, `getByLabel()`, `getByText()` | User-facing, accessible |
| Good | `getByTestId()` | Stable, explicit |
| Avoid | CSS classes, tag names | Fragile, style-coupled |
| Critical | XPath, `nth-child()`, complex CSS paths | Brittle, must fix |

### Waiting

| Rating | Pattern |
|--------|---------|
| Good | Playwright auto-wait (default), `expect().toBeVisible()`, `waitForURL()` |
| Critical | `waitForTimeout()`, `setTimeout()`, hardcoded delays |

### Test Independence

- Each test should run in isolation
- No shared mutable state between tests
- No reliance on test execution order
- Proper setup/teardown with `beforeEach`/`afterEach`

### Assertions

- Every test should have meaningful assertions
- Use web-first assertions (`expect(locator).toBeVisible()`)
- Avoid `expect(await locator.isVisible()).toBe(true)` - use `expect(locator).toBeVisible()`

### Test Naming

- Descriptive names: `should {behavior} when {condition}`
- Logical grouping with `test.describe()`

### Page Objects (if applicable)

- Locators defined as properties
- Actions as methods
- No assertions inside page objects

### Flakiness Risk

Flag these patterns:
- Arbitrary timeouts or delays
- Race conditions
- Time-dependent logic
- Non-deterministic data
- Unhandled animations

## Output

Present findings by severity:

**Critical** - Causes failures or flakiness (file:line, issue, fix)

**Warning** - Maintenance risk (file:line, issue, recommendation)

**Suggestion** - Improvement opportunity (file, issue, suggestion)

**Good Practices** - What's done well

## Guidelines

- **Read-only** - Do not modify code
- **Be specific** - Reference file and line numbers
- **Provide fixes** - Don't just identify problems
- **Match project patterns** - Respect existing conventions
