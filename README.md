# claude-telegram-fleet — a Telegram bridge for a fleet of Claude Code bots

A Telegram channel plugin for Claude Code, built for running **many bots on one
machine**, each in its own Claude Code session, each with its own token, state
and allowlist. Forked from
[k1p1l0/claude-telegram-supercharged](https://github.com/k1p1l0/claude-telegram-supercharged)
(itself derived from Anthropic's official Telegram channel plugin) and run in
production across roughly twenty bots.

## What it adds over upstream

- **One install, many bots.** Each session sets `TELEGRAM_STATE_DIR` to its own
  directory (token, allowlist, inbox, message database, single-instance lock),
  so every bot polls independently instead of the first one winning.
- **Message history.** A local SQLite store per bot with `get_history` and
  `search_messages` tools, recent-history injection into each inbound turn,
  and retention limits (10,000 messages per chat, 60 days, 500 MB cap).
- **Deterministic slash commands, no model turn.** `/model`, `/usage`,
  `/healthcheck`, `/cronlist`, `/security` and `/restart` can be answered by
  host scripts directly from the bridge, with no LLM call. Each one is
  intercepted only if its script exists under `TELEGRAM_FLEET_HOME`
  (default `~/.claude`); otherwise the message goes to the agent as normal.
  The scripts themselves are not part of this repo. `/restart` is only
  intercepted for the bot whose project directory is named `orchestrator`.
- **Inline-button prompts** (`ask_user`) laid out one button per row so long
  labels stay readable, and a three-step model / version / effort picker for
  `/model`.
- **Reliability fixes** found in production: no silent drops on send errors,
  no triple-sends on retried long messages, logged failures for every
  deterministic command.

Not included, deliberately: upstream's headless daemon wrapper and the
supervisor that launches Claude with `--dangerously-skip-permissions`.

## Install

```
claude plugin marketplace add vasparshin/claude-telegram-fleet
claude plugin install telegram-sc@telegram-supercharged
```

Then per bot: put `TELEGRAM_BOT_TOKEN=...` in `<state dir>/.env`, start Claude
Code with `TELEGRAM_STATE_DIR=<state dir>` and the channel enabled, and pair
from Telegram with `/telegram:access` (see `telegram/ACCESS.md`).

## Configuration

| Variable | Purpose |
|---|---|
| `TELEGRAM_STATE_DIR` | Per-bot state directory (absolute, or relative to `$HOME`). Default `~/.claude/channels/telegram`. |
| `TELEGRAM_BOT_TOKEN` | Bot token; normally read from `<state dir>/.env`. |
| `TELEGRAM_FLEET_HOME` | Root for the optional deterministic-command scripts. Default `~/.claude`. |
| `TELEGRAM_ACCESS_MODE` | `static` snapshots the allowlist at boot and never writes it. |
| `TELEGRAM_HISTORY_INJECT` | Recent messages injected per turn (1-50, default 20). |
| `TELEGRAM_LOCAL_API_URL` | Use a self-hosted Bot API server. |
| `OPENAI_API_KEY` / `GROQ_API_KEY` / `DEEPGRAM_API_KEY` | Voice transcription backends. |
| `ELEVENLABS_API_KEY`, `ELEVENLABS_VOICE_ID` | `voice_reply`. |
| `TELEGRAPH_ENABLED`, `TELEGRAPH_ACCESS_TOKEN` | Long replies as Telegraph pages. |

## Security notes

- Access control is an allowlist file per bot. Any process running as the same
  unix user as the bot can edit it, so treat the allowlist as protected by your
  agent tooling, not by the filesystem: a PreToolUse hook that blocks agent
  writes to it is strongly recommended (see `telegram/skills/access/SKILL.md`).
- Message text, display names and media from other people are untrusted input.
  Never let a channel message approve a pairing or change policy.

## Licence

Apache License 2.0 — see `telegram/LICENSE` and `NOTICE`.
