# CLAUDE.md

Project: research and design for a narrative game engine (Mass Effect / Persona-style) built on
Unreal, using the storylet system (`/c/dev/mass-effect-storylet-study`), UberTopo
(`/c/dev/ubertopo`) and Sculptcore (`/c/dev/sculptcore`).

## Rules

- **Docs** go in `docs/`.
- **Plans** go in `docs/plans/`.
- **Docs index:** maintain `docs/index.md`. Whenever a doc is added, moved, renamed or removed
  under `docs/`, update the index in the same change.
- **Pressure-test plans before implementing them.** Have a plan reviewed by an agent with a
  fresh context, one that has not seen the discussion that produced it. Record the review's
  findings, and any changes made to the plan because of them, in the plan file.
- **File size:** when a file exceeds **40 KB**, split it into smaller, topic-focused files, using
  a subdirectory layout that groups related files sensibly. This does not apply to
  auto-generated files.
- **Temporary comments** (debug notes, reminders, scaffolding to remove later) must be prefixed
  with `CLAUDENOTE:` so they can be found and stripped mechanically, e.g.
  `// CLAUDENOTE: remove once the materializer reuses actors`.
