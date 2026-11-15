---
description: Turn Claude Code OpenTelemetry export to the Tachly server on or off (no prompt text is sent)
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" telemetry:*)
---

Run exactly one command via Bash and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" telemetry "$ARGUMENTS"
```

`on` asks the server for a personal telemetry key and sets the OpenTelemetry variables in the `env` block of the Claude settings (the previous values are saved); `off` restores them and revokes the key. The change takes effect in a new Claude Code session, in the terminal and in the desktop app.

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
