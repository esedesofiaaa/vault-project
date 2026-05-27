---
name: vault-project
description: >
  Creates and manages an interconnected markdown note system (AI Vault Protocol)
  for project management and long-term memory. Use it when the user wants to
  initialize a new vault, create a project with linked notes (Landing, Brief, Map,
  State, Decisions, References, Changelog), or update an existing project.
  Trigger with: "create a vault", "new project in the vault", "update project state",
  "/vault-project".
tools: Read, Write, Edit, Glob, Bash
user-invocable: true
---

# vault-project — AI Vault Protocol Manager

Creates and manages an interconnected markdown note system for project management and long-term memory of AI agents.

---

## Phase 1 — Detect intent

Read the user's arguments and classify the action:

| Argument / detected intent | Action |
|----------------------------|--------|
| `init`, "initialize vault", "create vault" | → Phase 2A |
| `new`, "new project", "create project" | → Phase 2B |
| `update`, "update state", "update project" | → Phase 2C |
| No clear arguments | Ask the user which of the 3 actions they want and wait for response |

---

## Phase 2A — Initialize vault (`init`)

1. Ask for the vault path. Default: `~/Documents/Vault`. If the user does not specify another, use the default.
2. Expand `~` to the absolute home path using `echo $HOME`.
3. Create the structure with Bash:
   ```bash
   mkdir -p "{vault}/Projects"
   ```
4. Create `{vault}/START HERE - AI.md` with Write using this exact template:

```markdown
---
type: meta
updated: {today}
---

# START HERE - AI

## Goal
Use this vault as shared long-term memory for AI agents with minimum token cost.

## First Step
Read this note before exploring or writing anything else in the vault.

## Project Entry Order
For project work, read in this order:
1. Project landing note in `Projects/`
2. `Brief`
3. `Map`
4. `State`
5. Only then inspect code, docs, or repo paths referenced there

## Rules
- Write short, explicit notes.
- Prefer updating existing notes over creating near-duplicates.
- Keep frontmatter stable across note types.
- Put current state near the top.
- Store decisions and next actions explicitly.
- Link only notes that materially help retrieval.
- Do not use daily notes as canonical project state.

## Standard Shapes

### Project Landing
Frontmatter: `type`, `status`, `area`, `updated`, `root`, `kind`, `remote`, `priority`
Body: Entry · Quick Facts · Related

### Project Brief
Body: Objective · Scope · Current State · Next Actions

### Project Map
Body: Important Paths · Ignore By Default · Notes

### Project State
Body: Session Status · Open Work · Handoff

### Project Decisions
Only durable, still-relevant choices.

### Project References
Remotes, docs, and related notes only.

### Project Changelog
Chronological history by sprint or version. Format based on [keepachangelog.com](https://keepachangelog.com/en/1.1.0/)
Body: [Unreleased] · [{Sprint/version} — date] with subsections Added, Changed, Fixed, Decided.

## Token Discipline
- Avoid long narrative dumps.
- Summaries should be shorter than source material.
- One note should answer one retrieval question.
- Move stale detail to references or archive notes.
```

5. Show the user the created structure and confirm success.

---

## Phase 2B — New project (`new`)

1. Ask the user (if not provided as args):
   - **Project name** (will be the folder name and prefix for all files)
   - **Area** (e.g. "development", "research", "design")
   - **Priority** (high / medium / low)
   - **Vault path** (default: `~/Documents/Vault`)
   - **Kind** (e.g. "software", "research", "design", "ops") — optional, default: "software"
   - **Remote** (repo URL or other remote) — optional, default: ""

2. Expand `~` with `echo $HOME` and build the base path: `{vault}/Projects/{Name}/`

3. Create the folder:
   ```bash
   mkdir -p "{vault}/Projects/{Name}"
   ```

4. Create the 7 project files. Use today's date in `YYYY-MM-DD` format.

### File 1: `{Name}.md` (Landing)

```markdown
---
type: project
status: active
area: {area}
updated: {date}
root: Projects/{Name}/
kind: {kind}
remote: "{remote}"
priority: {priority}
---

# {Name}

## Entry
[[{Name} Brief]] · [[{Name} Map]] · [[{Name} State]] · [[{Name} Changelog]]

## Quick Facts
- **Area:** {area}
- **Priority:** {priority}
- **Kind:** {kind}
- **Started:** {date}
- **Remote:** {remote}

## Related
- [[START HERE - AI]]
- [[{Name} Decisions]]
- [[{Name} References]]
- [[{Name} Changelog]]
```

### File 2: `{Name} Brief.md`

```markdown
---
type: brief
project: {Name}
updated: {date}
---

# {Name} Brief

← [[{Name}]]

## Objective
_What is the goal of this project?_

## Scope
_What is included and excluded from this project?_

## Current State
_Current state as of {date}._

## Next Actions
- [ ] 
```

### File 3: `{Name} Map.md`

```markdown
---
type: map
project: {Name}
updated: {date}
---

# {Name} Map

← [[{Name}]]

## Important Paths
_Key paths in the project (code, docs, assets)._

## Ignore By Default
_Folders or files to skip when exploring._

## Notes
_Observations about the structure._
```

### File 4: `{Name} State.md`

```markdown
---
type: state
project: {Name}
updated: {date}
---

# {Name} State

← [[{Name}]]

## Session Status
_Status at the end of the last working session._

## Open Work
_Open tasks and current blockers._

## Handoff
_What the next session or agent needs to know to continue._
```

### File 5: `{Name} Decisions.md`

```markdown
---
type: decisions
project: {Name}
updated: {date}
---

# {Name} Decisions

← [[{Name}]]

_Record only durable, still-relevant decisions here._
_Suggested format: **Decision** — Reason._
```

### File 6: `{Name} References.md`

```markdown
---
type: references
project: {Name}
updated: {date}
---

# {Name} References

← [[{Name}]]

## Remotes
_Repository URLs, APIs, external services._

## Docs
_Relevant documentation (links or paths)._

## Related Notes
_Other vault notes that provide useful context._
```

### File 7: `{Name} Changelog.md`

```markdown
---
type: changelog
project: {Name}
updated: {date}
---

# {Name} Changelog

← [[{Name}]]

> Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)

## [Unreleased]
_Changes in progress._

## [Sprint 1] — {date}
### Added
### Changed
### Fixed
### Decided
```

> Subsections `Added`, `Changed`, `Fixed`, `Decided` are optional — include only the ones that apply per entry.

5. After creating all files, show the user a list with the 7 created files and their paths.

---

## Phase 2C — Update project (`update`)

1. Ask the user:
   - **Project name**
   - **Vault path** (default: `~/Documents/Vault`)
   - **Which file to update** (State / Brief / Decisions / Map / References / Changelog)
   - **New content** for that section

2. Locate the corresponding file with Read.

3. Use Edit to replace **only the indicated section** (the block between the `## Section` heading and the next `##`). Do not rewrite the entire file.

4. Update the `updated:` field in the frontmatter to today's date.

5. Confirm the change to the user by showing the modified lines.

---

## General rules

- Always use `[[WikiLinks]]` Obsidian-style for links between notes (no absolute paths, no standard markdown links).
- The project name in wikilinks must match the file name exactly (case-sensitive).
- If the vault does not exist when creating a new project, create it automatically before creating the project (run init implicitly).
- If a file with the same name already exists, warn the user before overwriting.
- Use today's date in `YYYY-MM-DD` format in all `updated` fields.
