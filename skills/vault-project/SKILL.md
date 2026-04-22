---
name: vault-project
description: >
  Crea y gestiona un sistema de notas markdown interconectadas (AI Vault Protocol)
  para gestión de proyectos y memoria a largo plazo. Úsalo cuando el usuario quiera
  inicializar un vault nuevo, crear un proyecto con sus notas enlazadas (Landing,
  Brief, Map, State, Decisions, References), o actualizar el estado de un proyecto
  existente. Triggear con: "crea un vault", "nuevo proyecto en el vault", "actualiza
  el state del proyecto", "/vault-project".
tools: Read, Write, Edit, Glob, Bash
user-invocable: true
---

# vault-project — AI Vault Protocol Manager

Crea y gestiona un sistema de notas markdown interconectadas para gestión de proyectos y memoria a largo plazo de agentes IA.

---

## Phase 1 — Detectar intención

Lee los argumentos del usuario y clasifica la acción:

| Argumento / intención detectada | Acción |
|----------------------------------|--------|
| `init`, "inicializar vault", "crear vault" | → Phase 2A |
| `new`, "nuevo proyecto", "crear proyecto" | → Phase 2B |
| `update`, "actualizar state", "actualizar estado" | → Phase 2C |
| Sin argumentos claros | Pregunta al usuario cuál de las 3 acciones quiere y espera respuesta |

---

## Phase 2A — Inicializar vault (`init`)

1. Pregunta la ruta del vault. Default: `~/Documents/Vault`. Si el usuario no indica otra, usa el default.
2. Expande `~` a la ruta absoluta del home del usuario usando `echo $HOME`.
3. Crea la estructura con Bash:
   ```bash
   mkdir -p "{vault}/Proyectos"
   ```
4. Crea `{vault}/START HERE - AI.md` con Write usando esta plantilla exacta:

```markdown
---
type: meta
updated: {fecha-hoy}
---

# START HERE - AI

## Goal
Use this vault as shared long-term memory for AI agents with minimum token cost.

## First Step
Read this note before exploring or writing anything else in the vault.

## Project Entry Order
For project work, read in this order:
1. Project landing note in `Proyectos/`
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

## Token Discipline
- Avoid long narrative dumps.
- Summaries should be shorter than source material.
- One note should answer one retrieval question.
- Move stale detail to references or archive notes.
```

5. Muestra al usuario la estructura creada y confirma el éxito.

---

## Phase 2B — Nuevo proyecto (`new`)

1. Pide al usuario (si no los proporcionó como args):
   - **Nombre del proyecto** (será el nombre de la carpeta y prefijo de todos los archivos)
   - **Área** (ej: "desarrollo", "investigación", "diseño")
   - **Prioridad** (alta / media / baja)
   - **Ruta del vault** (default: `~/Documents/Vault`)
   - **Kind** (ej: "software", "research", "design", "ops") — opcional, default: "software"
   - **Remote** (URL del repo u otro remote) — opcional, default: ""

2. Expande `~` con `echo $HOME` y construye la ruta base: `{vault}/Proyectos/{Nombre}/`

3. Crea la carpeta:
   ```bash
   mkdir -p "{vault}/Proyectos/{Nombre}"
   ```

4. Crea los 6 archivos del proyecto. Usa la fecha de hoy en formato `YYYY-MM-DD`.

### Archivo 1: `{Nombre}.md` (Landing)

```markdown
---
type: project
status: active
area: {area}
updated: {fecha}
root: Proyectos/{Nombre}/
kind: {kind}
remote: "{remote}"
priority: {prioridad}
---

# {Nombre}

## Entry
[[{Nombre} Brief]] · [[{Nombre} Map]] · [[{Nombre} State]]

## Quick Facts
- **Área:** {area}
- **Prioridad:** {prioridad}
- **Kind:** {kind}
- **Iniciado:** {fecha}
- **Remote:** {remote}

## Related
- [[START HERE - AI]]
- [[{Nombre} Decisions]]
- [[{Nombre} References]]
```

### Archivo 2: `{Nombre} Brief.md`

```markdown
---
type: brief
project: {Nombre}
updated: {fecha}
---

# {Nombre} Brief

← [[{Nombre}]]

## Objective
_¿Qué se quiere lograr?_

## Scope
_¿Qué incluye y qué excluye este proyecto?_

## Current State
_Estado actual al {fecha}._

## Next Actions
- [ ] 
```

### Archivo 3: `{Nombre} Map.md`

```markdown
---
type: map
project: {Nombre}
updated: {fecha}
---

# {Nombre} Map

← [[{Nombre}]]

## Important Paths
_Rutas clave del proyecto (código, docs, assets)._

## Ignore By Default
_Carpetas o archivos a ignorar al explorar._

## Notes
_Observaciones sobre la estructura._
```

### Archivo 4: `{Nombre} State.md`

```markdown
---
type: state
project: {Nombre}
updated: {fecha}
---

# {Nombre} State

← [[{Nombre}]]

## Session Status
_Estado de la última sesión de trabajo._

## Open Work
_Tareas abiertas y bloqueos actuales._

## Handoff
_Lo que debe saber la próxima sesión o agente para continuar._
```

### Archivo 5: `{Nombre} Decisions.md`

```markdown
---
type: decisions
project: {Nombre}
updated: {fecha}
---

# {Nombre} Decisions

← [[{Nombre}]]

_Registra aquí solo decisiones duraderas y todavía relevantes._
_Formato sugerido: **Decisión** — Razón._
```

### Archivo 6: `{Nombre} References.md`

```markdown
---
type: references
project: {Nombre}
updated: {fecha}
---

# {Nombre} References

← [[{Nombre}]]

## Remotes
_URLs de repositorios, APIs, servicios externos._

## Docs
_Documentación relevante (enlaces o rutas)._

## Related Notes
_Otras notas del vault que aportan contexto._
```

5. Después de crear todos los archivos, muestra al usuario una lista con los 6 archivos creados y sus rutas.

---

## Phase 2C — Actualizar proyecto (`update`)

1. Pide al usuario:
   - **Nombre del proyecto**
   - **Ruta del vault** (default: `~/Documents/Vault`)
   - **Qué sección actualizar** (State / Brief / Decisions / Map / References)
   - **Contenido nuevo** para esa sección

2. Localiza el archivo correspondiente con Read.

3. Usa Edit para reemplazar **solo la sección indicada** (el bloque entre el encabezado `## Sección` y el siguiente `##`). No reescribas el archivo completo.

4. Actualiza el campo `updated:` en el frontmatter a la fecha de hoy.

5. Confirma el cambio al usuario mostrando las líneas modificadas.

---

## Reglas generales

- Siempre usa `[[WikiLinks]]` estilo Obsidian para los enlaces entre notas (no rutas absolutas, no markdown links).
- El nombre del proyecto en los wikilinks debe coincidir exactamente con el nombre del archivo (case-sensitive).
- Si el vault no existe al crear un proyecto nuevo, créalo automáticamente antes de crear el proyecto (ejecuta el init implícitamente).
- Si ya existe un archivo con el mismo nombre, advierte al usuario antes de sobreescribir.
- Usa la fecha de hoy en formato `YYYY-MM-DD` en todos los campos `updated`.
