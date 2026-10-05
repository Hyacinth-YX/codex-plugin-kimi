---
description: Check whether the local Codex CLI is ready and optionally toggle the stop-time review gate ([--enable-review-gate|--disable-review-gate])
---

Runtime path:
- Resolve the companion script in this order: (1) the absolute path given by the `codex-cli-runtime` skill (loaded at session start), (2) `$KIMI_PLUGIN_ROOT/scripts/codex-companion.mjs`, (3) `$HOME/.kimi-code/plugins/managed/codex/scripts/codex-companion.mjs`. Below it is written as `<companion>`.

Run:

```bash
node "<companion>" setup --json $ARGUMENTS
```

If the result says Codex is unavailable and npm is available:
- Use `AskUserQuestion` exactly once to ask whether to install Codex now.
- Put the install option first and suffix it with `(Recommended)`.
- Use these two options:
  - `Install Codex (Recommended)`
  - `Skip for now`
- If the user chooses install, run:

```bash
npm install -g @openai/codex
```

- Then rerun:

```bash
node "<companion>" setup --json $ARGUMENTS
```

If Codex is already installed or npm is unavailable:
- Do not ask about installation.

Output rules:
- Present the final setup output to the user.
- If installation was skipped, present the original setup output.
- If Codex is installed but not authenticated, tell the user to run `codex login` in a terminal (ChatGPT account or API key) and then re-run `/codex:setup`.
