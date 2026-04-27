# code-like-jose 🤖

A personal collection of AI skills — structured instruction sets that guide AI agents to write code the way I would on my own.

These skills encode my personal coding standards, conventions, and workflows so that AI agents produce output that's consistent with how I actually work, not just how AI thinks I work.

---

## What are AI Skills?

Each skill is a `.skill` file containing a `SKILL.md` — a structured prompt that tells an AI agent exactly how to approach a specific task. Think of them as reusable, opinionated playbooks: the agent reads the skill, follows its steps, and produces output that matches my preferred patterns, tooling, and code style.

Skills are task-specific, composable, and designed to be dropped into any compatible AI agent workflow.

---

## Who is this for?

This repo is personal — built around my specific stack and the way I like to work. That said, the skills are open to read, fork, and adapt.

**My primary stack:** React · Next.js · TypeScript · JavaScript

Most skills in this repo are tailored around frontend engineering in that ecosystem, though the collection will grow over time.

---

## Using a skill

Point your AI agent at the relevant `.skill` file before starting a task. The agent will read the skill's instructions and follow them throughout the session.

Some skills depend on project-level context files (like `AGENTS.md`) being present in the repository. If a skill mentions `AGENTS.md`, run the appropriate skim/setup skill first.

---

## Repo structure

```
/
├── <skill-name>/
│   └── SKILL.md        # The skill instruction set
├── <skill-name>.skill  # Packaged skill file
└── README.md
```

New skills will be added to this repo over time as new workflows are standardized.

---

## Co-authors

These skills are co-authored by:

- **Jose Gunawarman** ([@joseadhitya](https://github.com/joseadhitya)) — requirements, domain knowledge, and real-world validation
- **Claude** (Anthropic) — skill drafting, structure, and iteration

---

## License

MIT — use freely, adapt to your own workflow.