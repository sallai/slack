---
name: slack
description: Use Slack through a bundled single-file Python CLI. Use when Codex needs to authenticate to Slack, search Slack, read or export channel/conversation history, inspect threads, download or upload files, post/update/delete messages, manage reactions, pins, bookmarks, users, team info, emoji, or call raw Slack Web API methods. Also use when Slack credentials are missing or invalid and the user needs an agent-readable setup walkthrough.
---

# Slack

## Core Rule

Use the bundled CLI for Slack work:

```sh
uv run <skill-dir>/scripts/slack-cli ...
```

Resolve `<skill-dir>` to this skill folder before running commands. Always use `uv` for Python execution. Do not create Python packages for this skill; keep future Python tooling as single-file scripts with PEP 723 metadata.

Prefer documentation, command help, setup text, manifest YAML, examples, and reusable assets embedded in `scripts/slack-cli` over separate files. Start with:

```sh
uv run <skill-dir>/scripts/slack-cli setup
uv run <skill-dir>/scripts/slack-cli --help
uv run <skill-dir>/scripts/slack-cli conversation --help
uv run <skill-dir>/scripts/slack-cli chat --help
uv run <skill-dir>/scripts/slack-cli file --help
```

## Credentials

Load credentials from `~/.slack/env`. Prefer `SLACK_USER_TOKEN` when the CLI should act as the user. `SLACK_BOT_TOKEN` and `SLACK_TOKEN` are accepted for bot-token or generic-token workflows.

If credentials are missing or invalid, run:

```sh
uv run <skill-dir>/scripts/slack-cli setup
```

Use the command's STDOUT to guide the user. Never ask the user to paste real tokens into chat. Never print or commit real Slack tokens.

## Workflows

Use read/export commands before mutations:

```sh
uv run <skill-dir>/scripts/slack-cli me
uv run <skill-dir>/scripts/slack-cli conversation list --limit 50
uv run <skill-dir>/scripts/slack-cli conversation history "<channel-id>" --limit 50
uv run <skill-dir>/scripts/slack-cli conversation replies "<channel-id>" "<message-ts>"
uv run <skill-dir>/scripts/slack-cli conversation export "<channel-id>" --days 3 --include-replies --download-files --output-dir slack-export
uv run <skill-dir>/scripts/slack-cli search messages "query"
uv run <skill-dir>/scripts/slack-cli file list --channel "<channel-id>"
```

Use write commands only when the user has requested changes. After any mutation, verify by reading back with the CLI when Slack exposes a read path.

Common mutation surfaces:

```sh
uv run <skill-dir>/scripts/slack-cli chat post "<channel-id>" --text "message"
uv run <skill-dir>/scripts/slack-cli chat update "<channel-id>" "<message-ts>" --text "message"
uv run <skill-dir>/scripts/slack-cli chat delete "<channel-id>" "<message-ts>"
uv run <skill-dir>/scripts/slack-cli file upload "./path" --channel "<channel-id>"
uv run <skill-dir>/scripts/slack-cli reaction add thumbsup "<channel-id>" "<message-ts>"
uv run <skill-dir>/scripts/slack-cli pin add "<channel-id>" "<message-ts>"
uv run <skill-dir>/scripts/slack-cli bookmark add "<channel-id>" "title" "https://example.com"
```

Use the raw API escape hatch for endpoints that do not need a new high-level wrapper:

```sh
uv run <skill-dir>/scripts/slack-cli api GET auth.test
uv run <skill-dir>/scripts/slack-cli api POST chat.postMessage --data '{"channel":"C1234567890","text":"hello"}'
```

Use `conversation export` for channel history tasks where files, images, or thread context matter. It writes `history.json`, `messages.md`, `files.json`, and optional downloads under the chosen output directory.

## Self-Maintenance

If a needed Slack capability is missing:

1. First check `setup`, `--help`, subcommand help, and the raw `api` command.
2. If the raw API command is enough, use it without changing the script.
3. If a reusable CLI feature is needed, explain the missing capability and ask the user before editing `scripts/slack-cli`.
4. After explicit approval, update only the bundled single-file CLI unless the user asks otherwise.
5. Keep new docs, setup guidance, manifests, examples, and small reusable snippets embedded in the CLI script when practical.
6. Re-run the updated CLI and verify the original Slack task succeeds.

When changing `scripts/slack-cli`, keep it dependency-free unless a dependency is clearly necessary and declared in the PEP 723 block.
