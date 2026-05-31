# README Rewrite Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the outdated README and create a `.env.example` so developers can self-host the server with minimal friction.

**Architecture:** Two file changes only — full rewrite of `README.md` using the approved spec, and creation of `.env.example` with all 6 configuration variables commented. No code changes.

**Tech Stack:** Markdown, shell env variables.

**Spec:** `docs/superpowers/specs/2026-05-31-readme-design.md`

---

## File Map

| Action | File |
|--------|------|
| Create | `.env.example` |
| Rewrite | `README.md` |

---

## Task 1: Create .env.example

**Files:**
- Create: `.env.example`

- [ ] **Step 1: Create the file**

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

- [ ] **Step 2: Verify .env.example is not gitignored**

Run:
```bash
git check-ignore -v .env.example
```

Expected: no output (not ignored). If it IS ignored, open `.gitignore` and confirm only `.env` (not `.env.example`) is listed.

- [ ] **Step 3: Commit**

```bash
git add .env.example
git commit -m "chore: add .env.example with all configuration variables"
```

---

## Task 2: Rewrite README.md

**Files:**
- Modify: `README.md` (full replacement)

- [ ] **Step 1: Replace README.md with the new content**

Write the following as the complete content of `README.md`:

````markdown
# obsidian-web-mcp

Your Obsidian vault, accessible from Claude everywhere.

This MCP server runs on the machine where your vault lives and exposes it securely over HTTPS — so Claude on the web, desktop, and mobile can all read and write your notes, not just Claude Code on your local machine.

## What this fork adds

The original project is a solid foundation. This fork extends it with:

- **Full OAuth 2.0 support** — works with claude.ai's "Add custom connector" flow out of the box. No API keys to paste, no manual token management.
- **Tunnel-agnostic deployment** — one env variable (`VAULT_MCP_ALLOWED_HOSTS`) is all it takes to work with Tailscale Funnel, Cloudflare Tunnel, or any reverse proxy.
- **Surgical file editing** — two new tools (`vault_patch`, `vault_append`) let Claude edit a section of a large file without rewriting the whole thing.

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

## Quick Start

### Requirements

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- An Obsidian vault (any folder of markdown files)

### Run locally

```bash
git clone https://github.com/marcelotoledo/obsidian-web-mcp.git
cd obsidian-web-mcp

cp .env.example .env
# Edit .env: set VAULT_PATH and generate tokens (see comments inside)

uv run vault-mcp
```

The server starts on port 8420. That's it.

## Connecting to Claude

There are three ways to connect, depending on how you're using Claude:

**Option A — claude.ai web and mobile (recommended)**  
Go to **Settings → Integrations → Add custom connector** and enter your server's public URL. Claude handles the OAuth flow automatically.

**Option B — Claude Code with a static token**

```bash
claude mcp add --scope user --transport http obsidian-vault \
  https://your-server-url \
  --header "Authorization: Bearer YOUR_VAULT_MCP_TOKEN"
```

**Option C — Claude Desktop (legacy)**  
Add an `npx mcp-remote` block to `claude_desktop_config.json`.  
⚠️ Don't use Option A and Option C at the same time — they conflict.

## Remote Access

The server runs locally and needs a secure tunnel to be reachable from the internet. Two options:

**Tailscale Funnel** (easier — no domain required)

```bash
tailscale funnel 8420
```

Then set `VAULT_MCP_ALLOWED_HOSTS=your-machine.tailnet.ts.net` in your `.env`.

**Cloudflare Tunnel** (if you already have a domain on Cloudflare)

```bash
brew install cloudflare/cloudflare/cloudflared
./scripts/setup-tunnel.sh
```

Then set `VAULT_MCP_ALLOWED_HOSTS=vault-mcp.yourdomain.com` in your `.env`.

Both work the same way from Claude's perspective.

## Production (macOS)

To run the server automatically at login and keep it alive:

```bash
cp scripts/launchd/com.example.vault-mcp.plist ~/Library/LaunchAgents/
# Edit the plist to fill in your paths and tokens
launchctl load ~/Library/LaunchAgents/com.example.vault-mcp.plist
```

Logs go to `~/Library/Logs/vault-mcp.log` and `~/Library/Logs/vault-mcp-error.log`.

## Development

```bash
uv run pytest tests/ -v   # all tests use temp dirs, never touch your real vault
```

## License

MIT — see [LICENSE](LICENSE).
````

- [ ] **Step 2: Verify the file looks right**

Run:
```bash
wc -l README.md
```

Expected: roughly 90–110 lines (much shorter than the original 237).

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: rewrite README — concise, English, highlights fork improvements"
```

---

## Self-Review

**Spec coverage:**
- Opening tagline + description ✓ (Task 2)
- "What this fork adds" section with 3 bullets ✓ (Task 2)
- Tools table with ✦ markers ✓ (Task 2)
- Quick Start with `.env.example` reference ✓ (Tasks 1 + 2)
- Connecting to Claude — Options A, B, C ✓ (Task 2)
- Remote Access — Tailscale + Cloudflare ✓ (Task 2)
- Production (macOS) with launchd ✓ (Task 2)
- Development + License ✓ (Task 2)
- `.env.example` with all 6 variables ✓ (Task 1)

**Placeholder scan:** No TBDs. All content is complete and final.

**Consistency:** `.env.example` variables match `config.py` exactly (`VAULT_PATH`, `VAULT_MCP_TOKEN`, `VAULT_OAUTH_CLIENT_SECRET`, `VAULT_MCP_PORT`, `VAULT_OAUTH_CLIENT_ID`, `VAULT_MCP_ALLOWED_HOSTS`).
