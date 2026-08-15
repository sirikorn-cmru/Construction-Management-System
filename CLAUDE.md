# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

This repo currently contains **documentation only** — no application source, no build system, no package manifest, no tests. There are no build/lint/test commands to run. Everything lives under `docs/`, which is an Obsidian-style vault of `index.md` files that define the project's documentation skeleton; the actual documents have not been written yet.

Do not invent build or test commands. If application code is added later, update this section with the real ones.

## Documentation pipeline

`docs/` is not a flat folder set — it encodes a one-directional workflow, and each `index.md` states which stage feeds it and which stage it feeds. Understanding this flow matters more than the file listing:

```
01-requirements/01-spec  →  01-requirements/02-plan  →  01-requirements/03-task
                                                              ↓
                          02-design/01-prototypes  →  02-design/02-technical
                                                              ↓
                          03-testing/01-test-plan   →  03-testing/02-test-result
                                                              ↓
                                                    04-retrospectives
```

- `01-spec` is the source of truth for requirements. Plans and tasks derive from it, never the reverse.
- `02-technical` is the blueprint developers code against, and the basis for `01-test-plan`.
- `05-log` is a chronological cross-cutting record (changelog, decision log, incidents). Completed tasks record design outcomes in `02-design` and notable events in `05-log`.
- `04-retrospectives` draws its evidence from `02-test-result` and `05-log`.

When adding a document, place it at the stage that owns that kind of content and link it from the stage's `index.md`.

## Conventions

- **Language**: all documentation is written in Thai. Match it — do not switch new docs to English.
- **Links**: Obsidian wikilinks with an explicit alias, relative to the current file — `[[../02-plan/index|02-plan]]`. Every folder has an `index.md` that serves as its entry point; link to `index`, not to the folder.
- **Numbered prefixes**: folder names carry a `NN-` ordering prefix that reflects workflow order. Keep new folders consistent with it.
- **Never delete documents.** Superseded specs, cancelled plans, and obsolete decisions move to `docs/00-archived/` so the decision history survives.
