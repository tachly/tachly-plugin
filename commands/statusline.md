---
description: Turn collection of Claude Code limits (5 hours and 7 days) on or off through a status line wrapper
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" statusline-install:*)
---

Run exactly one command via Bash and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" statusline-install "$ARGUMENTS"
```

`on` wraps the current status line (the original is kept and keeps working) and, in the Claude desktop app where there is no status line, lets the plugin ask the signed-in terminal `claude` for the limits (no model call, no cost); `off` restores the status line as it was and stops both. Limits are recorded only while the plugin is signed in; signed out, the wrapper just runs the original. The change takes effect in a new Claude Code session.

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
