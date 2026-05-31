# README Rewrite — Design Spec

**Date:** 2026-05-31  
**Goal:** Replace the outdated README with a concise, persuasive document aimed at developers who want to self-host the server. English only. Easy to understand for a non-technical reader. Honest — no exaggeration.

---

## Context

This repo is a fork of `jimprosser/obsidian-web-mcp`. Since the fork, it has accumulated meaningful improvements: full MCP spec 2025-06-18 compliance, complete OAuth 2.0 support (DCR + PKCE), tunnel-agnostic deployment, and two new tools (`vault_patch`, `vault_append`). The current README still references Cloudflare Tunnel throughout and doesn't mention any of these improvements.

## Audience

Developers who find the repo on GitHub and want to self-host. They are deciding whether to use this fork or the original. The README needs to answer "why this one?" in the first 10 seconds.

## Tone

- English
- Direct and clear — no jargon, no filler
- Persuasive without overpromising
- Show the differentials plainly; let them speak for themselves

---

## Structure

### 1. Opening

One-line tagline + two-sentence description. Answers "what is this and why does it exist?" immediately.

```
# obsidian-web-mcp

Your Obsidian vault, accessible from Claude everywhere.

This MCP server runs on the machine where your vault lives and exposes
it securely over HTTPS — so Claude on the web, desktop, and mobile can
all read and write your notes, not just Claude Code on your local machine.
```

### 2. What this fork adds

Three bullets, one per key improvement. Positions the fork clearly against the upstream.

```
## What this fork adds

The original project is a solid foundation. This fork extends it with:

- **Full OAuth 2.0 support** — works with claude.ai's "Add custom connector"
  flow out of the box. No API keys to paste, no manual token management.
- **Tunnel-agnostic deployment** — one env variable (`VAULT_MCP_ALLOWED_HOSTS`)
  is all it takes to work with Tailscale Funnel, Cloudflare Tunnel, or any
  reverse proxy.
- **Surgical file editing** — two new tools (`vault_patch`, `vault_append`)
  let Claude edit a section of a large file without rewriting the whole thing.
```

### 3. Tools

Full table of all 11 tools. New tools marked with ✦ to distinguish fork additions.

```
## Tools

| Tool | What it does |
|------|-------------|
| `vault_read` | Read a file — content, metadata, and parsed frontmatter |
| `vault_batch_read` | Read multiple files in one call |
| `vault_write` | Write or overwrite a file; optionally merge frontmatter |
| `vault_patch` | ✦ Replace a specific string in a file without rewriting it |
| `vault_append` | ✦ Add content to the end of a file |
| `vault_batch_frontmatter_update` | Update frontmatter fields on multiple files at once |
| `vault_search` | Full-text search across all vault files |
| `vault_search_frontmatter` | Search files by frontmatter field value |
| `vault_list` | List files and folders with optional depth and glob filtering |
| `vault_move` | Move or rename a file or directory |
| `vault_delete` | Soft-delete a file (moves to `.trash/`, not permanent) |

✦ Added in this fork
```

### 4. Quick Start

Three steps: clone, configure via `.env`, run. References `.env.example` which must be created as part of this implementation.

```
## Quick Start

### Requirements

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- An Obsidian vault (any folder of markdown files)

### Run locally

git clone https://github.com/marcelotoledo/obsidian-web-mcp.git
cd obsidian-web-mcp

cp .env.example .env
# Edit .env: set VAULT_PATH and generate tokens (see comments inside)

uv run vault-mcp

The server starts on port 8420. That's it.
```

### 5. Connecting to Claude

Three options documented. Warning about not using A + C simultaneously.

```
## Connecting to Claude

**Option A — claude.ai web and mobile (recommended)**
Go to Settings → Integrations → Add custom connector and enter your
server's public URL. Claude handles the OAuth flow automatically.

**Option B — Claude Code with a static token**
claude mcp add --scope user --transport http obsidian-vault \
  https://your-server-url \
  --header "Authorization: Bearer YOUR_VAULT_MCP_TOKEN"

**Option C — Claude Desktop (legacy)**
Add an `npx mcp-remote` block to `claude_desktop_config.json`.
⚠️ Don't use Option A and Option C at the same time — they conflict.
```

### 6. Remote Access

Two tunnel options presented as equals. `VAULT_MCP_ALLOWED_HOSTS` is the single config point for both.

```
## Remote Access

The server runs locally and needs a secure tunnel to be reachable from
the internet. Two options:

**Tailscale Funnel** (easier — no domain required)
tailscale funnel 8420
Then set VAULT_MCP_ALLOWED_HOSTS=your-machine.tailnet.ts.net in your .env.

**Cloudflare Tunnel** (if you already have a domain on Cloudflare)
brew install cloudflare/cloudflare/cloudflared
./scripts/setup-tunnel.sh
Then set VAULT_MCP_ALLOWED_HOSTS=vault-mcp.yourdomain.com in your .env.

Both work the same way from Claude's perspective.
```

### 7. Production (macOS)

launchd setup in three commands. Log paths stated explicitly.

```
## Production (macOS)

To run the server automatically at login and keep it alive:

cp scripts/launchd/com.example.vault-mcp.plist ~/Library/LaunchAgents/
# Edit the plist to fill in your paths and tokens
launchctl load ~/Library/LaunchAgents/com.example.vault-mcp.plist

Logs go to ~/Library/Logs/vault-mcp.log and vault-mcp-error.log.
```

### 8. Development + License

Minimal.

```
## Development

uv run pytest tests/ -v   # all tests use temp dirs, never touch your real vault

## License

MIT
```

---

## Artifacts to create / modify

| File | Action | Notes |
|------|--------|-------|
| `README.md` | Rewrite | Full replacement per this spec |
| `.env.example` | Create | All 6 config variables with inline comments and token generation instructions |

## .env.example content

```
# Absolute path to your Obsidian vault directory
VAULT_PATH=~/Obsidian/MyVault

# Bearer token for MCP requests — generate with:
# python -c "import secrets; print(secrets.token_hex(32))"
VAULT_MCP_TOKEN=

# OAuth 2.0 client secret — generate with the same command
VAULT_OAUTH_CLIENT_SECRET=

# Port the server listens on (default: 8420)
# VAULT_MCP_PORT=8420

# OAuth client ID (default: vault-mcp-client)
# VAULT_OAUTH_CLIENT_ID=vault-mcp-client

# Hostnames accepted by DNS rebinding protection.
# Add your tunnel hostname here (comma-separated if multiple).
# Examples:
#   Tailscale: your-machine.tailnet.ts.net
#   Cloudflare: vault-mcp.yourdomain.com
# VAULT_MCP_ALLOWED_HOSTS=
```

---

## What this spec does NOT include

- Architecture diagram (removed — adds complexity, not value for this audience)
- Security model section (removed — good content but leigo audience doesn't need it upfront; security is implied by OAuth + atomic writes mentioned elsewhere)
- Project structure tree (removed — developers who care will read the code)
- Obsidian Sync compatibility section (removed — covered implicitly by "atomic writes" in the write tool description)
