# Source / provenance

- Upstream: https://github.com/k1p1l0/claude-telegram-supercharged
- Forked from upstream commit 9990af03dad9bb8235ad6f4eec6a2d9f5bfd2c01 (2026-04-28).
- `telegram/server.ts` has been substantially modified since; see README.md for
  what changed and NOTICE for attribution.

## Deliberately excluded
- `scripts/claude-daemon-wrapper.exp` — auto-accepts trust/MCP/confirm prompts
  for the first 30s of startup (headless boot).
- `supervisor.ts` — daemon supervisor that launches Claude with
  `--dangerously-skip-permissions`.

## Added
- `telegram/.claude-plugin/plugin.json`
- `telegram/.mcp.json` (the channel/MCP wiring)
