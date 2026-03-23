# SUPABASE AI-MANAGED TABLE GUIDE

A spec for creating Supabase tables managed by AI. Follow this when setting up any new table. All complexity here exists for two reasons: prevent data loss from AI errors, and keep data readable enough for humans to validate at a glance.

---

## SECTION 1 — STRUCTURE & PROTECTION

Everything that makes the table safe. Database-level protections are hard constraints enforced by Postgres. Policy-level protections are AI behavioural rules — they rely on AI following the rules table, not on the database blocking anything.

| Protection | Database-level (hard) | Policy-level (AI behaviour) |
| --- | --- | --- |
| DELETE locked rules | ✓ RLS blocks it | — |
| UPDATE locked rules | ✓ RLS blocks it | — |
| Unlock rows via SQL | ✓ Trigger blocks it | — |
| Modifying table/column comments | — | ✓ Human approval rule |
| Dropping tables or columns | — | ✓ Human approval rule |
| Adding columns without comments | — | ✓ Comment sync rule |

- [ ] Create `TABLENAME_RULES` with columns: `ID` (serial primary key), `TARGET` (text, not null), `RULE` (text, not null), `LOCKED` (boolean, not null, default true)
- [ ] Enable RLS on `TABLENAME_RULES` — allow reads and inserts; add restrictive policies blocking DELETE and UPDATE where LOCKED = TRUE
- [ ] Add a BEFORE UPDATE trigger on `TABLENAME_RULES`: if OLD.LOCKED = TRUE and NEW.LOCKED = FALSE and current_user NOT IN ('postgres', 'supabase_admin') → raise exception *(prevents AI from unlocking its own constraints — only a superuser can do it via the dashboard)*
- [ ] Create the data table with an `UPDATED_AT timestamptz` column defaulting to `now()`
- [ ] Add a BEFORE UPDATE trigger on the data table that sets `UPDATED_AT = now()` on every update
- [ ] Insert global rule 1 (TARGET = `*`): *All table and column comments must use the format: human-readable description in all caps first, then " | AI instructions: always check the [TABLENAME]_RULES table before doing anything." — this exact phrase is the fixed mandatory ending. No line breaks.*
- [ ] Insert global rule 2 (TARGET = `*`): *Human approval required for all of the following — never do any of these without explicit approval: (1) modifying or adding any rule — show the proposed change in before → after style and wait for approval before executing, then relock the rule immediately after; (2) deleting, modifying, or adding any table or column comment; (3) deleting any table or dropping or adding any column.*
- [ ] Insert global rule 3 (TARGET = `*`): *Whenever a new table or column is added to the database, it must immediately receive a comment following the comment format rule. No table or column should ever exist without a comment.*
- [ ] Lock all rules (LOCKED = TRUE)

---

## SECTION 2 — READABILITY & DISPLAY

Everything that makes the table easy to read, validate, and work with. Covers comment format, rule syntax, and when to add display/writing rules.

- [ ] Add comments to both tables and every column in the data table using this exact format: `HUMAN DESCRIPTION IN ALL CAPS. | AI instructions: always check the [TABLENAME]_RULES table before doing anything.` — human part in ALL CAPS, AI part lowercase, no line breaks
- [ ] Insert a table-level display rule (TARGET = `TABLENAME`) — describe header format, field order, separators, and how rows should be rendered
- [ ] Insert column-level rules only where genuinely needed. Good reasons: the column stores structured or formatted content where inconsistency is likely; it has specific display behaviour (e.g. render as quote block or bullet list); it has writing constraints AI must follow (e.g. always lowercase, short label only); it stores AI-generated content where framing affects quality; or the human explicitly requests it. Simple name or free-text columns need no rule — absence of a rule means use common sense and display as plain text. When writing a rule, be specific about format, allowed values, length constraints, and what to include or leave out
- [ ] Rule text is sentence case, no line breaks, everything on one line. Column rules start with "When displaying..." or "When writing...". Multiple points use inline numbering: `(1) First; (2) Second; (3) Third`
- [ ] TARGET uses URL-style ALL CAPS paths: `*` for global, `TABLENAME` for table-level, `TABLENAME/COLUMN_NAME` for column-level
- [ ] Before changing any rule, show the full before → after text and wait for human approval. Nothing is executed until approved. After the change is made, the rule must be immediately relocked (LOCKED = TRUE)
