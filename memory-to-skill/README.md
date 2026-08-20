# memory-to-skill (plugin)

Turn the sections of your Claude Code sessions that actually *worked* into repeatable skills and slash commands. Full documentation: https://github.com/maludb-ed/memory-to-skill

## Quick reference

```
/memory-to-skill:mark <name>            drop [SKILL-START: <name>] into the session
/memory-to-skill:mark end [<name>]      drop [SKILL-END: <name>] (optional)
/memory-to-skill:distill <name>         distill the marked section into a skill
```

`/distill` extras: `from <file>` to read a specific transcript, `as skill` / `as command` to force the output form, `personal` to write to `~/.claude/` instead of the project's `.claude/`.

Markers can also be pasted by hand into any saved markdown transcript (e.g. the `docs/claude-log/` files written by the [preserve-the-evidence](https://github.com/vibetemplates/preserve-the-evidence) plugin) — `/distill` searches there when the current conversation has no match.

Output lands in `.claude/skills/<name>/SKILL.md` (default) or `.claude/commands/<name>.md`. Commit those directories to share with your team.

[MIT](../LICENSE) — Copyright © 2026 Edward Honour.
