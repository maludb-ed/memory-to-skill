---
description: Turn a marked section of session history into a repeatable skill or slash command
argument-hint: [<marker-name>] [from <log-file>] [as skill|command] [personal]
---

# Distill a session section into something repeatable

The user wants to turn a section of prompt history that WORKED into a reusable
skill or slash command. Their input:

> $ARGUMENTS

## Step 1 — Parse the request

From the input above, extract:

- **marker name** — the first bare word, if any.
- **source** — `from <path>`: a specific log file to read instead of searching.
- **form override** — `as skill` or `as command` skips the choice in Step 4.
- **scope override** — the word `personal` writes to `~/.claude/` instead of
  the project's `.claude/` directory. Default is project scope.

## Step 2 — Locate the section

Markers look like `[SKILL-START: <name>]` and `[SKILL-END: <name>]`. They were
placed either live via the /mark command or pasted into a log file by hand
after the fact. Section boundaries:

- A section runs from its START marker to the END marker with the same name.
- If there is no matching END, it runs to the next SKILL-START or to the end
  of the record, whichever comes first.
- An END whose name is `unnamed` closes the most recently opened START.

Search in this order:

1. **The current conversation.** If the named marker (or, when no name was
   given, any SKILL-START marker) appears in the conversation itself, use the
   conversation as the source — no file reading needed. Ignore marker text
   that appears inside this command's instructions or inside the /mark
   command's own definition; only actual conversation messages count.
2. **The `from <log-file>` argument**, if one was given. Read that file.
3. **Project logs.** Search `docs/claude-log/` (written by the
   preserve-the-evidence plugin, but any markdown transcript works), e.g.
   `grep -ln "SKILL-START: <name>" docs/claude-log/*.md`. When a `.md` /
   `.full.md` pair both match, read the `.full.md` — it has the tool detail.

If several distinct marked sections match, list them (name, source, first line
of the section) and ask the user which to distill. If none is found, stop and
tell the user the two ways to mark a section: run `/mark <name>` live in a
session, or paste `[SKILL-START: <name>]` (and optionally
`[SKILL-END: <name>]`) lines directly into a transcript file in
`docs/claude-log/`.

## Step 3 — Analyze the section

Treat the section as evidence of what worked, not as text to copy.

1. **Goal** — what was the user actually trying to accomplish?
2. **Working path** — the sequence of steps and decisions that led to success.
3. **Dead ends** — attempts that failed or were corrected. Do NOT include
   them as steps; convert the instructive ones into warnings (gotchas).
4. **Generalize** — replace project-specific paths, names, and values with
   placeholders (or `$ARGUMENTS` for a command) unless they are intrinsic to
   this project. Test each instruction: would it still make sense on a
   different day, in a different repo?
5. **Verification** — find the check that proved success in the original
   session (a test run, a build, a visual check). It belongs in the output.
6. Ignore the marker lines themselves and the /mark exchanges around them.

## Step 4 — Choose the form

- **Skill** (`SKILL.md`) — the DEFAULT. Choose when the section encodes a
  procedure, convention, or technique: "whenever doing X, here is how."
  Skills load automatically when their description matches the task at hand.
- **Slash command** (a `.md` file in `commands/`) — choose when the section is
  a replayable prompt the user will fire explicitly, usually with an argument
  ("run this same recipe against a different table").

Honor an override from Step 1. If genuinely ambiguous, ask the user one short
question rather than guessing.

## Step 5 — Write it

Destinations (create directories as needed):

| Form    | Project scope (default)           | Personal scope (`personal`)         |
| :------ | :-------------------------------- | :---------------------------------- |
| Skill   | `.claude/skills/<name>/SKILL.md`  | `~/.claude/skills/<name>/SKILL.md`  |
| Command | `.claude/commands/<name>.md`      | `~/.claude/commands/<name>.md`      |

If the destination already exists, summarize what would change and ask before
overwriting.

Skill template:

```markdown
---
name: <name>
description: <What it does, one sentence. Then triggers: "Use when the user asks to X, mentions Y, or needs Z.">
---

# <Title>

<One paragraph: what this accomplishes and when to reach for it.>

## Steps

1. ...

## Gotchas

- <warnings distilled from the dead ends>

## Verification

<How to confirm it worked — the check that succeeded in the original session.>
```

Command template:

```markdown
---
description: <one line>
argument-hint: <expected arguments>
---

<The generalized, replayable prompt. Use $ARGUMENTS where the user's input goes.>
```

Quality bar, either form:

- The `description` frontmatter decides whether a skill ever triggers — spend
  real effort on it and include concrete "use when" phrases.
- Instructions, not transcript: never paste conversation excerpts. Write what
  a fresh Claude with none of this context should DO.
- Keep it short — well under 150 lines. If genuine reference material is
  needed, put it in files next to SKILL.md and point to them from the body.
- Include the Verification section; a skill that can't check its own success
  is a recipe for silent failure.

## Step 6 — Report

Tell the user:

1. The file path written.
2. How it activates — the trigger phrases for a skill, or `/<name> <args>`
   for a command.
3. Two or three bullets: what was kept, what was generalized, and which dead
   ends became gotchas.
4. For project scope: suggest committing `.claude/skills/` (or
   `.claude/commands/`) so the whole team gets it.
