# memory-to-skill

> A Claude Code plugin that turns the sections of your sessions that actually *worked* into repeatable skills and slash commands — mark an attempt, and when it succeeds, distill it.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Version](https://img.shields.io/badge/version-0.1.0-blue.svg)
![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey.svg)

---

## The idea

Some sessions produce a result you'll want again: a migration recipe, a debugging procedure, a deploy sequence that finally worked. Rather than hoping you remember the prompts, you delimit the good part with markers and run `/distill`. Claude re-reads that section as *evidence* — extracting the goal, keeping the working path, converting dead ends into "gotchas", generalizing project-specific details — and writes a proper skill or slash command from it.

## Highlights

- **Two markdown commands, zero runtime.** No hooks, no scripts, no Node.js, no dependencies. Works identically on macOS, Linux, and Windows.
- **Mark live or after the fact.** `/mark <name>` drops a marker into the session as you work; the identical `[SKILL-START: <name>]` string can be pasted into a saved transcript later when you realize yesterday's session had the good stuff.
- **Skill or command output.** Procedures become auto-triggering skills (`SKILL.md`); replayable prompts become parameterized slash commands. Claude picks the right form, or you force one.
- **Project scope by default.** Output lands in your project's `.claude/` directory, so committing it shares the skill with your whole team. A `personal` flag targets `~/.claude/` instead.
- **Distills, never copies.** The output is instructions for a fresh Claude — dead ends become warnings, specifics become placeholders, and the verification step that proved success comes along.

## Prerequisites

| Requirement | Why                                                     | How to check       |
| :---------- | :------------------------------------------------------ | :----------------- |
| Claude Code | This is a Claude Code plugin                            | `claude --version` |
| Git         | Claude Code uses git to clone the marketplace under the hood | `git --version` |

That's it — the plugin itself is pure markdown.

## Installation

Inside Claude Code, run:

```
/plugin marketplace add maludb-ed/memory-to-skill
/plugin install memory-to-skill
```

The plugin is active immediately — no restart required. `/memory-to-skill:mark` and `/memory-to-skill:distill` appear in your slash-command list (the short forms `/mark` and `/distill` work too when no other plugin claims them).

> Note: for anyone other than the repo owner to install, the repository must be public (or shared with them).

### Updating

```
/plugin marketplace update memory-to-skill
/plugin install memory-to-skill
```

### Uninstalling

```
/plugin uninstall memory-to-skill
/plugin marketplace remove memory-to-skill
```

### Local development

Clone the repo and point Claude Code at the **plugin subdirectory** (not the repo root):

```
git clone https://github.com/maludb-ed/memory-to-skill.git
claude --plugin-dir ./memory-to-skill/memory-to-skill
```

The outer `memory-to-skill/` is the marketplace repo; the inner `memory-to-skill/` is the plugin itself. Edits to the command files are picked up on the next `/reload-plugins`.

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

Claude finds the marker in the current conversation, analyzes everything after it, and writes the skill. An end marker (`/mark end fix-flaky-tests`) is optional — an unclosed section runs to the next start marker or the end of the session.

### Mark after the fact (manual)

Realized *yesterday's* session had the good stuff? Open a saved transcript — any markdown transcript works; the [preserve-the-evidence](https://github.com/vibetemplates/preserve-the-evidence) plugin writes them automatically to `docs/claude-log/` — and paste marker lines around the good section:

```
[SKILL-START: fix-flaky-tests]
...the section...
[SKILL-END: fix-flaky-tests]
```

Then run `/distill fix-flaky-tests` — it searches `docs/claude-log/` for the marker. You can also point at a file directly:

```
/memory-to-skill:distill fix-flaky-tests from docs/claude-log/2026-08-19_a1b2c3d4.full.md
```

### `/distill` options

| Argument                 | Effect                                                                                    |
| :----------------------- | :---------------------------------------------------------------------------------------- |
| `<name>`                 | Which marked section to distill. Omit to have Claude list what it finds.                  |
| `from <file>`            | Read a specific transcript file instead of searching.                                     |
| `as skill` / `as command`| Force the output form. Default: Claude picks — skill for procedures, command for prompts. |
| `personal`               | Write to `~/.claude/` instead of the project's `.claude/`.                                |

### What you get

- **Skill** (default): `.claude/skills/<name>/SKILL.md` — auto-triggers when its description matches future work. Contains steps, gotchas distilled from your dead ends, and the verification check that proved success originally.
- **Command**: `.claude/commands/<name>.md` — a generalized, parameterized prompt you fire explicitly with `/<name> <args>`.

Commit `.claude/skills/` (or `.claude/commands/`) to share the result with everyone who works on the project.

### How `/distill` finds sections

It searches in order:

1. **The current conversation** — no files needed.
2. **An explicit `from <file>` argument.**
3. **`docs/claude-log/*.md`** — when a summary/full pair both match, it reads the `.full.md` for tool-level detail.

If several marked sections match, it lists them and asks which one you meant.

## Marker reference

```
[SKILL-START: <name>]     begin a section
[SKILL-END: <name>]       end it (optional)
```

Names are slugified: lowercase, hyphens for spaces. `/mark` emits exactly these strings, so live marking and manual pasting are interchangeable.

## Works great with preserve-the-evidence

This plugin is fully standalone — it can distill from the current session with nothing else installed. But it pairs naturally with [preserve-the-evidence](https://github.com/vibetemplates/preserve-the-evidence), which automatically writes every session as markdown to `docs/claude-log/`. One preserves the evidence, the other mines it: install both and every past session becomes distillable.

## Project structure

```
maludb-ed/memory-to-skill/                 ← this repo (a one-plugin marketplace)
├── .claude-plugin/
│   └── marketplace.json                   ← marketplace catalog
├── memory-to-skill/                       ← the plugin itself
│   ├── .claude-plugin/
│   │   └── plugin.json                    ← plugin manifest
│   ├── README.md                          ← plugin-level quick reference
│   └── commands/
│       ├── mark.md                        ← /mark — section delimiter
│       └── distill.md                     ← /distill — history → skill
├── README.md
└── LICENSE                                ← MIT
```

## Contributing

PRs welcome. When changing behavior:

1. Test locally with `claude --plugin-dir ./memory-to-skill` (from the repo root).
2. Bump `version` in both `.claude-plugin/marketplace.json` and `memory-to-skill/.claude-plugin/plugin.json` when shipping user-visible changes, and keep them in sync.

Bug reports and feature requests: open an issue at https://github.com/maludb-ed/memory-to-skill/issues.

## License

[MIT](LICENSE) — free to use, modify, and distribute. Copyright © 2026 Edward Honour.

## Author

Edward Honour ([@edhonour](https://github.com/edhonour))

Built with [Claude Code](https://claude.com/claude-code).
