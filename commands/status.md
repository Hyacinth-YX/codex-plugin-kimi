---
description: Show active and recent Codex jobs for this repository, including review-gate status ([job-id] [--wait] [--timeout-ms <ms>] [--all])
---

Runtime path:
- Resolve the companion script in this order: (1) the absolute path given by the `codex-cli-runtime` skill (loaded at session start), (2) `$KIMI_PLUGIN_ROOT/scripts/codex-companion.mjs`, (3) `$HOME/.kimi-code/plugins/managed/codex/scripts/codex-companion.mjs`. Below it is written as `<companion>`.

Run via Bash:

```bash
node "<companion>" status "$ARGUMENTS"
```

If the user did not pass a job ID:
- Render the command output as a single Markdown table for the current and past runs in this session.
- Keep it compact. Do not include progress blocks or extra prose outside the table.
- Preserve the actionable fields from the command output, including job ID, kind, status, phase, elapsed or duration, summary, and follow-up commands.

If the user did pass a job ID:
- Present the full command output to the user.
- Do not summarize or condense it.
