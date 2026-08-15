# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

This repo currently contains **documentation only** — no application source, no build system, no package manifest, no tests. There are no build/lint/test commands to run. Everything lives under `docs/`, which is an Obsidian-style vault whose `index.md` files define the documentation skeleton; most of the actual documents have not been written yet.

Do not invent build or test commands. If application code is added later, update this section with the real ones.

## Current focus: Requirements & Product Backlog

The active phase of this project is **writing requirements documents and building the Product Backlog**. Work concentrates in `docs/01-requirements/`:

| Path | Holds |
| --- | --- |
| `01-spec/` | Spec documents, one per requirement — the source of truth |
| `backlog.md` | **Product Backlog** — the single master list, append-only |
| `02-plan/` | Roadmap, phases and milestones, feature priority, resource and time estimates |
| `03-task/` | Detailed task breakdown of individual backlog items, once work starts |

Use the **`requirement-to-backlog` skill** for anything that starts from a raw requirement. It runs the full pipeline — analyze against existing specs, clarify with the user, write the spec, update the backlog, write the log — and it owns the file-naming and ID rules below. Do not hand-roll these steps; the skill exists so the numbering and traceability stay consistent. It delegates the read-only analysis to the `requirement-analyst` subagent, which never writes and never asks the user directly.

Established conventions (set by that skill — follow them if you ever touch these files by hand):

- Spec filename: `docs/01-requirements/01-spec/{YYYYMMDD}-{NNN}-{slug}.md`. `NNN` is a **global** 3-digit sequence that never resets and is never reused; the slug is lowercase ASCII kebab-case even though the body is Thai.
- Requirement ids: `SPEC-{NNN}` → `FR-{NNN}-01` → backlog `BL-{NNN}-01`.
- Backlog priority is MoSCoW; status is `ยังไม่เริ่ม / กำลังทำ / เสร็จแล้ว / ยกเลิก`.
- Each working session appends to `docs/05-log/{YYYYMMDD}-log.md` — decisions and their reasons, not a file diff.

Guidance for this phase:

- Requirements flow **one direction**: `01-spec` → `backlog.md` → `02-plan` / `03-task`. Every backlog item must trace back to an `FR-` id in a spec. A backlog item with no matching spec is a gap in the spec — raise it rather than inventing a requirement to justify the item.
- Scope belongs in `01-spec` and nowhere else. Both what the system does *and* what it explicitly does not do.
- `02-design/`, `03-testing/`, `04-retrospectives/` are placeholders for later phases. Do not populate them ahead of the requirements — a design doc written before its spec exists will conflict with it.
- When a requirement is unclear, ask the user rather than guessing, and always offer at least 3 distinct options with their trade-offs. Anything you assume instead of confirming must be written into the spec's ข้อสมมติ section, not left implicit.

## Documentation pipeline

`docs/` is not a flat folder set — it encodes a one-directional workflow, and each `index.md` states which stage feeds it and which stage it feeds. Understanding this flow matters more than the file listing:

```
01-requirements/01-spec  →  01-requirements/backlog.md  →  02-plan  →  03-task
                                                              ↓
                          02-design/01-prototypes  →  02-design/02-technical
                                                              ↓
                          03-testing/01-test-plan   →  03-testing/02-test-result
                                                              ↓
                                                    04-retrospectives
```

- `02-technical` is the blueprint developers code against, and the basis for `01-test-plan`.
- `05-log` is a chronological cross-cutting record (changelog, decision log, incidents). Completed tasks record design outcomes in `02-design` and notable events in `05-log`.
- `04-retrospectives` draws its evidence from `02-test-result` and `05-log`.

When adding a document, place it at the stage that owns that kind of content and link it from the stage's `index.md`.

## Conventions

- **Language**: all documentation is written in Thai. Match it — do not switch new docs to English.
- **Links**: Obsidian wikilinks with an explicit alias, relative to the current file — `[[../02-plan/index|02-plan]]`. Every folder has an `index.md` that serves as its entry point; link to `index`, not to the folder.
- **Numbered prefixes**: folder names carry a `NN-` ordering prefix that reflects workflow order. Keep new folders and documents consistent with it.
- **Never delete documents.** Superseded specs, cancelled plans, and dropped backlog items move to `docs/00-archived/` so the decision history survives.
