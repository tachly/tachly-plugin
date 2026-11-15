---
description: Connect this machine to a Tachly server (sign in through the browser)
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" login:*)
---

Run exactly one command via Bash with the `timeout` parameter set to 600000 and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" login $ARGUMENTS
```

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
