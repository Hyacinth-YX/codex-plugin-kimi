---
description: Show the stored final output for a finished Codex job in this repository ([job-id])
---

Runtime path:
- Resolve the companion script in this order: (1) the absolute path given by the `codex-cli-runtime` skill (loaded at session start), (2) `$KIMI_PLUGIN_ROOT/scripts/codex-companion.mjs`, (3) `$HOME/.kimi-code/plugins/managed/codex/scripts/codex-companion.mjs`. Below it is written as `<companion>`.

Run via Bash:

```bash
node "<companion>" result "$ARGUMENTS"
```

Present the full command output to the user. Do not summarize or condense it. Preserve all details including:
- Job ID and status
- The complete result payload, including verdict, summary, findings, details, artifacts, and next steps
- File paths and line numbers exactly as reported
- Any error messages or parse errors
- Follow-up commands such as `/codex:status <id>` and `/codex:review`
