---
name: react-component-generator
description: Generates production-ready React components from a reference image, prop definitions, and component scope (reusable global vs. page-specific). Use this skill whenever a frontend engineer asks to build, scaffold, create, or generate a React component — especially when a design reference or prop list is involved. Trigger even on casual phrasing like "make me a card component", "build this UI", or "turn this mockup into code". This skill enforces project design-system rules (colors, typography, spacing), auto-detects TypeScript vs. JavaScript, and runs ESLint automatically after writing code.
---

# React Component Generator

Generates a React component that faithfully implements a visual reference while conforming to the project's design system, ESLint rules, and typing conventions.

---

## Step 0 — Pre-flight checks (always do these first, in order)

### 0.1 AGENTS.md
```bash
find . -maxdepth 2 -name "AGENTS.md" | head -5
```
If found, read it fully. Follow every instruction and coding guideline it contains for the remainder of this task. These override your defaults.

### 0.2 ESLint config
```bash
ls .eslint* eslint.config.* eslintrc.* 2>/dev/null || true
```
If any ESLint config is found, note the path — you will run a lint check after writing the component.

### 0.3 TypeScript or JavaScript?
```bash
ls tsconfig.json 2>/dev/null && echo "TS" || echo "JS"
```
- **TypeScript** → define an `interface` or `type` for props, use `.tsx` extension  
- **JavaScript** → use JSDoc `@typedef` + `@param` for type checking. **Do not** use `React.PropTypes`.

### 0.4 Color tokens
```bash
cat src/colors.js 2>/dev/null || cat src/colors.ts 2>/dev/null || echo "NOT FOUND"
```
Import and use these tokens in the component. Never hardcode color hex values that exist in the design system.

---

## Step 1 — Gather inputs (if not already provided)

You need all five of these before writing code:

| Input | Description | Example |
|---|---|---|
| **Reference image** | Screenshot or mockup of the component to build | _(attached image)_ |
| **Props list** | Name, type, and whether each prop is required or optional | `label: string`, `isDisabled?: boolean` |
| **Component scope** | `global` (reusable) or `page` (one-time, page-specific) | `global` |
| **File name** | PascalCase component name, used as the filename | `CustomDropdown` |
| **File location** | Full directory path where the file should be created | `src/components/CustomDropdown` or `src/pages/Profile/components/CustomDropdown` |
 
If any of these are missing, ask for them before proceeding. Do not infer or guess the file name or location — always confirm with the user.

---

## Step 2 — Analyze the reference image
 
Study the image carefully:
- Identify every visual sub-element (text, icons, buttons, images, containers, dividers, etc.)
- Note states if visible (default, hover, disabled, loading, error)
- Identify the layout direction (row vs. column, wrapping behavior)
- Note any interactive behaviors implied by the design

---

## Step 3 — Plan the component

Before writing code, confirm and state:
1. **Component name** — use the file name provided by the user (PascalCase), e.g. `CustomDropdown`
2. **Output file path** — combine the user-provided location + file name + correct extension:
   - TypeScript: `<location>/<FileName>.tsx`
   - JavaScript: `<location>/<FileName>.jsx`
   - Example: `src/components/CustomDropdown/CustomDropdown.tsx`
3. Sub-components or helper functions needed
4. External dependencies required (if any)

---

## Step 4 — Write the component

### Design system rules — strictly enforce all of these:

**Colors**
- Import from `src/colors.js` (or `src/colors.ts`)
- Never hardcode colors that exist in the design system

**Typography** — only use these exact combinations:
| font-size | line-height | font-weight |
|---|---|---|
| 10px | 14px | 600 |
| 10px | 14px | 800 |
| 12px | 16px | 600 |
| 12px | 16px | 800 |
| 14px | 18px | 600 |
| 14px | 18px | 800 |
 
**Spacing**
- Padding / margin → multiples of **4px** (4, 8, 12, 16, 20, 24, 28, 32…)
- Border width → multiples of **2px** starting at 1px (1, 2, 4…)
- Border-radius → multiples of **4px** OR multiples of **5%** (4px, 8px… or 5%, 10%, 15%…)
- Container padding → **16px**; container margin → **0px**
- Multi-child containers → use `display: flex`
- Element spacing → use `gap`, not margins between siblings

**Props typing**
- TypeScript: `interface Props { … }` or `type Props = { … }`
- JavaScript: JSDoc `@typedef` + annotated props. No `PropTypes`.

### Component template (JavaScript example):

```jsx
import React from 'react';
import colors from 'src/colors';
 
/**
 * @typedef {Object} MyComponentProps
 * @property {string} title - The title text
 * @property {boolean} [isActive=false] - Whether the component is active
 */
 
/**
 * MyComponent — short description of what it does.
 *
 * @param {MyComponentProps} props
 */
const MyComponent = ({ title, isActive = false }) => {
  return (
    <div style={{
      padding: '16px',
      margin: '0',
      display: 'flex',
      gap: '8px',
    }}>
      <span style={{
        fontSize: '14px',
        lineHeight: '18px',
        fontWeight: 600,
        color: colors.textPrimary,
      }}>
        {title}
      </span>
    </div>
  );
};
 
export default MyComponent;
```

---

## Step 5 — Lint and auto-fix

After writing the file, run:

```bash
npx eslint <path-to-component> --fix
```

Then check for remaining errors:

```bash
npx eslint <path-to-component>
```

If errors remain that `--fix` could not resolve, fix them manually and re-run until the output is clean. Report any warnings to the user.

---

## Step 6 — Deliver

Provide the user with:
1. **The component file** (path + full code)
2. **Usage example** — a short snippet showing how to import and render the component with realistic props
3. **Lint status** — confirm clean, or list any remaining warnings
4. **Design system notes** — call out any place where the design deviated from project rules and how you handled it

---

## Edge cases & guidance

| Situation | Handling |
|---|---|
| Color not in `src/colors.js` | Use the closest available token; flag the mismatch to the user |
| Font combo not in the allowed set | Pick the closest allowed combo; note the deviation |
| Spacing value not a valid multiple | Round to the nearest valid multiple; note it |
| Image shows a state not described in props | Add an optional prop for it with a sensible default; document it |
| `src/colors.js` not found | Proceed with CSS variables or inline placeholders; warn the user |
| AGENTS.md conflicts with this skill | AGENTS.md takes precedence |
