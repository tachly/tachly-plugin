---
description: Disconnect this machine from the Tachly server and turn its telemetry off (the queue is kept)
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" logout:*)
---

Run exactly one command via Bash and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" logout
```

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
