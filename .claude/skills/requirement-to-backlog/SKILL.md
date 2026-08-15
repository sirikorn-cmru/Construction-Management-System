---
name: requirement-to-backlog
description: Turns a raw requirement from the user into a numbered spec document under docs/01-requirements/01-spec/, then updates the product backlog and the daily log. Use when the user brings a new requirement, feature request, or change request in unstructured form — "อยากได้ระบบที่...", "ลูกค้าขอเพิ่ม...", "ช่วยเขียน requirement เรื่อง...", "สรุปเป็น backlog ให้หน่อย" — or invokes /requirement-to-backlog. Asks the user to resolve anything ambiguous before writing, always offering at least 3 options.
---

# Requirement → Spec → Backlog

Run the whole pipeline in the main loop. The `requirement-analyst` subagent does the read-heavy analysis, but **you** ask the user the questions — a subagent cannot.

## Paths

All paths are relative to the repo root. The user's shorthand maps like this:

| Shorthand | Actual path |
| --- | --- |
| `01-requirements/01-spec/` | `docs/01-requirements/01-spec/` |
| `01-requirements/backlog.md` | `docs/01-requirements/backlog.md` |
| `log/{YYYYMMDD}-log.md` | `docs/05-log/{YYYYMMDD}-log.md` |

## Step 1 — Capture the raw requirement

Take it verbatim from the user's message or the skill arguments. If the skill was invoked with nothing to work on, ask what the requirement is and stop until they answer.

Do not summarize or clean it up yet. The raw text is evidence — it gets quoted in the spec's "ที่มา" section.

## Step 2 — Get today's date

```
Get-Date -Format 'yyyyMMdd'
```

Never guess the date or reuse one from earlier in the conversation.

## Step 3 — Analyze

Launch the `requirement-analyst` subagent with `run_in_background: false`, passing the raw requirement verbatim and today's date. Wait for its report — everything downstream depends on it.

## Step 4 — Confirm the document decision

If the analyst returned **EXTEND** or **SUPERSEDE**, do not act on it silently. Ask the user with `AskUserQuestion`, putting the analyst's recommendation first and marking it `(แนะนำ)`:

- แก้เอกสารเดิม `<file>` — ไม่เพิ่มไฟล์ใหม่ แต่ประวัติของ requirement เดิมจะถูกกลืน
- สร้างเอกสารใหม่ที่ทดแทนเอกสารเดิม — เอกสารเดิมถูกตั้งเป็น `superseded` และเก็บประวัติไว้ครบ
- สร้างเอกสารใหม่แยกอิสระ — เหมาะเมื่อจริง ๆ แล้วคนละเรื่องกัน

If the decision is **NEW**, skip this step.

## Step 5 — Clear every open question

For each question in the analyst's report, ask the user with `AskUserQuestion`.

Hard rules:

- **Every question carries at least 3 options.** No yes/no questions. If you cannot think of a third distinct approach, the question is not ready to ask — merge it into another question or move it to assumptions.
- Recommendation first, labelled `(แนะนำ)`. Each option's `description` states the consequence of choosing it, not just a restatement of the label.
- Thai, matching the vault's language.
- Batch up to 4 questions per call rather than one at a time.
- Answers can raise new questions. Loop until nothing material is unresolved — but do not manufacture questions to look thorough. Two well-aimed questions beat eight.
- If the user declines to answer, pick the recommended option, write it into the spec's ข้อสมมติ section, and say plainly which assumption you made.

## Step 6 — Write the spec

Path: `docs/01-requirements/01-spec/{YYYYMMDD}-{NNN}-{slug}.md`, using the analyst's `NNN` and slug. Write in **Thai**. Wikilinks use the vault convention — `[[../../05-log/index|05-log]]`.

