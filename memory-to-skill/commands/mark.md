---
description: Mark the start (or end) of a session section worth distilling into a skill
argument-hint: <name> | end [<name>]
---

The user is placing a skill marker in the session record. Their input:

> $ARGUMENTS

Do NOT read files, run commands, or use any tools. Reply with ONLY the single
marker line described below — no other text before or after it.

1. Determine the name, then slugify it: lowercase, trim, spaces and underscores
   become hyphens, drop any character that is not a letter, digit, or hyphen.
   If nothing remains, use `unnamed`.
2. If the input begins with the word `end`, the name is whatever follows that
   word. Reply exactly:

   [SKILL-END: <slug>]

3. Otherwise the entire input is the name. Reply exactly:

   [SKILL-START: <slug>]
