---
description: Open the Tachly dashboard in your browser (signed in by this device, no password)
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" me:*)
---

Run exactly one command via Bash and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" me
```

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
