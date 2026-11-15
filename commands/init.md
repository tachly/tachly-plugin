---
description: Link this repository to a Tachly project (creates .tachly.json)
allowed-tools: Bash("${CLAUDE_PLUGIN_ROOT}/native/tachly" init:*)
---

Run exactly one command via Bash and nothing else:

```
"${CLAUDE_PLUGIN_ROOT}/native/tachly" init "$ARGUMENTS"
```

The argument is the project name or slug (without it, the name comes from an existing `.tachly.json` or from the folder name). If no project with that slug exists yet, the server creates it. `--force` overwrites a `.tachly.json` that points to a different project.

Show its output to the user verbatim, with no explanations or additions. Run nothing else and read no files.
