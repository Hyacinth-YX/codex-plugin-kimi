# Changelog

## 1.0.6-kimi.1

- Ported the Codex plugin (upstream openai/codex-plugin-cc 1.0.6) to Kimi Code CLI
- Slash commands, rescue sub-agent, skills, and SessionStart/SessionEnd/Stop hooks adapted to the Kimi plugin manifest
- Stop review gate emits Kimi's blocking protocol (exit code 2 + stderr); gate review timeout reduced to 8 minutes to fit Kimi's 600s hook limit
- `/codex:transfer` removed (depends on Claude Code session transcripts)

## 1.0.0

- Initial version of the Codex plugin for Claude Code
