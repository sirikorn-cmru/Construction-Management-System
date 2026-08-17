---
name: build-prototype
description: Builds a clickable HTML prototype under docs/02-design/01-prototypes/ from the specs, backlog, feature list, and user journeys, styled with the tokens in DESIGN.md. Use when the user asks for a prototype, mockup, wireframe, or UI draft, wants to see what a feature would look like, or invokes /build-prototype. The scope can be narrowed to one actor, feature, spec, or journey. Always presents a plan for approval before building, and always asks whether to open a new version folder or edit the latest one.
---

# Build Prototype

A prototype exists to make disagreement visible **before** anyone writes production code. That only works if it is built from what the requirements actually say, and if the hard states — offline, no permission, cannot compute — are drawn alongside the happy path.

Two rules govern this skill and neither has an exception:

1. **Never build before the user approves the plan.**
2. **Never write into an existing version folder without asking first.**

## Paths

| What | Path |
| --- | --- |
| Prototypes | `docs/02-design/01-prototypes/{YYYYMMDD}-v{NN}-{slug}/` |
| Design system | `DESIGN.md` (repo root) |
| Sources | `docs/01-requirements/01-spec/`, `backlog.md`, `02-plan/feature-list.md`, `02-plan/user-journey.md` |
| Log | `docs/05-log/{YYYYMMDD}-log.md` |

## Step 1 — Get today's date

```
Get-Date -Format 'yyyyMMdd HH:mm'
```

## Step 2 — Check that DESIGN.md exists

If `DESIGN.md` is missing, **stop and build it first** — every screen decision depends on it, and a prototype built without one produces a design nobody agreed to.

Ask the user for the three things a design system cannot be derived from requirements:

1. **โทนสี** — offer at least 3 directions, each with ข้อดี/ข้อเสีย, and warn when a direction collides with colours the specs already reserve for meaning (status greens, ambers, reds).
2. **สไตล์** — offer at least 3, described by what they cost and buy: dense and utilitarian, spacious and calm, high-contrast field-first, and so on.
3. **ตัวอย่างภาพหรือโลโก้** — invite the user to attach a logo or a reference image. If they do, read it and derive the palette and tone from it rather than guessing. If they decline, say which colours you chose and why.

Then write `DESIGN.md` covering Brand Identity & CI, Design Tokens, UI Components & Patterns, and UX Guidelines & Rules, with every rule traced to the requirement that forces it. Get the user's confirmation on the finished file before continuing to Step 3.

## Step 3 — Plan

Launch the `prototype-planner` subagent with `run_in_background: false`, passing the scope the user gave (an actor, a feature id, a spec id, a journey, or ทั้งหมด) and today's date.

It returns the screens, their requirement mapping, coverage gaps, existing version folders, a proposed build order, and questions.

## Step 4 — Clear the questions

Put every question from the planner to the user with `AskUserQuestion`.

**Each question carries at least 3 options, and every option states its ข้อดี and ข้อเสีย concretely.** Mark the recommended one `(แนะนำ)` and say why it beats the others. Batch up to 4 per call.

If the user declines to answer, take the recommended option, record it in the version's `README.md` as an assumption, and say plainly which ones you decided on their behalf.

## Step 5 — Present the plan and wait

Show the user, in Thai, before building anything:

1. **หน้าจอที่จะสร้าง** — a table of screen id, name, surface, actor, and the requirement ids behind it
2. **สถานะที่จะวาดนอกเหนือจาก happy path** — offline, no permission, cannot compute, conflict
3. **ช่องว่างที่พบ** — requirements with no screen, screens with no requirement, journey steps with no screen
4. **ลำดับการสร้าง** and roughly how many files
5. **สิ่งที่จะไม่ทำในรอบนี้**

Then stop and wait for approval. Silence is not approval; neither is a comment about something else. If the user asks for changes, revise the plan and present it again.

## Step 6 — Decide the version folder

**Ask this every time, including the first time an existing folder is present.** Never infer it.

Offer three options with ข้อดี/ข้อเสีย, and recommend one using this rule:

