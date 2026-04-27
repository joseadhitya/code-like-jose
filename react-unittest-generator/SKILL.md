---
name: react-unittest-generator
description: Generates a unit test file for a React component or a JavaScript/TypeScript helper function. Use this skill whenever a frontend engineer asks to write, generate, create, or add unit tests or test cases — whether for a component, a helper, a utility function, or a hook. Trigger on phrases like "write tests for", "add unit tests to", "generate test cases for", "cover this component with tests", or "test this helper". Always use this skill before writing any test code so that the plan-first, confirm-then-write workflow is respected.
---

# React Unit Test Generator

Generates a unit test file by first reading the target code, planning test cases with the user for confirmation, then writing and running the tests autonomously.

---

## Step 0 — Pre-flight: read AGENTS.md

```bash
find . -maxdepth 3 -name "AGENTS.md" | head -3
cat AGENTS.md 2>/dev/null || echo "NOT FOUND"
```

Extract and note the following (if present):
- **Test framework** — Jest, Vitest, etc.
- **Testing libraries** — React Testing Library, Enzyme, etc.
- **Test file convention** — co-located (`Component.test.tsx`) or in a separate directory (`__tests__/`)
- **Mock helper directory** — where shared mock helpers/fixtures live
- **Existing mock helpers** — any documented helpers for API calls, context, global state, etc.
- **Mandatory rules** — any guidelines that must be followed when writing tests

If AGENTS.md is not found, proceed with best-effort detection in Step 2, and confirm conventions with the user where ambiguous.

---

## Step 1 — Gather inputs (if not already provided)

**If testing a component:**

| Input | Description | Example |
|---|---|---|
| **Location** | Directory or file path of the component | `src/components/CustomDropdown` |
| **Type** | `global` (reusable) or `page` (single-use, page-specific) | `global` |

**If testing a helper function:**

| Input | Description | Example |
|---|---|---|
| **Location** | File path of the helper | `src/helpers/dateHelpers` |
| **Function name** | `All` to test every exported function, or a specific name | `formatDate` or `All` |

If any input is missing, ask before proceeding.

---

## Step 2 — Detect test framework (if not in AGENTS.md)

```bash
cat package.json | grep -E '"jest"|"vitest"|"@testing-library|"enzyme"'
ls jest.config.* vitest.config.* 2>/dev/null
```

Use whatever framework is found. If none is detectable, ask the user before assuming Jest.

---

## Step 3 — Read and understand the target

```bash
cat <target-file-path>
```

For components, also read closely related files that affect behavior (one level deep):
- Custom hooks it calls
- Context providers it consumes
- Key child components

Understand and note:
- **What it renders / returns** — the core output
- **What props / arguments it accepts** — required vs. optional, their types
- **What conditional branches exist** — loading, error, empty, disabled, active states
- **What side effects exist** — API calls, state mutations, context reads/writes
- **What user interactions it supports** — clicks, inputs, form submits, keyboard events

> ⚠️ **Do not modify component logic.** You may add `data-testid` attributes to elements that need selecting in tests. For any other change, describe it and ask the user for approval first.

---

## Step 4 — Write the test plan

Present a plan **before writing any code**. Format it as a table:

```
| # | Test case | Expected result |
|---|---|---|
| 1 | Renders without crashing with required props | Component mounts, no console errors |
| 2 | Displays the label text from `label` prop | `getByText('My Label')` is in the document |
| 3 | Calls `onChange` when an option is selected | `onChange` mock is called once with the selected value |
| 4 | Shows disabled styling when `isDisabled` is true | Trigger has `disabled` attribute or `aria-disabled="true"` |
| 5 | Displays loading spinner when `isLoading` is true | Spinner visible; options list not rendered |
```

Cover these areas as applicable:

**For components:**
- Rendering — mounts without errors, key elements present
- Props — required and optional props produce correct output
- States — loading, error, empty, disabled, active
- User interactions — click, input, keyboard, submit
- Side effects — API calls fired, callbacks called with correct args
- Edge cases — null/undefined props, empty arrays, long strings, boundary values

**For helper functions:**
- Happy path — correct inputs → correct output
- Edge cases — empty, null, zero, boundary values
- Error cases — invalid input, expected throws
After the table, say:

> "Here's my proposed test plan. Let me know if you'd like to add, remove, or adjust any test cases — I'll wait for your confirmation before writing any code."

**Stop here and wait for user confirmation.**

---

## Step 5 — Identify what needs to be mocked

Before writing tests, scan the target for everything that needs mocking:

