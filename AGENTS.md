# Cursor-Match Solver

Your diff is scored position-by-position against Cursor (Claude Sonnet 4).

```
score = matched_changed_lines / max(your_changed_lines, cursor_changed_lines)
```

One wrong line cascades — destroys ALL subsequent matches in that file.
Under-editing hurts exactly as much as over-editing.

---

## Data from 180 real validator tasks

| Task type | Count | Avg cursor lines | King wins | Our win chance |
|-----------|-------|-----------------|-----------|----------------|
| feature   | 98    | 317             | 62%       | 38%            |
| refactor  | 37    | 266             | 42%       | 58% best       |
| bugfix    | 33    | 153             | 55%       | 45%            |
| style     | 7     | 300             | 43%       | 57%            |
| docs      | 3     | 51              | 67%       | 33%            |

Best opportunities: refactor and style tasks.
On feature tasks, king wins by writing more complete implementations.

---

## How Cursor fails (real examples from data)

1. File permission changes instead of logic (2 confirmed cases):
   - "Fix tray icon logic" — Cursor changed chmod flags, zero logic written
   - "Add RDD material" — Cursor changed file permissions, wrote nothing
   - "Fix NuGet deployment" — Cursor only changed mode 100755

2. Over-engineering (Cursor adds more than needed, king is tighter):
   - 265 vs king 204 (redundant CSS rules)
   - 285 vs king 121 (changed too many files for a targeted role-based fix)
   - 375 vs king 330 (extra caching abstractions not needed)
   - 391 vs king 131 (energy dashboard over-engineered)
   - 655 vs king 334 (team colors feature bloated with duplicate state)

3. Under-implementing (missing criteria, king covers more):
   - 202 vs king 293 (missed fallback emoji for all 16 cities)
   - 71 vs king 148 (Tailwind grid fix incomplete)
   - 94 vs king 160 (PostgreSQL fallback logic missing)
   - 87 vs king 391 (deduplication logic not written)

The pattern: Cursor loses when it adds wrong content or misses criteria.
Cursor wins when it matches scope exactly.

---

## CRITICAL RULE 1: Never touch file permissions

NEVER include chmod or mode changes in your diff.
Lines like "old mode 100755" / "new mode 100644" destroy your score.
Cursor includes these accidentally and LOSES those rounds.
You must actively filter them out.

If you accidentally modified file permissions — revert them before finishing.

---

## CRITICAL RULE 2: Implement ALL acceptance criteria

Read every single criterion. Count them. Implement all of them.

If the task has 7 criteria — your diff must address all 7 areas.
Missing one criterion = missing lines = losing the round.

Real failure examples:
- "Add disclaimer, update portal link, replace resources section" — only did 2 of 3
- "Fix chat streaming issues and update URLs" — streaming logic was missing
- "Upgrade Node.js to 22" — missed jest.yml and .nvmrc files
- "Add index-based for loop to Dart cheatsheet" — missed the StringBuffer example

Before stopping: re-read every criterion. Did your diff touch each one?

---

## CRITICAL RULE 3: Scope calibration

| Signal | Action |
|--------|--------|
| King diff much larger than cursor | You are probably missing criteria |
| Your diff much larger than king | You over-engineered |
| Sizes within 15% | Good scope |

Do not add:
- Comments or docstrings (unless already in the file section you are editing)
- Type annotations (unless the whole file uses them throughout)
- Error handling (unless the surrounding code already has it)
- Helper functions (unless the task explicitly requires them)
- Extra blank lines or import reorganization

Do add:
- Everything each criterion requires
- Changes to ALL files mentioned in criteria
- Complete implementations, not stubs

---

## RULE 4: Naming conventions — copy from codebase

| Language / context | Convention | Real examples |
|--------------------|------------|---------------|
| Python variables/functions | snake_case | num_clients, range_start, load_from_db |
| JavaScript/TypeScript functions | camelCase | spawnDropParticles, fileToJpeg1024Base64 |
| CSS classes | kebab-case | result-image-wrap, spot-{row}-{col}, hero-fade-up |
| React/Vue components | PascalCase | Footer, CreateGameDto, HomeScreen, PublicLayout |
| Constants | SCREAMING_SNAKE_CASE | DATABASE_URL, MANUAL_CHECKLIST_KEY |
| Database columns | snake_case | singer_id, musical_key, subject_id, action_date |

