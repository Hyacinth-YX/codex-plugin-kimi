---
description: Cancel an active background Codex job in this repository ([job-id])
---

Runtime path:
- Resolve the companion script in this order: (1) the absolute path given by the `codex-cli-runtime` skill (loaded at session start), (2) `$KIMI_PLUGIN_ROOT/scripts/codex-companion.mjs`, (3) `$HOME/.kimi-code/plugins/managed/codex/scripts/codex-companion.mjs`. Below it is written as `<companion>`.

Run via Bash:

```bash
node "<companion>" cancel "$ARGUMENTS"
```

Present the command output to the user exactly as returned.
