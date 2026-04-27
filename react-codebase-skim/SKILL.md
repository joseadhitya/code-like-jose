---
name: react-codebase-skim
description: Scans and summarizes an unfamiliar React repository so the engineer can quickly understand what it does, its tech stack, its size, and its top pages/routes. Use this skill whenever the user says they are exploring a new repo, onboarding to a codebase, or asks something like "skim this repo", "what does this codebase do", "give me an overview of this project", "scan the codebase", or "I'm new to this repo". Always trigger this skill when the user is trying to get oriented in an unfamiliar React or Next.js codebase, even if they don't say "skim" explicitly.
---

# React Codebase Skim

Produces a human-readable orientation summary for a React/Next.js repository, then creates or validates an `AGENTS.md` file.

---

## Step 1 — Understand the project's purpose

```bash
# Read the top-level README
cat README.md 2>/dev/null || cat readme.md 2>/dev/null || echo "NO README"

# Check package.json for name, description, and scripts
cat package.json
```

From these, determine:
- What product or service this app represents
- Whether it is a customer-facing app, internal tool, admin panel, etc.
- Any notable scripts (build, test, lint, etc.)

---

## Step 2 — Identify the tech stack

### 2.1 Core framework & major libraries

Parse `package.json` `dependencies` and `devDependencies`. Classify what you find:

**Always report if present:**
| Category | Examples to look for |
|---|---|
| Main framework | `react`, `next`, `remix`, `gatsby`, `vite` |
| State management | `redux`, `@reduxjs/toolkit`, `zustand`, `jotai`, `recoil`, `mobx` |
| Server state / data fetching | `react-query`, `@tanstack/react-query`, `swr`, `apollo-client`, `urql` |
| HTTP client | `axios`, `ky`, `got` |
| UI / component library | `antd`, `@mui/material`, `@chakra-ui`, `shadcn`, `mantine`, `@headlessui` |
| Styling | `tailwindcss`, `styled-components`, `@emotion`, `sass`, `less` |
| Utility | `lodash`, `ramda`, `date-fns`, `moment`, `dayjs`, `immer` |
| Forms | `react-hook-form`, `formik`, `yup`, `zod` |
| Routing | `react-router`, `react-router-dom`, `wouter` |
| i18n | `i18next`, `react-intl`, `lingui` |
| Animation | `framer-motion`, `react-spring`, `gsap` |

**Skip** small task-specific libraries (date pickers, number formatters, carousels, etc.).

### 2.2 Code quality tooling

```bash
# Check for ESLint
ls .eslint* eslint.config.* 2>/dev/null

# Check for Prettier
ls .prettier* prettier.config.* 2>/dev/null

# Check for Husky / lint-staged
ls .husky/ 2>/dev/null
cat package.json | grep -E '"husky"|"lint-staged"'
```

Report: which tools are present and what they enforce (e.g. "ESLint with Airbnb ruleset", "Prettier enforced on commit via Husky").

### 2.3 Unit testing

```bash
# Check for test frameworks
cat package.json | grep -E '"jest"|"vitest"|"@testing-library"|"cypress"|"playwright"'

# Check if tests are co-located or in a separate folder
find . -name "*.test.*" -o -name "*.spec.*" | head -20 2>/dev/null
find . -name "__tests__" -type d | head -10 2>/dev/null
```

Determine:
- Which test framework is used (Jest, Vitest, Cypress, Playwright, etc.)
- Whether tests exist alongside components or in a separate directory
- Whether tests appear mandatory — look for CI config or lint rules enforcing test coverage:

```bash
ls .github/workflows/ 2>/dev/null
cat .github/workflows/*.yml 2>/dev/null | grep -i "test\|coverage" | head -20
```

---

## Step 3 — Measure the size of the repository

```bash
# Count total source files
find src -type f \( -name "*.tsx" -o -name "*.ts" -o -name "*.jsx" -o -name "*.js" \) | wc -l

# Count component files
find src -type f \( -name "*.tsx" -o -name "*.jsx" \) | wc -l
```

Report total source files and component files as a rough size indicator (small / medium / large).

---

## Step 4 — Map the routes and pages

### 4.1 Find all routes

The strategy depends on the framework:

**Next.js (App Router)**
```bash
find app -type f -name "page.tsx" -o -name "page.jsx" | sort
```

**Next.js (Pages Router)**
```bash
find pages -type f \( -name "*.tsx" -o -name "*.jsx" \) | grep -v "_app\|_document\|api/" | sort
```

