# memory-to-skill

> Turn the sections of your Claude Code sessions that actually *worked* into repeatable skills and slash commands — mark an attempt, and when it succeeds, distill it.

Companion to [preserve-the-evidence](../preserve-the-evidence/), but fully standalone: it can distill from the current session with no other plugin installed. The two pair naturally — one preserves the evidence, the other mines it.

## The idea

Some sessions produce a result you'll want again: a migration recipe, a debugging procedure, a deploy sequence that finally worked. Rather than hoping you remember the prompts, you delimit the good part with markers and run `/distill`. Claude re-reads that section as *evidence* — extracting the goal, keeping the working path, converting dead ends into "gotchas", generalizing project-specific details — and writes a proper skill or slash command from it.

## Installation

```
/plugin marketplace add vibetemplates/preserve-the-evidence
/plugin install memory-to-skill
```

No hooks, no scripts, no Node.js requirement — the plugin is two markdown commands.

## Usage

### Mark, then distill (live)

Before starting an attempt you suspect might be worth keeping:

```
/memory-to-skill:mark fix-flaky-tests
```

Claude replies with only `[SKILL-START: fix-flaky-tests]` — no tools run, nothing else happens. Work normally. If the attempt succeeds:

```
/memory-to-skill:distill fix-flaky-tests
```

An end marker (`/mark end fix-flaky-tests`) is optional — an unclosed section runs to the next start marker or the end of the session.

### Mark after the fact (manual)

Realized *yesterday's* session had the good stuff? Open the transcript (e.g. `docs/claude-log/2026-08-19_a1b2c3d4.full.md` if you use preserve-the-evidence, or any markdown transcript) and paste these lines around the good section:

```
[SKILL-START: fix-flaky-tests]
...the section...
[SKILL-END: fix-flaky-tests]
```

Then run `/distill fix-flaky-tests` — it searches `docs/claude-log/` for the marker. You can also point at a file directly:

```
/memory-to-skill:distill fix-flaky-tests from docs/claude-log/2026-08-19_a1b2c3d4.full.md
```

### Options

| Argument | Effect |
| :--- | :--- |
| `<name>` | Which marked section to distill. Omit to have Claude list what it finds. |
| `from <file>` | Read a specific transcript file instead of searching. |
| `as skill` / `as command` | Force the output form. Default: Claude picks (skill for procedures, command for replayable prompts). |
| `personal` | Write to `~/.claude/` instead of the project's `.claude/`. |

### What you get

- **Skill** (default): `.claude/skills/<name>/SKILL.md` — auto-triggers when its description matches future work. Contains steps, gotchas distilled from your dead ends, and the verification check that proved success originally.
- **Command**: `.claude/commands/<name>.md` — a generalized, parameterized prompt you fire explicitly with `/<name> <args>`.

Project scope by default, so committing `.claude/skills/` shares the skill with your whole team.

## How it finds sections

`/distill` searches in order: the **current conversation** (no files needed), an explicit **`from <file>`** argument, then **`docs/claude-log/*.md`**. When a summary/full pair both match, it reads the `.full.md` for tool-level detail.

## Marker reference

```
[SKILL-START: <name>]     begin a section
[SKILL-END: <name>]       end it (optional)
```

Names are slugified: lowercase, hyphens for spaces. `/mark` emits exactly these strings, so live marking and manual pasting are interchangeable.

## License

[MIT](../LICENSE) — Copyright © 2026 Edward Honour.
