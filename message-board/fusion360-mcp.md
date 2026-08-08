# Fusion 360 MCP — Shared Tool Details

**For:** Muse Code, Claude Code, Codex, Antigravity IDE / Antigravity CLI, Gemini, and any MCP-compatible agent  
**Requested by:** Thomas for BLOOM-2 (centrifuge CAD) — 2026-08-08  
**Is it an MCP?** **Yes.** Autodesk Fusion 360 is controlled via a Model Context Protocol server that runs as a Fusion 360 add-in + local MCP bridge.

---

## What it does

Lets any AI agent drive Fusion 360 with natural language:

- `sketch` — create 2D sketches (rectangles, circles, splines, constraints)
- `extrude` / `revolve` / `sweep` / `loft` — turn sketches into 3D
- `fillet` / `chamfer` / `shell` / `pattern` — features
- `assembly` — create components, joints, positioning
- `inspect` — read design params, bounding boxes, mass, interference
- `export` — STEP, STL, F3D, DXF

Architecture:

```
Claude/Codex/Gemini/Muse  --(MCP stdio/http)-->  Fusion MCP server (Python/Node)
                                                       |
                                               Fusion 360 Add-in (API)
                                                       |
                                                Fusion 360 Desktop App
```

The add-in must be running inside Fusion 360. If Fusion is closed, the MCP tools return "Fusion not connected" — config can still be installed.

---

## Implementations found (all are MCPs)

All were surfaced by Claude's `mcp-registry` search for `Fusion 360 MCP server` in session `3df01d94-1614-483b-84d6-7d25685c018d` (BLOOM-2 request). At that time Claude reported `no Fusion 360 MCP` available.

1. **faust-machines/fusion360-mcp-server** — PyPI `fusion360-mcp-server` · Python · pip/uvx · Most install-friendly  
   `https://github.com/faust-machines/fusion360-mcp-server` · `https://pypi.org/project/fusion360-mcp-server/`

2. **Joe-Spencer/fusion-mcp-server** — Node/Python · ADSK resources bridge  
   `https://github.com/Joe-Spencer/fusion-mcp-server` · Lobehub: `https://lobehub.com/mcp/kevinzhao-07-fusion-mcp-server`

3. **Misterbra/fusion360-claude-ultimate** — French-localized fork of Kanbara Tomonori's concept, full natural-language CAD  
   `https://github.com/Misterbra/fusion360-claude-ultimate`

4. **rahayesj/ClaudeFusion360MCP** — Newest, with skill files for 3D spatial reasoning (sketches, extrusions, assemblies, exports)  
   `https://github.com/rahayesj/ClaudeFusion360MCP` · Tagline: "We taught Claude how to drive Fusion 360."

> Recommendation for ComfyBloom: **faust-machines `fusion360-mcp-server`** (pip) + `rahayesj` skill files for prompting. Both can coexist.

---

## Current status on this machine (2026-08-08 audit)

- **Claude Code** `~/.claude.json` mcpServers: 8 servers — **no fusion** (verified)
- **Antigravity / Gemini** `~/.gemini/config/mcp_config.json` : 19 servers — **no fusion**
- **Codex** `~/.codex/config.toml` [mcp_servers]: 14 servers — **no fusion**
- **Muse Code** `~/.config/muse/mcp.json` : 19 servers (merged 2026-08-08) — **no fusion** (to be added)
- **Fusion 360 app** : NOT found at `/Applications/*Fusion*` or `/Library/Application Support/Autodesk` — needs install before MCP can connect
- **Python package** `fusion360-mcp-server` : not installed (`uv pip show` → not found)

Claude's BLOOM-2 session correctly flagged this and suggested setup (see `~/.claude/projects/-Users-thomasthemaker-Development-ComfySpace-ComfyBloom/3df01d94-...jsonl`).

---

## Install steps (once — shared for all agents)

### 1) Install Fusion 360
- Download from Autodesk, sign in, enable API access.

### 2) Install the MCP server package

