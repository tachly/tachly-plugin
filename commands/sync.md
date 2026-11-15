---
description: Import older Claude Code session history to the server (no duplicates)
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" sync:*)
---

Run exactly one command via Bash with the `timeout` parameter set to 600000 and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" sync "$ARGUMENTS"
```

With no arguments, sessions from the last 30 days are imported; `--days N` changes the period and `--all` takes everything. Running it again is safe.

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
