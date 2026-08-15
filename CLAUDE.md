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

Two skills own this phase. Do not hand-roll their steps; they exist so the numbering and traceability stay consistent. Each delegates its read-only analysis to a subagent that never writes and never asks the user directly.

| Skill | Use it when | Subagent |
| --- | --- | --- |
| `requirement-to-backlog` | A raw requirement arrives. Analyzes it against existing specs, clarifies with the user, writes the spec, updates the backlog, writes the log. Owns the naming and id rules below. | `requirement-analyst` |
| `audit-backlog` | Checking whether the backlog still matches the specs, or after a spec was edited outside the pipeline. Reports drift and fixes the backlog — never the specs. | `backlog-auditor` |
| `approve-spec` | Moving a spec from draft to approved. Runs the readiness gates, presents what approving commits to, records the owner's decision. | `spec-readiness-reviewer` |

Established conventions (set by that skill — follow them if you ever touch these files by hand):

- Spec filename: `docs/01-requirements/01-spec/{YYYYMMDD}-{NNN}-{slug}.md`. `NNN` is a **global** 3-digit sequence that never resets and is never reused; the slug is lowercase ASCII kebab-case even though the body is Thai.
- Requirement ids: `SPEC-{NNN}` → `FR-{NNN}-01` → backlog `BL-{NNN}-01`.
- Backlog priority is MoSCoW; status is `ยังไม่เริ่ม / กำลังทำ / เสร็จแล้ว / ยกเลิก`.
- Each working session appends to `docs/05-log/{YYYYMMDD}-log.md` — decisions and their reasons, not a file diff.

### Spec lifecycle

`draft → approved → superseded`. A spec's status governs what may be done to it:

- **Only the project owner approves.** A passing readiness check is not approval; neither is agreement about something else in the conversation. Never set `สถานะ: approved` without an explicit yes.
- Approval requires all three gates: no open questions, every `FR-` covered by a backlog row, and a clean `audit-backlog` run.
- **After approval, wording may be corrected in place; meaning may not.** Any change to an `FR-`, `BR-`, scope boundary, or acceptance criterion requires a new spec carrying `supersedes:`. This is why `requirement-analyst` offers EXTEND only for specs still in draft.
- Work may start against a draft spec. Its backlog rows carry `(draft)` in the `Spec` column so whoever picks one up knows the requirement can still move; `approve-spec` clears the marker.
- Superseded specs are never deleted — they keep `สถานะ: superseded` and later move to `docs/00-archived/`.

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