Algorithm for naming new functions:
1. Find the most similar existing function in the same file
2. Identify its verb+noun pattern
3. Apply same pattern, change only the semantic part

Example: file has listBitableRecords — new function = retrieveBitableRecords, NOT getRecords
Example: file has createUser — new function = updateUser, NOT editUser or modifyUser

---

## RULE 5: Use native APIs over custom implementations

Cursor always reaches for standard library or browser-native solutions.

| Custom (bad) | Native (good) |
|--------------|---------------|
| Custom gregorianToHijri() function | Intl.DateTimeFormat('ar-SA-u-ca-islamic') |
| Custom pickle-based cache class | diskcache.Cache |
| Manual EOCD ZIP scanning | Standard ZIP with EOCD lookup |
| Custom retry loop | tenacity or retry decorator |
| Manual date formatting | dayjs, date-fns, or Intl.DateTimeFormat |

---

## RULE 6: For new files — mirror nearest existing file exactly

1. Find the most similar existing file in the same directory
2. Copy its exact structure: imports, exports, class/function layout, error handling style
3. The new file must look indistinguishable from existing ones

Real pattern examples:
- New test file — copy test_*.py or *.test.ts from same directory
- New React component — copy nearest Component.tsx structure
- New DTO — copy CreateXxxDto.ts pattern exactly
- New route module — copy nearest routes/*.js structure
- New migration file — copy version number pattern from existing migrations

---

## RULE 7: Cascading failure prevention

One wrong character destroys ALL positions after it in the file.

- Edit top-to-bottom within each file
- Process files in alphabetical path order
- Never add or remove blank lines unless explicitly required
- Never reorder imports
- Match exactly: indentation (tabs vs spaces), quote style, semicolons, trailing commas
- Match the operation: delete=delete, replace=replace, insert=insert

---

## RULE 8: Task-type specific behavior

### Bugfix
Insert the fix at the exact point where data is consumed, not where it is produced.
Keep all surrounding code structure identical.
One surgical insertion or replacement per bug.

Example: "Fix Zabbix IP assignment" — insert coordinate validation right before IP generation,
not at the data loading stage.

### Refactor
Replace the old pattern everywhere it appears in all affected files.
Do not leave partial replacements.
Prefer native or standard library alternatives where they exist.
Do not add abstractions — just replace the pattern.

Example: "Replace httpx with requests" — update all method calls, imports, dependencies. Nothing else.

### Feature
Implement incrementally following existing patterns in the codebase.
Add new code inline — do not create new abstraction layers unless codebase already has them.
Complete ALL criteria before stopping — features tend to have many sub-requirements.

### Style / formatting
Match the exact formatting convention of the file.
If 2-space indent — use 2-space. If tabs — use tabs. If single quotes — use single quotes.
Change only what the criterion requires.

### Test
Use the test framework already present in the file.
Mock at module level: @patch('app.module.Class'), not at import level.
Assert both response content AND mock call verification in the same test.
Follow existing test naming: test_<endpoint>_<behavior> for pytest.

### Docs
Minimal snippets matching exact formatting.
Preserve HTML entities (&lt;, &amp;), span class names, attribute patterns verbatim.
Copy surrounding style completely.

---

## Workflow

1. Read ALL acceptance criteria — understand complete scope before writing a single line
2. Count criteria items — you need to address every one
3. Identify ALL files that need changes
4. Read each target file IN FULL — no skipping, no truncation
5. For new functions or files: find naming pattern from existing code
6. Implement everything — alphabetical file order, top-to-bottom within each file
7. Check: does every criterion have corresponding lines in your diff?
8. Check: any file permission changes? Any extra blank lines? Any comments you added?
9. Stop

## Decision test before each edit

- "Does this implement what the criterion requires?" — Yes: do it. No: skip.
- "Would Cursor write exactly this line?" — Yes: include. No: omit. Unsure: omit.
- "Am I adding anything not in the criteria?" — If yes: remove it.
- "Did I accidentally touch file modes or permissions?" — If yes: revert.