| Situation | Recommend |
| --- | --- |
| A requirement, spec, feature, or journey changed since the last version | **โฟลเดอร์เวอร์ชันใหม่** — the old one is the record of what was reviewed against the old requirements |
| The last version was already shown to or approved by anyone | **โฟลเดอร์เวอร์ชันใหม่** — editing it rewrites what people agreed to |
| Fixing a defect in the prototype itself, or a copy or styling correction, and nobody has reviewed it yet | **แก้โฟลเดอร์ล่าสุด** — a new version for a typo buries the real versions |
| Unsure whether anyone reviewed it | **โฟลเดอร์เวอร์ชันใหม่** — the cheaper mistake |

State which situation applies and why before the user chooses.

The third option is always **สร้างโฟลเดอร์ใหม่โดยคัดลอกของเดิมมาเป็นฐาน** — useful when most screens stay and a few change.

New folders are named `{YYYYMMDD}-v{NN}-{slug}` where `{NN}` is a 2-digit sequence that never resets, so the folder listing reads in both date and version order.

## Step 7 — Build

One self-contained `.html` file per screen, plus `index.html` that links them in journey order.

- **All CSS inline in the file, using the token values from `DESIGN.md`** as CSS custom properties. No build step, no external stylesheet, no CDN — the vault has no build system and a prototype that needs one is a prototype nobody opens.
- Every screen carries the density profile of its surface (touch-target minimum, base font size, whether shadows or borders carry hierarchy, whether outdoor mode applies).
- Draw every state the plan listed, not only the happy path. Reachable from the screen itself — a state nobody can navigate to is a state nobody reviews.
- Use realistic Thai content and realistic numbers. Lorem ipsum hides layout problems that only appear with real Thai line-breaking.
- Interactions may be faked with plain inline JavaScript. No framework.
- Do not invent a screen, a field, or a control that no requirement backs.

## Step 8 — Write the version README

Every version folder gets `README.md`:

```markdown
# Prototype {YYYYMMDD}-v{NN} — {หัวข้อ}

**ขอบเขต**: {actor / feature / spec ที่ครอบคลุม}
**สร้างเมื่อ**: {YYYY-MM-DD}
**อ้างอิง DESIGN.md เวอร์ชัน**: {commit หรือวันที่}

## หน้าจอ

| ไฟล์ | หน้าจอ | ส่วนงาน | Journey step | Feature | Requirement |
| --- | --- | --- | --- | --- | --- |

## สถานะที่วาดไว้
## ช่องว่างที่พบ
## ข้อสมมติที่ตั้งไว้
## สิ่งที่ยังไม่ได้ทำ
```

The mapping table is the point of the folder. A prototype whose screens cannot be traced back to requirements is a drawing, not a design artefact.

## Step 9 — Link and log

Add the version to `docs/02-design/01-prototypes/index.md`, then append to `docs/05-log/{YYYYMMDD}-log.md`:

```markdown
## {HH:mm} — Prototype {YYYYMMDD}-v{NN} {หัวข้อ}

**ขอบเขต**: {...}
**เวอร์ชันใหม่หรือแก้ของเดิม**: {ทางที่เลือก + เหตุผล}
**หน้าจอ**: {n} หน้า / สถานะเพิ่มเติม {n}
**คำถามที่ถามและคำตอบ**: {...}
**ช่องว่างที่พบ**: {requirement ที่ไม่มีหน้าจอ, หน้าจอที่ไม่มี requirement}
**ไฟล์ที่เปลี่ยน**:
- {path}
```

## Step 10 — Report back

In Thai: where the prototype is, how to open it, how many screens and states, what the mapping table says, which assumptions you made without confirmation, and every gap found. Then ask whether to commit and push — do not do it unprompted.

## Guardrails

- **Never edit a spec, the backlog, the feature list, or the user journeys.** A problem found in any of them is reported: specs go through `requirement-to-backlog`, the backlog through `audit-backlog`, the derived views through `backlog-to-features`.
- Never build before the plan is approved, and never write into an existing version folder before asking.
- Never invent a requirement to justify a screen. A screen with nothing behind it comes out.
- Prototypes are never deleted. A version that is superseded stays where it is — `docs/00-archived/` is for documents, and the version history here is the record of what was reviewed when.
- If `DESIGN.md` changes after a version was built, say so in the report rather than restyling an old version in place.
