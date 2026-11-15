---
description: Turn Coach hints in Claude Code on or off, or mute one tip
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" coach:*)
---

Run exactly one command via Bash and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" coach "$ARGUMENTS"
```

`on` and `off` switch the one-line Coach hint that Tachly can show at the start of a session. `mute <id>` hides one tip (the server hides it on the Coach page too), and `mute all` turns every hint off. Nothing is sent to the model.

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
