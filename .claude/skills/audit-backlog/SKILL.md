---
name: audit-backlog
description: Checks whether docs/01-requirements/backlog.md is still in sync with every spec in docs/01-requirements/01-spec/, and brings it up to date when it is not. Use when the user asks whether the backlog is current, asks to verify or reconcile specs against the backlog, says the backlog looks stale, or invokes /audit-backlog. Also worth running after a spec is edited outside the requirement-to-backlog pipeline.
---

# Backlog audit

Find where the backlog and the specs disagree, then fix the backlog — never the specs. A spec is the source of truth; if the audit suggests the *spec* is wrong, that is a finding to report to the user, not something to edit here.

## Step 1 — Get today's date

```
Get-Date -Format 'yyyyMMdd HH:mm'
```

## Step 2 — Audit

Launch the `backlog-auditor` subagent with `run_in_background: false`. It returns findings graded BLOCKER / WARNING / NIT, each with a proposed fix and the next free `BL-` numbers.

If it reports `UP_TO_DATE` with zero findings, skip to step 6 and say so plainly. Do not invent work to look useful.

## Step 3 — Decide what needs asking

Apply without asking:

- **ADD_ROW** for a requirement with no coverage — the spec already says what is needed; adding the row is transcription, not a decision
- **FIX_REFERENCE** on a row whose status is `ยังไม่เริ่ม` and whose text already matches the spec — the id was simply mistyped
- **NIT** convention fixes (date format, priority spelling) on rows nobody has started

Ask the user, with at least 3 options each, when:

- A finding is **BLOCKER** and the fix would cancel or replace a row
- A requirement has no coverage and it is genuinely unclear whether it needs one backlog row, several, or none because another row already absorbs it
- A cross-spec dependency is broken — resolving it may mean changing a spec, which is out of this skill's authority
- A row's status is `กำลังทำ` or `เสร็จแล้ว` and the fix would alter what it means

Batch up to 4 questions per `AskUserQuestion` call. Recommendation first, marked `(แนะนำ)`, each option stating its consequence.

## Step 4 — Update the backlog

`docs/01-requirements/backlog.md` is append-only. That constrains how fixes are applied:

| Fix | How |
| --- | --- |
| Missing coverage | Append a new row using the next free `BL-{NNN}-{MM}` |
| Row describes work nobody wants | Set its status to `ยกเลิก` and keep the row; append a replacement if one is needed |
| Wrong `Spec`/`FR` reference, row not started | Correct the reference in place, and record it in the log |
| Wrong reference, row started or finished | Do not edit — cancel it and append a corrected row referencing the original |
| Convention nit, row not started | Fix in place |
| Spec missing from `01-spec/index.md`, or its status there disagrees with its frontmatter | Fix the index table — it is a pointer, not a spec, so this stays within the skill's authority |

Set `อัปเดตล่าสุด` to today on every row you touch. Never renumber, never delete a row, never reorder the table.

## Step 5 — Write the log

Append to `docs/05-log/{YYYYMMDD}-log.md` (create with a `# Log {YYYY-MM-DD}` heading if it is the day's first entry):

```markdown
## {HH:mm} — ตรวจสอบความสอดคล้องของ backlog กับ spec

**ผลการตรวจ**: BLOCKER {n} / WARNING {n} / NIT {n}
**สิ่งที่แก้**:
- {รายการที่เพิ่ม/ยกเลิก/แก้อ้างอิง พร้อมเหตุผล}
**สิ่งที่ไม่ได้แก้และเหตุผล**:
- {finding ที่ต้องให้คนตัดสินใจ หรืออยู่นอกอำนาจของสกิลนี้}
**ข้อเสนอต่อ spec**: {ถ้าพบว่า spec เองมีปัญหา — ระบุไว้ ไม่ต้องแก้}
**ไฟล์ที่เปลี่ยน**:
- {path}
```

An audit that found nothing still gets a log entry. Knowing the backlog was verified on a given date and was clean is itself worth recording.

## Step 6 — Report back

In Thai: how many findings by severity, what was changed, what was left for the user to decide, and any problem found in a spec that this skill deliberately did not touch. Then ask whether to commit and push — do not do it unprompted.

## Guardrails

- **Never edit a spec.** Problems in a spec are reported, not fixed. Changing a spec goes through `requirement-to-backlog`, which asks the user first.
- Never delete a backlog row or renumber ids.
- Never mark a row `เสร็จแล้ว`. This skill reconciles scope, not progress.
- If the audit and the user disagree about whether something is drift, the user wins — record the decision in the log so the next audit does not re-raise it.