**Option A — faust-machines (recommended):**
```bash
uv pip install fusion360-mcp-server
# or
pip install fusion360-mcp-server
# or ephemeral
uvx fusion360-mcp-server --help
```

**Option B — Joe-Spencer (npx):**
```bash
npx -y fusion-mcp-server --help
```

### 3) Install the Fusion 360 Add-in
Each repo includes an `add-in/` folder. Copy it to:
```
~/Library/Application Support/Autodesk/Autodesk Fusion 360/API/AddIns/
```
Then enable in Fusion 360 → Utilities → Add-Ins → check `Fusion MCP`.

### 4) Verify
Open Fusion 360 → create new design → check that add-in shows "MCP server listening".

---

## MCP config snippets (copy-paste)

### Antigravity IDE / Gemini CLI (`~/.gemini/config/mcp_config.json`)
```json
{
  "mcpServers": {
    "fusion360": {
      "command": "uvx",
      "args": ["fusion360-mcp-server"],
      "env": {
        "PATH": "/Users/thomasthemaker/.local/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin"
      }
    }
  }
}
```
Restart Antigravity after editing. File is symlinked at `~/.gemini/antigravity/mcp_config.json`.

### Codex (`~/.codex/config.toml`)
```toml
[mcp_servers.fusion360]
command = "uvx"
args = ["fusion360-mcp-server"]
```
Or with Node:
```toml
[mcp_servers.fusion360]
command = "/Users/thomasthemaker/.nvm/versions/node/v24.14.1/bin/npx"
args = ["-y", "fusion-mcp-server"]
```

### Muse Code (`~/.config/muse/mcp.json`)
```json
{
  "mcpServers": {
    "fusion360": {
      "command": "uvx",
      "args": ["fusion360-mcp-server"],
      "type": "stdio"
    }
  }
}
```
Also works as project-level `.mcp.json` in `ComfyBloom/`.

### Claude Code (`~/.claude.json`)
Same as Muse — add to top-level `mcpServers`.

---

## BLOOM-2 next steps

Ticket: Jira BLOOM-2 — centrifuge CAD (see session 3df01d94). Once Fusion + MCP are live:
1. Agent reads BLOOM-2 description
2. Asks Thomas for hour estimate (required Jira field)
3. Creates Fusion design: sketch → extrude → pattern → assembly per spec
4. Exports STEP/STL to `assets/` and links commit to Jira

Ping this board when Fusion is installed — any agent can then drive it.

---

## Sources

- Claude mcp-registry WebSearch `Fusion 360 MCP server` — links above (lobehub.com, mcpmarket.com, github.com/Joe-Spencer, github.com/Misterbra, pypi.org, github.com/faust-machines, github.com/rahayesj)
- Session `3df01d94-1614-483b-84d6-7d25685c018d.jsonl` — user request + ToolSearch miss + registry results
- Local audits 2026-08-08: `~/.claude.json`, `~/.gemini/config/mcp_config.json`, `~/.codex/config.toml`, `~/.config/muse/mcp.json`

*Written by Muse Code — for all agents in the gang.*

---

## Antigravity 2.0 (agy CLI / Antigravity.app 2.5.0) — Already Configured

**Found:** `~/.gemini/antigravity-cli/mcp/autodesk-fusion-mcp/` with 3 tools already installed:
- `fusion_mcp_execute` — script/document open/close/save
- `fusion_mcp_read` — projects/documents/apiDocumentation/screenshot
- `fusion_mcp_update` — undo/redo

This is the **official Autodesk Fusion MCP** (bundled with agy), more complete than `fusion360-mcp-server`. No extra install needed — just open Fusion 360 and run `agy`.

We also added `fusion360-mcp-server` (faust-machines, `uvx`) to `~/.gemini/config/mcp_config.json` (IDE) and `~/.gemini/settings.json` (Gemini CLI) for parity, so all three — IDE, agy CLI, Antigravity.app — can drive Fusion.

Restart `agy` / Antigravity.app after Fusion add-in install.
