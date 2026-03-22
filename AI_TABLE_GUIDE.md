# SUPABASE AI-MANAGED TABLE

Use this spec when creating any new Supabase table that will be read, written and managed by AI. It covers three things: data protection, readability, and a whole setup checklist. Table-specific column rules are always defined separately per project. A single Supabase project can contain multiple tables. Each table has its own dedicated rules table named after it: `TABLENAME_RULES`. There is no shared rules table across tables — each is fully self-contained. All this complexity needed to prevent data loss due to AI stupid behavior, and also to keep table data easily readable by humans for ease of control and validation.

---

## PART 1 — DATA PROTECTION

### THE RULES TABLE

Every data table needs its own rules table named `TABLENAME_RULES` — for example, a table called PROSPECTS gets a rules table called PROSPECTS_RULES. This is the source of truth for all AI behaviour related to that table. Without it, AI has no constraints and will act unpredictably.

**Structure:**

| Column | Type | Notes |
|---|---|---|
| ID | serial primary key | Auto-incremented |
| TARGET | text, not null | URL-style ALL CAPS path: `*` for rules applying to the whole rules table itself, `TABLENAME` for table-level rules, `TABLENAME/COLUMN_NAME` for column-level rules |
| RULE | text, not null | Normal sentence case — first letter capitalised |
| LOCKED | boolean, not null, default true | When true, row cannot be modified or deleted — only unlockable by a human via Supabase dashboard |

**RLS policies required:**
- Allow all reads
- Allow inserts
- Restrictive policy blocking DELETE where LOCKED = TRUE
- Restrictive policy blocking UPDATE where LOCKED = TRUE

**Trigger required:** BEFORE UPDATE — if OLD.LOCKED = TRUE and NEW.LOCKED = FALSE and current_user NOT IN ('postgres', 'supabase_admin') → raise exception. This ensures AI cannot unlock its own constraints via SQL; only a superuser can do it via the dashboard.

---

### GLOBAL RULES TO INSERT ON SETUP

Insert these three locked rows into every `TABLENAME_RULES` table immediately after creation. They apply database-wide (TARGET = `*`) and must be present in every rules table:

**Comment format rule:**
> All table and column comments must use the format: human-readable description in all caps first, then " | AI instructions: always check the rules table before doing anything." — this exact phrase is the fixed mandatory ending. No line breaks.

**Human approval rule:**
> Human approval required for all of the following — never do any of these without explicit approval: (1) modifying or adding any rule — show the proposed change in before → after style and wait for approval before executing, then relock the rule immediately after; (2) deleting, modifying, or adding any table or column comment — this is a policy restriction enforced by AI behaviour, not a hard database constraint; (3) deleting any table or dropping or adding any column.

**Comment sync rule:**
> Whenever a new table or column is added to the database, it must immediately receive a comment following the comment format rule. No table or column should ever exist without a comment.

---

### WHAT IS PROTECTED AT DATABASE LEVEL VS POLICY LEVEL

| Protection | Database-level (hard) | Policy-level (AI behaviour) |
|---|---|---|
| DELETE locked rules | ✓ RLS blocks it | — |
| UPDATE locked rules | ✓ RLS blocks it | — |
| Unlock rows via SQL | ✓ Trigger blocks it | — |
| Modifying table/column comments | — | ✓ Human approval rule |
| Dropping tables or columns | — | ✓ Human approval rule |
| Adding columns without comments | — | ✓ Comment sync rule |

---

## PART 2 — READABILITY

### COMMENT FORMAT

Every table and column comment follows this exact format:

```
HUMAN DESCRIPTION IN ALL CAPS. | AI instructions: always check the [TABLENAME]_RULES table before doing anything.
```

- Human part in ALL CAPS — easy to scan
- ` | AI instructions: ` as separator — normal text, clear boundary
- AI part always points to the specific rules table for this data table
- No line breaks
- AI instruction is always identical in structure — only the rules table name changes per table

---

### TARGET SYNTAX

URL-style ALL CAPS paths make it immediately clear what each rule applies to:

| Scope | TARGET | Example |
|---|---|---|
| Whole rules table | `*` | Global policies applying to all |
| Whole data table | `TABLENAME` | `PROSPECTS` |
| Specific column | `TABLENAME/COLUMN_NAME` | `PROSPECTS/STATUS` |

---

### RULE TEXT FORMAT

- Normal sentence case — first letter capitalised, rest as normal
- Column-specific rules start with "When displaying..." or "When writing..." so context is clear without reading the TARGET
- Multiple points use numbered inline format: `(1) First point; (2) Second point; (3) Third point`
- No line breaks inside rule text — everything on one line
- Rules with TARGET = `*` do not need a "When..." prefix

---

### WRITING RULES

Column-level rules (TARGET = `TABLENAME/COLUMN_NAME`) tell AI how to write and format values. Be specific about:

- Format (plain text, Markdown, structured syntax)
- Allowed values for enum-style columns
- Whether numbers require a time reference
- Length or density constraints
- What to include and what to leave out

---

### THE BEFORE → AFTER APPROVAL PATTERN

Before changing any rule, AI must show the complete rule text — not just the changed fragment:

```
BEFORE:
[full current rule text]

AFTER:
[full proposed rule text]
```

AI executes nothing until you explicitly approve. After the change is made, AI must immediately relock the rule (LOCKED = TRUE).

---

## PART 3 — CHECKLIST

Repeat the steps below for each new table added to the project.

- [ ] Create `TABLENAME_RULES` table with ID, TARGET, RULE, LOCKED columns
- [ ] Enable RLS on `TABLENAME_RULES` — block DELETE and UPDATE on LOCKED=TRUE rows
- [ ] Add unlock-protection trigger on `TABLENAME_RULES`
- [ ] Create the data table itself with UPDATED_AT and auto-update trigger
- [ ] Add comments to every table and column following the format standard
- [ ] Insert the 3 global rules into `TABLENAME_RULES` (comment format, human approval, comment sync)
- [ ] Insert table-level display rule for the data table
- [ ] Insert column-level rules for each column
- [ ] Lock all rules (LOCKED = TRUE)
