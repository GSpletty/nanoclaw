# Local Customizations

This file tracks modifications made to this NanoClaw install beyond the upstream
([qwibitai/nanoclaw](https://github.com/qwibitai/nanoclaw)). Useful when running
`/update-nanoclaw` to anticipate merge conflicts.

---

## Discord Channel

**Source:** Merged from `discord` remote (`qwibitai/nanoclaw-discord`)
**Commit:** `82f260d` — *Merge remote-tracking branch 'discord/main'*

**Files added/modified:**
- `src/channels/discord.ts` — Discord bot channel implementation
- `src/channels/discord.test.ts` — Tests for the Discord channel
- `src/channels/index.ts` — Registers Discord at startup
- `package.json` / `package-lock.json` — Added `discord.js` dependency

**Required env vars:** `DISCORD_BOT_TOKEN`, `DISCORD_MAIN_CHANNEL_ID`

**Notes:** The discord skill branch is kept in sync via a GitHub Actions workflow
(`.github/workflows/fork-sync-skills.yml`). Future upstream merges should not
conflict here unless core channel registry or types change.

---

## Notion + Google Workspace MCP Servers

**Added:** 2026-03-25

**What it does:** Gives the container agent tools to read/write Notion databases
and access Google Calendar and Gmail.

**Files modified:**
- `container/agent-runner/src/index.ts` — Added `notion` and `google-workspace`
  entries to the `mcpServers` config passed to the Claude Agent SDK `query()` call
- `src/container-runner.ts` — Two changes:
  1. Reads `NOTION_TOKEN`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`,
     `GOOGLE_WORKSPACE_SERVICES` from `.env` and injects them as Docker `-e` flags
  2. Mounts `~/.config/google-workspace-mcp/` into the container at
     `/home/node/.config/google-workspace-mcp/` (writable, so OAuth tokens persist
     across container runs)

**MCP packages used:**
- `@notionhq/notion-mcp-server` — fetched via `npx -y` on first invocation
- `@dguido/google-workspace-mcp` — fetched via `npx -y` on first invocation

**Required env vars:** `NOTION_TOKEN`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`,
`GOOGLE_WORKSPACE_SERVICES` (defaults to `calendar,gmail`)

**Required host files:** `~/.config/google-workspace-mcp/credentials.json` and
`tokens.json` (OAuth flow must be completed on the host before container use)

**Conflict risk on upstream update:** Medium — `container-runner.ts` and
`agent-runner/src/index.ts` both see regular upstream changes. Check both files
after any upstream merge.

---

## CLAUDE.md — Personal Context

**Added:** ongoing

Personal behavioral rules and schema knowledge added to `CLAUDE.md`:
- Notion database schema (Tasks, Notes, Areas) with creation/update rules
- Google Calendar and Gmail usage guidelines
- Behavioral rules (when to create tasks/notes proactively vs. ask first)

**Conflict risk:** Low — upstream rarely touches `CLAUDE.md` content beyond the
Quick Context table. Add new personal rules at the bottom to stay out of the
conflict zone.

---

## Fork Sync CI Workflow

**Source:** Added as part of the discord skill merge
**File:** `.github/workflows/fork-sync-skills.yml`

Keeps skill branches (e.g. `skill/discord`) merged forward as upstream advances.
Runs on a schedule and on pushes to `main`.

**Conflict risk:** Low — upstream does not ship this file.
