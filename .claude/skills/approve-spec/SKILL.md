---
name: approve-spec
description: Moves a spec in docs/01-requirements/01-spec/ from draft to approved after checking the readiness gates, then clears the draft markers on its backlog rows and records the decision. Use when the user says a spec is ready, asks to approve or sign off on a spec, asks whether a spec can be approved, or invokes /approve-spec. Approval itself is always the project owner's explicit decision — this skill checks and records, it never decides.
---

# Spec approval

Approval turns a spec from a proposal into a commitment. After it, changing the meaning of a requirement costs a whole new document — so the check before it has to be real.

**You never approve a spec yourself.** You run the gates, present what approving would commit the project to, and wait for the project owner to say yes in their own words. Silence, a thumbs-up on something else, or "looks good" about the gate report are not approval.

## Step 1 — Identify the target and check its current status

Find the spec the user means. If ambiguous, list the draft specs and ask which one.

Read its `สถานะ` frontmatter:

- `draft` — proceed
- `approved` — already approved. Say so, report who approved it and when, and stop
- `superseded` — refuse. A superseded spec cannot be approved; the replacement is what matters

## Step 2 — Get today's date

```
Get-Date -Format 'yyyyMMdd HH:mm'
```

## Step 3 — Run the gates

Launch both subagents in the same message so they run concurrently, with `run_in_background: false`:

- `spec-readiness-reviewer` with the target spec — gates 1 to 5
- `backlog-auditor` — the backlog must be in sync before any spec is frozen against it

## Step 4 — If any gate fails, stop

Report every blocker with where to fix it. Do not approve, and do not offer to approve anyway.

Fix what is safely fixable first and re-run the gates: missing backlog coverage can be added through `requirement-to-backlog`, and backlog drift through `audit-backlog`. Anything that needs the spec itself to change — an unanswered question, an untraceable assumption, a broken dependency — goes back to the user as a decision, not an edit you make here.

## Step 5 — Present the decision to the owner

Show, in Thai and in this order:

1. Gate results as a table
2. `SUMMARY_FOR_APPROVER` — what approving commits the project to
3. `CONCERNS` — risks and accepted limitations the owner is signing off on, stated plainly
4. What changes the moment it is approved: the spec's wording can still be corrected, but **every change to an FR, BR, scope, or acceptance criterion from then on requires a new spec that supersedes this one**

Then ask for an explicit decision. Offer at least 3 options — approve now, approve after a named change, or hold with the reason recorded.

## Step 6 — Apply the approval

Only after an explicit yes.

**In the spec's frontmatter:**

```yaml
สถานะ: approved
อนุมัติโดย: <ชื่อผู้อนุมัติ>
วันที่อนุมัติ: YYYY-MM-DD
```

**In `docs/01-requirements/01-spec/index.md`:** update the row's status to `approved`.

**In `docs/01-requirements/backlog.md`:** every row citing this spec loses its `(draft)` marker in the `Spec` column, and gets today's date in `อัปเดตล่าสุด`. Clearing the marker is allowed on any row regardless of its status — the marker describes the spec's state, not the row's scope, so it is not a rewrite of work history.

## Step 7 — Write the log

Append to `docs/05-log/{YYYYMMDD}-log.md`:

```markdown
## {HH:mm} — อนุมัติ SPEC-{NNN} {หัวข้อ}

**ผู้อนุมัติ**: {ชื่อ}
**ผลการตรวจ gate**: {สรุปผล 5 gate + ผล audit}
**สิ่งที่ผูกมัดหลังอนุมัติ**: {สรุปจาก SUMMARY_FOR_APPROVER}
**ความเสี่ยงที่ผู้อนุมัติรับไว้**: {จาก CONCERNS}
**ผลต่อ backlog**: ล้างเครื่องหมาย (draft) จาก {n} รายการ
**ไฟล์ที่เปลี่ยน**:
- {path}
```

A refused or postponed approval also gets a log entry, with the reason. Knowing why a spec was *not* approved is worth as much as knowing why it was.

## Step 8 — Report back

State what was approved, by whom, how many backlog rows cleared, and remind the user that substantive changes now require a superseding spec. Then ask whether to commit and push.

## Rules this skill enforces

| Rule | Meaning |
| --- | --- |
| ผู้อนุมัติคือเจ้าของโปรเจกต์ | Only the project owner approves. Never approve on their behalf, and never treat a gate PASS as approval |
| แก้ถ้อยคำได้ แก้เนื้อหาต้อง supersede | After approval, wording, formatting, typos, and links may be corrected in place. Any change to an FR, BR, scope boundary, or acceptance criterion requires a new spec with `supersedes:` pointing at this one |
| backlog เริ่มทำจาก draft ได้ | Work may start against a draft spec, but its backlog rows must carry `(draft)` in the `Spec` column so whoever picks them up knows the ground may still move |
| ไม่มีคำถามค้าง + FR ครบ + ผ่าน audit | All three gates must pass before approval. They are not advisory |

## Guardrails

- Never edit an approved spec's requirements. If asked to, explain that it needs a superseding spec and offer to start one through `requirement-to-backlog`.
- Never set `สถานะ: approved` without an explicit approval from the user in this conversation.
- Never delete a backlog row or change its scope during approval. Only the `(draft)` marker and the date change.
