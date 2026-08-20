# preserve-the-evidence

> A Claude Code plugin that preserves a permanent, readable record of every session — no paste-the-transcript, no guessing what changed, no hoping you remember what you asked for last Tuesday.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)
![Node.js](https://img.shields.io/badge/node-%E2%89%A516.0.0-brightgreen.svg)
![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey.svg)

---

## Highlights

- **Automatic per-session markdown transcripts.** Every Claude Code turn overwrites your log files — no manual save, no copy/paste, always up to date.
- **Dual output by default.** Out of the box you get *two* files per session: a readable summary and a complete audit trail with every tool input and result. Pick one-file-only by setting a mode.
- **Lives in your project.** Logs land in `docs/claude-log/` next to the code they document, so they're commit-able (or ignorable) just like any other file.
- **`/note` slash command.** Drop annotations into the transcript without Claude running any tools — useful for tagging sections, leaving reminders, or marking decision points.
- **Four modes.** `both` (default, two files), `none`, `summary`, or `full` — choose how much tool-call detail to keep.
- **Cross-platform, zero dependencies.** Pure Node.js using only built-ins (`fs`, `path`). No `bash`, no `jq`, no `npm install`. Same code runs unchanged on macOS, Linux, and Windows.
- **Fails safely.** Hook errors never block Claude Code — the script always exits 0 and writes diagnostics to stderr.

## What you get

After each turn, two files appear in your project (by default):

```
docs/claude-log/2026-04-20_a1b2c3d4.md         ← readable summary
docs/claude-log/2026-04-20_a1b2c3d4.full.md    ← complete audit trail
```

The summary file looks like this:

```
# Session a1b2c3d4 — 2026-04-20 14:32:17
_mode: summary_

## 🧑 User

Can you refactor the auth middleware to use the new token helper?

## 🤖 Claude

I'll refactor `src/auth/middleware.ts` to use `lib/tokens.ts`. Let me
start by reading both files.

🔧 **Read**: src/auth/middleware.ts

🔧 **Read**: lib/tokens.ts

## 🤖 Claude

The helper already exposes `verifyToken` — I'll swap the inline JWT
check for that call.

🔧 **Edit**: src/auth/middleware.ts

## 🧑 User

/preserve-the-evidence:note Stopped here. Need to add tests before the review.

## 🤖 Claude

📝 Noted.
```

The `.full.md` sibling contains the same conversation plus every tool input and tool result inside collapsible `<details>` blocks, so you have a complete replayable record.

Both files are refreshed on every turn until the session ends. A new session = a new pair of files.

## Prerequisites

| Requirement   | Why                                                          | How to check          |
| :------------ | :----------------------------------------------------------- | :-------------------- |
| Claude Code   | This is a Claude Code plugin                                 | `claude --version`    |
| Node.js ≥ 16  | The `Stop` hook runs `node scripts/log-session.js`           | `node --version`      |
| Git           | Claude Code uses git to clone the marketplace & plugin repos | `git --version`       |

If `node` is missing, [install it via `nvm`](https://github.com/nvm-sh/nvm#install--update-script) (lightest), the [official installer](https://nodejs.org/en/download), or your package manager.

## Installation

### Install from the marketplace (recommended)

Inside Claude Code, run:

```
/plugin marketplace add vibetemplates/preserve-the-evidence
/plugin install preserve-the-evidence
```

The plugin is active immediately — no restart required. The `Stop` hook will start writing logs on the next turn, and `/preserve-the-evidence:note` will appear in your slash-command list.

You do **not** need to `git clone` anything manually. Claude Code handles the clone under the hood and caches it in `~/.claude/plugins/`.

### Updating

When a new version is published, refresh the marketplace and reinstall:

```
/plugin marketplace update preserve-the-evidence
/plugin install preserve-the-evidence
```

### Uninstalling

```
/plugin uninstall preserve-the-evidence
/plugin marketplace remove preserve-the-evidence
```

### Local development

If you're hacking on the plugin itself, clone the repo and point Claude Code at the **plugin subdirectory** (not the repo root):

```
git clone https://github.com/vibetemplates/preserve-the-evidence.git
claude --plugin-dir ./preserve-the-evidence/preserve-the-evidence
```

The outer `preserve-the-evidence/` is the marketplace repo; the inner `preserve-the-evidence/` is the plugin itself. Any edits you make to scripts, hooks, or commands are picked up on the next `/reload-plugins` — no reinstall needed.

## Usage

### Automatic session logging

There's nothing to run. Once installed, every time Claude Code finishes a turn the `Stop` hook fires, reads the session's JSONL transcript, and rewrites your log file at:

```
<current-project>/docs/claude-log/<YYYY-MM-DD>_<first-8-of-session-id>.md
```

- **Same session** → same file, overwritten each turn.
- **New session** → new file with a new 8-char session ID.
- **Different project** → logs land in that project's `docs/claude-log/`. The plugin follows `cwd`.

### The `/note` command

Type this into Claude Code at any point:

```
/preserve-the-evidence:note Decided to split auth and billing into separate services.
```

Claude will respond **only** with `📝 Noted.` — no files read, no commands run, no tools invoked. On the next log write, your note appears in the transcript exactly like any other user message. Useful for:

- Marking decision points ("Picked PostgreSQL over SQLite because…")
- Leaving reminders mid-session ("Come back to this after lunch")
- Flagging unresolved questions ("Ask Dana about the rate-limit policy")
- Documenting intent so future-you can understand the transcript

## Configuration

### Logging modes

The plugin supports four modes. `both` is the default and writes two files per session. The other three each produce a single file:

| Mode      | Files written                          | Content                                                                                             |
| :-------- | :------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| `both`    | `<id>.md` **and** `<id>.full.md`       | **Default.** Summary + full audit, side by side.                                                    |
| `none`    | `<id>.md` only                         | User prompts and Claude's text responses only. No tool calls. Cleanest reading.                     |
| `summary` | `<id>.md` only                         | Prompts + text + one-line tool summaries like `🔧 **Bash**: \`npm test\``.                          |
| `full`    | `<id>.md` only                         | Prompts + text + every tool input/result inside collapsible `<details>` blocks. Best for debugging. |

### Setting the mode

Mode is resolved in this order of precedence (first match wins):

1. **CLI argument** passed to the hook script (advanced — only if you're wrapping the hook command yourself)
2. **Environment variable** `CLAUDE_LOG_MODE`
3. **File** at `<project>/.claude/log-mode` containing one of `none`, `summary`, `full`, or `both`
4. **Default**: `both`

Both the env var and the file accept any of `none`, `summary`, `full`, `both`. Invalid values are ignored and precedence falls through to the next source.

**Per-session override via env var:**

```bash
CLAUDE_LOG_MODE=full claude      # single file, full detail
CLAUDE_LOG_MODE=summary claude   # single file, summary only
CLAUDE_LOG_MODE=none claude      # single file, prompts only
```

**Persistent per-project setting** (no env var needed):

```bash
mkdir -p .claude && echo summary > .claude/log-mode
```

That file lives inside your project's `.claude/` directory (the same one Claude Code uses for project-local settings) and sticks for everyone who works on the project. It also takes effect on the next turn — no need to restart Claude Code.

To return to the dual-file default, delete the file:

```bash
rm .claude/log-mode
```

## Output format

### File naming

```
docs/claude-log/<YYYY-MM-DD>_<first-8-of-session-id>.md        ← summary (or whichever single mode is active)
docs/claude-log/<YYYY-MM-DD>_<first-8-of-session-id>.full.md   ← only in `both` mode
```

Example: `docs/claude-log/2026-04-20_a1b2c3d4.md` (+ `.full.md` by default)

Dates are ISO-format, UTC-based (from `new Date().toISOString()`). The 8-char session ID is a prefix of Claude Code's internal session UUID — enough to be unique in practice while staying readable in filenames.

### File structure

```markdown
# Session <short-id> — <date> <time>
_mode: <mode>_

## 🧑 User

<their message>

## 🤖 Claude

<Claude's text response>

🔧 **Tool**: <summary>     ← summary mode

<details><summary>🔧 Tool</summary>    ← full mode
```json
{ "tool input" }
```
</details>
```

### Multi-turn behavior

Each Stop event re-renders the entire session from the JSONL transcript, so the file always reflects the current state from turn 1 to now. If you switch modes mid-session, the next write reformats everything under the new mode.

## Managing logs

### Commit them

Logs are just markdown. Commit `docs/claude-log/` alongside your code if you want a historical record of how the codebase was built. Handy for:

- Onboarding (new teammates can read the thinking behind changes)
- Debugging regressions ("when did this function get added?")
- Documenting AI-assisted work for audit/compliance

### Ignore them

If you'd prefer to keep them local, add to your project's `.gitignore`:

```
docs/claude-log/
```

Or put them somewhere that's already ignored — the plugin always writes to `<project>/docs/claude-log/`, but you can symlink that to wherever you like.

## How it works

```
┌──────────────────────────────────────────────────────────────────────┐
│  Claude Code finishes a turn                                         │
│       │                                                              │
│       ▼                                                              │
│  Stop hook fires (from plugin's hooks/hooks.json)                    │
│       │                                                              │
│       ▼                                                              │
│  node scripts/log-session.js receives JSON on stdin:                 │
│    { session_id, transcript_path, cwd }                              │
│       │                                                              │
│       ▼                                                              │
│  Script reads transcript_path (JSONL), parses entries                │
│       │                                                              │
│       ▼                                                              │
│  Renders user + assistant blocks per current mode                    │
│       │                                                              │
│       ▼                                                              │
│  Writes to <cwd>/docs/claude-log/<date>_<short-id>.md                │
│       │                                                              │
│       ▼                                                              │
│  Exits 0 (always — even on error)                                    │
└──────────────────────────────────────────────────────────────────────┘
```

The plugin uses Claude Code's standard hook protocol. See [the Hooks reference](https://code.claude.com/docs/en/hooks) for the full hook spec.

## Troubleshooting

### Logs aren't appearing

1. **Check Node.js is on PATH.** `node --version` in the same shell where you launch `claude`. If missing, install via [`nvm`](https://github.com/nvm-sh/nvm#install--update-script).
2. **Check the plugin is installed and enabled.** Run `/plugin list` — you should see `preserve-the-evidence` with status **enabled**.
3. **Check the hook is registered.** Run `/hooks` — you should see a `Stop` entry pointing at `log-session.js`.
4. **Look for a `docs/claude-log/` directory in the project.** It's created on the first successful Stop event. If it isn't there after a few turns, Node isn't being found.

### The log file exists but is mostly empty

The script always writes a header even if it can't render anything. If the body is empty, the transcript JSONL probably doesn't contain user/assistant entries yet — try running one more turn.

### Mode changes aren't taking effect

Remember the precedence order: env var beats file. If you set `CLAUDE_LOG_MODE=summary` in your shell, a `.claude/log-mode` file with `full` won't win. Check with:

```bash
echo $CLAUDE_LOG_MODE
cat .claude/log-mode
```

The effective mode appears as `_mode: <mode>_` in the log file header — the most reliable check.

### Windows-specific notes

The quoted `${CLAUDE_PLUGIN_ROOT}` path in `hooks/hooks.json` handles spaces in paths (including `C:\Users\<name with space>\...`). No extra configuration needed. Uses Node's `path.join` throughout, so slashes resolve correctly on any OS.

### The hook runs but I see nothing

Errors go to stderr. If you launch `claude` from a terminal, you'll see them there. The hook also always exits 0 by design — so it never blocks Claude Code, but also never fails loudly. Check stderr first.

## Privacy

- **Where your data goes:** Logs are written to your local filesystem at `<project>/docs/claude-log/`. Nothing is sent over the network by this plugin.
- **What's logged:** Your prompts, Claude's responses, and (depending on mode) tool inputs and results. This can include file contents, command output, and anything else that appeared in the session.
- **Sensitive data:** Don't commit the log directory if your sessions contain secrets, credentials, or customer data. Use `.gitignore` (see above) or a `.gitattributes` filter. The plugin doesn't redact anything.

## Project structure

```
vibetemplates/preserve-the-evidence/           ← this repo
├── .claude-plugin/
│   └── marketplace.json                       ← marketplace catalog
├── preserve-the-evidence/                     ← the plugin itself
│   ├── .claude-plugin/
│   │   └── plugin.json                        ← plugin manifest
│   ├── commands/
│   │   └── note.md                            ← /note slash command
│   ├── hooks/
│   │   └── hooks.json                         ← Stop hook registration
│   └── scripts/
│       └── log-session.js                     ← the logger (pure Node, no deps)
├── memory-to-skill/                           ← companion plugin (see its README)
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── commands/
│       ├── mark.md                            ← /mark section delimiter
│       └── distill.md                         ← /distill history → skill
├── README.md
├── LICENSE                                    ← MIT
└── BUILD_PRESERVE_THE_EVIDENCE.md             ← original build spec
```

## Companion plugin: memory-to-skill

This marketplace also ships [`memory-to-skill`](memory-to-skill/), which mines the transcripts this plugin preserves: mark a section of a session that worked (`/mark <name>` live, or paste `[SKILL-START: <name>]` into a log file after the fact), then run `/distill` to turn it into a repeatable skill or slash command in your project's `.claude/` directory. Fully standalone — but the two are better together.

```
/plugin install memory-to-skill
```

## Contributing

PRs welcome. When changing behavior:

1. Test locally with `claude --plugin-dir ./preserve-the-evidence` (from the repo root).
2. Verify the hook still exits 0 on all code paths — the tests effectively are "does Claude Code keep working after a broken turn?"
3. Run `node -c preserve-the-evidence/scripts/log-session.js` before committing.
4. Bump `version` in both `.claude-plugin/marketplace.json` and `preserve-the-evidence/.claude-plugin/plugin.json` when shipping user-visible changes. (Per the [docs](https://code.claude.com/docs/en/plugin-marketplaces#version-resolution-and-release-channels), keep them in sync or let one be authoritative — `plugin.json` wins silently if both are set differently.)

Bug reports and feature requests: open an issue at https://github.com/vibetemplates/preserve-the-evidence/issues.

## License

[MIT](LICENSE) — free to use, modify, and distribute. Copyright © 2026 Edward Honour.

## Author

Edward Honour ([@edhonour](https://github.com/edhonour))

Built with [Claude Code](https://claude.com/claude-code).
