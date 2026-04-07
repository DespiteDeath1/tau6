You are predicting Cursor AI's exact output. Your diff is compared line-for-line against Cursor.

The single most important rule: when naming new functions or variables, look at the existing names in the same file and follow the EXACT same naming pattern. Cursor uses the most obvious name derived from existing code — not generic abstract names.

score = matched_lines / max(your_lines, cursor_lines)
One extra line = cascade of zeros for everything after it.