**React Router (look for route definitions)**
```bash
grep -r "path=" src --include="*.tsx" --include="*.jsx" --include="*.ts" --include="*.js" -l | head -10
# Then read the most likely route config files:
find src -name "*route*" -o -name "*Route*" -o -name "*router*" -o -name "*Router*" | head -10
```

Count the total number of routes/pages found.

### 4.2 Identify the top routes (max 20)

Follow this priority order to build the list:

1. **Home page is always #1.** Find it by looking for the root path (`/`, `index`, `app/page.tsx`, `pages/index.tsx`). Read that file to understand what it renders.

2. **Follow links from the home page.** Read the home page component file and identify `<Link>`, `<a>`, `useNavigate`, `router.push`, or `navigate()` calls that lead to other pages. Those destinations are candidates for #2 onwards.

3. **Exclude auth pages.** Do not include login, register, forgot-password, reset-password, set-password, verify-email, or any other authentication/onboarding flow pages in the top routes list.

4. **Stop at 20.** If there are fewer than 20 meaningful routes after exclusions, list only what exists. Do not pad the list with low-importance routes.

For each top route, note:
- The URL path
- A one-line description of what the page does (read the file briefly if needed)

---

## Step 5 — Write the human-readable summary

Present the findings in plain English with the following structure. Be concise — this is an orientation doc, not a deep dive.

---

### 📦 What is this project?
_One to two sentences describing the product and who uses it._

### 🛠 Tech Stack
_List the framework, major libraries, and tooling. Group them naturally (state, UI, styling, etc.). Keep it scannable._

**Code quality:** _ESLint / Prettier / Husky — what's enforced_
**Testing:** _Framework + whether tests are mandatory_

### 📐 Codebase Size
_X source files, Y components — overall scale (small / medium / large)_

### 🗺 Top Routes & Pages

| # | Path | Description |
|---|---|---|
| 1 | `/` | Home — … |
| 2 | `/dashboard` | … |
| … | … | … |

_Total routes in repo: Z_

---

## Step 6 — Handle AGENTS.md

### 6.1 Check if AGENTS.md exists

```bash
find . -maxdepth 3 -name "AGENTS.md" | head -5
```

### 6.2 If NO AGENTS.md exists → create one

Write `AGENTS.md` to the repo root with the following structure:

```markdown
# AGENTS.md

## Project Overview
<one paragraph from your research>

## Tech Stack
- **Framework:** <e.g. Next.js 14 (App Router)>
- **State:** <e.g. Redux Toolkit>
- **Data fetching:** <e.g. React Query>
- **UI library:** <e.g. Ant Design>
- **Styling:** <e.g. Tailwind CSS>
- **HTTP:** <e.g. Axios>
- **Utilities:** <e.g. Lodash, Day.js>

## Code Quality
- **Linting:** <ESLint config name / ruleset>
- **Formatting:** <Prettier — yes/no, config location>
- **Git hooks:** <Husky + lint-staged — yes/no>

## Testing
- **Framework:** <Jest + React Testing Library / Vitest / etc.>
- **Location:** <co-located with components / src/__tests__/>
- **Mandatory:** <yes / no / CI-enforced>

## Coding Guidelines
<Any guidelines discovered from README, existing AGENTS.md hints, or eslint rules. If none found, write "No additional guidelines found.">

## Top Routes
1. `/` — <description>
2. `/<route>` — <description>
...
```

After writing, tell the user: "I created `AGENTS.md` at the project root based on my scan."

### 6.3 If AGENTS.md already EXISTS → compare and flag differences

Read the existing file:
```bash
cat AGENTS.md
```

Compare each section against what you discovered in Steps 1–4. Flag any discrepancy like this:

> ⚠️ **Discrepancy found in AGENTS.md**
> - **Section:** Tech Stack
> - **AGENTS.md says:** Redux
> - **I found:** Zustand (no Redux in package.json)
>
> What would you like to do?
> - [ ] Update AGENTS.md with my findings
> - [ ] Keep the existing AGENTS.md as-is
> - [ ] Review the differences together

Do not modify the existing AGENTS.md until the user explicitly confirms what to do.

---

## Edge cases

| Situation | Handling |
|---|---|
| Monorepo (multiple packages) | Scan the most likely frontend package; mention others exist |
| No `src/` directory | Look for `app/`, `pages/`, `components/` at the root level |
| No README | Infer purpose from package name, page titles, and route names |
| Routes defined dynamically (e.g. in a JSON config) | Note this and read the config file to list routes |
| Test coverage CI step found | Mark testing as "mandatory (CI-enforced)" |
| 100+ routes | Still limit the top routes list to 20; note the full count |