```markdown
---
spec-id: SPEC-{NNN}
วันที่: {YYYY-MM-DD}
สถานะ: draft
supersedes: {spec-id หรือเว้นว่าง}
---

# SPEC-{NNN} — {หัวข้อ}

## 1. ที่มาและวัตถุประสงค์
> {ข้อความดิบจาก user แบบ verbatim}

{ตีความว่าปัญหาที่แท้จริงคืออะไร และแก้แล้วได้อะไร}

## 2. ขอบเขต
### อยู่ในขอบเขต
### ไม่อยู่ในขอบเขต

## 3. ผู้เกี่ยวข้อง (Actors)

## 4. ความต้องการเชิงหน้าที่ (Functional Requirements)
| ID | ความต้องการ | ความสำคัญ |
| --- | --- | --- |
| FR-{NNN}-01 | | Must |

## 5. ความต้องการที่ไม่ใช่หน้าที่ (Non-functional Requirements)

## 6. กฎทางธุรกิจ (Business Rules)

## 7. User Stories
- ในฐานะ {actor} ฉันต้องการ {สิ่งที่ต้องการ} เพื่อ {คุณค่าที่ได้}

## 8. เกณฑ์การยอมรับ (Acceptance Criteria)

## 9. ข้อสมมติและคำถามที่ยังค้าง

## 10. เอกสารอ้างอิง
```

Every FR must be traceable to the raw requirement or to an answer the user gave. Nothing invented. `ไม่อยู่ในขอบเขต` is not optional — an unbounded spec is the main source of scope creep later.

If the decision was SUPERSEDE, also set the old spec's `สถานะ: superseded` and add a pointer to the new spec id. Do not delete it.

## Step 7 — Update the backlog

Update `docs/01-requirements/backlog.md`. Create it from this template if it does not exist:

```markdown
# Product Backlog

Backlog รวมของโปรเจกต์ ทุกรายการต้องอ้างอิงกลับไปยังเอกสาร spec ใน [[01-spec/index|01-spec]] ได้เสมอ

- **ความสำคัญ**: MoSCoW — Must / Should / Could / Won't
- **สถานะ**: ยังไม่เริ่ม / กำลังทำ / เสร็จแล้ว / ยกเลิก
- รายการที่ถูกยกเลิกไม่ต้องลบ ให้เปลี่ยนสถานะเป็น "ยกเลิก" แล้วคงบรรทัดไว้

| ID | รายการ | Spec | FR | ความสำคัญ | สถานะ | อัปเดตล่าสุด |
| --- | --- | --- | --- | --- | --- | --- |
```

Append one row per backlog candidate. Item id is `BL-{NNN}-{MM}` where `{NNN}` is the spec number and `{MM}` a 2-digit counter within that spec. `อัปเดตล่าสุด` is today's date.

Never rewrite or renumber existing rows. If a new spec supersedes an old one, set the old rows' status to `ยกเลิก` and add the new rows below — the backlog is append-only so its history stays readable.

## Step 8 — Write the log

Append to `docs/05-log/{YYYYMMDD}-log.md`, creating it with a `# Log {YYYY-MM-DD}` heading if it is the first entry of the day. Get the time with `Get-Date -Format 'HH:mm'`.

```markdown
## {HH:mm} — SPEC-{NNN} {หัวข้อ}

**Requirement ดิบ**: {สรุปสั้น ๆ}
**การตัดสินใจ**: {NEW / EXTEND / SUPERSEDE} — {เหตุผลย่อ}
**คำถามที่ถามและคำตอบ**:
- {คำถาม} → {คำตอบที่ user เลือก}
**ข้อสมมติที่ตั้งไว้**: {ถ้ามี}
**ไฟล์ที่เปลี่ยน**:
- {path}
```

The log's purpose is answering "ทำไมตอนนั้นถึงตัดสินใจแบบนี้" months later. Record the decisions and their reasons, not a file diff.

## Step 9 — Report back

Tell the user, in Thai: the spec path, how many backlog items were added, which assumptions you made without confirmation, and anything still unresolved. Then ask whether to commit and push — do not do it unprompted.

## Guardrails

- Never invent a requirement to fill a gap. Unknown things go into คำถาม or ข้อสมมติ.
- Never write into `docs/02-design/`, `docs/03-testing/`, or `docs/04-retrospectives/` — those are later phases.
- Never delete a spec or a backlog row. Superseded content moves to `docs/00-archived/`.
- If any of steps 6–8 fails, say which ones completed and which did not. A spec written without its backlog row is a silent inconsistency.
