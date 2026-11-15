---
description: Show Tachly status (server, queue, last send, version)
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" status:*)
---

Run exactly one command via Bash and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" status
```

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
