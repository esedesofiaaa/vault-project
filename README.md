# claude-skills

A collection of custom skills for [Claude Code](https://claude.ai/code) — reusable, composable prompts that extend Claude's behavior for recurring workflows.

Each skill lives in its own folder and follows the Claude plugin format: a `SKILL.md` with metadata and instructions, plus an optional `plugin.json` for registration.

---

## Skills

### `vault-project` — AI Vault Protocol Manager

A skill that scaffolds and manages a structured Markdown note system designed as shared long-term memory for AI agents — with minimum token cost.

The core idea: instead of relying on conversation history, each project lives as a set of purpose-built notes that an agent can read in a defined order to reconstruct full context from scratch.

**Three commands:**

- `init` — Creates the vault directory and a `START HERE - AI.md` file: a meta-note that tells any agent the reading order, writing rules, and the shape of every note type before it touches anything else.

- `new` — Scaffolds a full project with 7 pre-linked notes, each answering a single retrieval question:

  | Note | Answers |
  |------|---------|
  | **Landing** | What is this project? Where do I start? |
  | **Brief** | What are we building, what's in scope, what's next? |
  | **Map** | Where are the important files? What should I ignore? |
  | **State** | What's the current sprint status? What's blocked? What's the handoff? |
  | **Decisions** | What durable architectural choices have been made and why? |
  | **References** | What external links, repos, and docs are relevant? |
  | **Changelog** | What changed each sprint? (Added / Changed / Fixed / Decided) |

  All notes are pre-linked with Obsidian-style `[[WikiLinks]]` and include structured frontmatter for consistent filtering.

- `update` — Edits a specific section of any note (State, Brief, Decisions, Map, References, or Changelog) without rewriting the file — and updates its `updated:` frontmatter field automatically.

---

## Usage

Install a skill by pointing Claude Code to this repo as a plugin source, or copy the skill folder into your local `.claude/skills/` directory.

Invoke with:
```
/vault-project init
/vault-project new
/vault-project update
```

---

## Structure

```
claude-skills/
└── vault-project/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── vault-project/
            └── SKILL.md
```

---

## Versioning

| Version | Date       | Notes                                           |
|---------|------------|-------------------------------------------------|
| 1.0.0   | 2026-04-22 | Initial release — vault-project skill           |
| 1.1.0   | 2026-04-23 | Added Changelog note (7th file) to `new` phase  |

---

*Last updated: 2026-04-23*
