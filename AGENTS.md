# Scoring System
Your diff is scored by EXACT POSITIONAL LINE MATCHING against a reference solution.
Score = matched_positions / max(your_lines, reference_lines)

One extra line shifts ALL subsequent positions → zero score for rest of file.
One missing line shifts ALL subsequent positions → zero score for rest of file.
FEWER LINES IS ALWAYS SAFER. An empty diff scores 0, but a wrong diff also scores 0.

# Non-Negotiable Rules
NEVER add blank lines unless they exist in surrounding code.
NEVER add comments or docstrings.
NEVER add type annotations not present in surrounding code.
NEVER reorder imports.
NEVER refactor, rename, or clean up unrelated code.
NEVER add error handling unless explicitly required.
NEVER create new files unless explicitly required.
NEVER run tests or verify your changes.
NEVER summarize or explain your changes.

# Workflow
1. Read the task title and ALL acceptance criteria completely.
2. Identify ONLY the files that must change.
3. Read each target file IN FULL before touching anything.
4. Make the MINIMUM change that satisfies each criterion.
5. Process files in ALPHABETICAL path order.
6. Within each file, edit TOP TO BOTTOM.
7. Stop immediately after edits. Do not re-read or verify.

# Style Rules
Match the file's existing style EXACTLY:
- Indentation: tabs vs spaces, exact width
- Quotes: single vs double, match the file
- Semicolons: present or absent, match the file
- Trailing commas: match surrounding code
- Spacing around operators: match the file

# Ambiguity Rule
When a change is ambiguous → choose the NARROWEST interpretation.
When still ambiguous after reading the file → SKIP that change entirely.
A smaller correct patch always beats a larger wrong one.

# For New Files
Copy the structure of the nearest existing similar file exactly.
Same imports pattern, same export style, same error handling style.

# Pre-Edit Checklist
Before EVERY edit confirm:
1. This change is explicitly required by the task.
2. This is the minimum possible implementation.
3. This matches the local file style exactly.
4. This does not touch unrelated lines.
If any check fails → do NOT make that edit.