| What | How to detect |
|---|---|
| API calls | `axios`, `fetch`, imported API service files |
| React Context | `useContext`, `Context.Consumer` |
| Global state | `useSelector`/`useDispatch`, Zustand stores, Jotai atoms |
| Custom hooks | Hooks doing async work or reading external state |
| Browser APIs | `localStorage`, `sessionStorage`, `window.location`, `navigator` |
| Router | `useNavigate`, `useParams`, `useLocation` |
| i18n | `useTranslation`, `intl.formatMessage` |
| Timers | `setTimeout`, `setInterval`, `Date.now` |

For each mock identified:

1. **Check AGENTS.md first** — if an existing mock helper covers this, use it.
2. **If not documented in AGENTS.md**, flag it to the user:
   > ⚠️ I need to mock `<thing>` (e.g. `axios` / `AuthContext`). I didn't find an existing helper for this in AGENTS.md.
   > I can create a reusable mock helper at `<mock-helper-directory>/<mockName>.ts`. Should I go ahead?
3. **If user agrees**, create the mock helper in the directory from AGENTS.md (ask if not specified), then update AGENTS.md to document it under a "Mock Helpers" section with: name, path, and what it mocks.

---

## Step 6 — Write the test file

Use the confirmed plan from Step 4 and the mocks from Step 5.

### File location & naming
Follow the convention from AGENTS.md. Default if not specified:
- Co-located: `<ComponentName>.test.tsx` (or `.test.jsx` / `.test.ts` / `.test.js`)
- Separate: `__tests__/<ComponentName>.test.tsx`

If a test file already exists at that path, **read it first** — do not overwrite. Append new test cases or ask the user how to proceed.

### Test structure

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import <ComponentName> from './<ComponentName>';
// import mock helpers as needed
 
describe('<ComponentName>', () => {
  const defaultProps = {
    // required props with sensible test values
  };
 
  describe('rendering', () => {
    it('renders without crashing', () => { ... });
  });
 
  describe('props', () => {
    it('...', () => { ... });
  });
 
  describe('interactions', () => {
    it('...', () => { ... });
  });
 
  describe('edge cases', () => {
    it('...', () => { ... });
  });
});
```

**Writing guidelines:**
- Prefer `userEvent` over `fireEvent` for user interactions
- Use `waitFor` / `findBy*` for async assertions
- Use `data-testid` only when no semantic or accessible selector works
- One focused behavior per test (multiple `expect` calls are fine if they all test the same behavior)
- Mock only what is necessary — keep tests as close to real behavior as possible

---

## Step 7 — Run tests and fix errors (max 3 cycles)

Run the test file immediately after writing it:

```bash
# Jest
npx jest <path-to-test-file> --no-coverage 2>&1

# Vitest
npx vitest run <path-to-test-file> 2>&1
```

### Fix cycle

Repeat until all tests pass or 3 cycles are exhausted:

```
Cycle N:
1. Read the full error output carefully
2. Identify root cause: wrong selector, missing mock, async not awaited, wrong assertion, import issue, etc.
3. Apply a targeted fix
4. Re-run the tests
```

**After 3 failed cycles**, stop and report:

> ⚠️ After 3 fix attempts, the following tests are still failing:
>
> | Test | Error |
> |---|---|
> | `<test name>` | `<concise error summary>` |
>
> Here's what I tried: `<brief explanation of each attempt>`
>
> I recommend we investigate this together. Would you like to dig into a specific failure?

---

## Step 8 — Deliver

Once tests pass (or after hitting the cycle limit), summarize:

1. **Test file path** — where the file lives
2. **Results** — X passed, Y failed (if any remaining)
3. **Mock helpers created** — new files added and their purpose
4. **AGENTS.md updates** — confirm if the file was updated with new mock info
5. **data-testid additions** — list any attributes added to the source component

---

## Edge cases

| Situation | Handling |
|---|---|
| Component has no props | Still test rendering and any internal interactions or state |
| Helper has 10+ functions and user said "All" | Plan covers all; group trivial pure functions to keep the table readable |
| Test framework not detectable | Ask the user before writing a single line |
| Mock helper directory not in AGENTS.md | Ask the user where to put shared mocks before creating anything |
| Existing test file already present | Read it first; never overwrite silently — append or ask |
| `data-testid` already on element | Reuse it, never add a duplicate |
| CI enforces a coverage threshold | Note the threshold; flag if new tests are unlikely to meet it |
| No AGENTS.md exists | Proceed with detected conventions; suggest running `react-codebase-skim` to generate one |
