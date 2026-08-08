# Onshape MCP Server Analysis — ComfyBloom

**Date:** 2026-08-08  
**Author:** Antigravity IDE  
**Purpose:** Evaluate Onshape MCP options for bloom's CAD pipeline (replacing Fusion 360 dependency)  
**Context:** Bloom needs AI-driven CAD for manufacturing parts (centrifuge, clinic equipment, etc.) — currently Fusion 360 is explored but requires a desktop app install. Onshape is cloud-native and may work better for headless AI-driven CAD.

---

## TL;DR — Recommendation

> **For bloom, go with `jarvis-onshape-mcp` (ReshefElisha).** It's the most battle-tested for AI-driven CAD, has the richest agent-facing features (vision, self-correction, hints), and it works on the **free plan** for prototyping. Upgrade to paid Onshape only when designs need to be private/commercial.

---

## Repos Downloaded

All cloned to `research/1-MCPCAD/`:

| Repo | Folder | Language | Stars | Forks | Last Commit |
|------|--------|----------|-------|-------|-------------|
| hedless/onshape-mcp | `hedless-onshape-mcp/` | Python | 126 | 64 | 2026-03-04 |
| altendky/onshape-mcp | `altendky-onshape-mcp/` | Rust | 14 | 4 | 2026-08-07 |
| Casys-AI/mcp-onshape | `casys-ai-mcp-onshape/` | TypeScript/Deno | 0 | 0 | 2026-08-08 |
| ReshefElisha/jarvis-onshape-mcp | `jarvis-onshape-mcp/` | Python | 153 | 31 | 2026-04-22 |

---

## 1. hedless/onshape-mcp

### Works with Onshape Free Plan?
**Yes.** No plan restrictions in code or docs. Uses API keys from the free developer portal.

### Features (45 tools)
- Document Management: List, search, create documents, find Part Studios
- Sketching: Rectangle, circle, line, arc on standard planes
- Features: Extrude, revolve, thicken, fillet, chamfer, boolean, linear/circular pattern
- Assembly: Create assemblies, add instances, 4 mate types, face alignment
- Assembly Analysis: Interference checking, position verification, body details
- Variables: Read/write variable tables for parametric designs
- FeatureScript: Eval expressions, bounding boxes
- Export: STL, STEP, Parasolid, GLTF, OBJ

### Strengths
- Well-documented, clean Python codebase
- Most forked (64 forks) — proven community adoption
- Simple setup: `pip install -e .` + env vars

### Weaknesses
- Not actively maintained (last commit Mar 2026)
- No vision/rendering — AI can't "see" what it built
- No self-correction hints
- Basic error handling

### Verdict for Bloom
Decent foundation but lacks AI-agent features. Better options exist that forked from this.

---

## 2. altendky/onshape-mcp

### Works with Onshape Free Plan?
**Yes.** Supports both OAuth and API key auth. Works with any plan.

### Features (Dynamic — full API coverage)
- Dynamic API Discovery: Embeds entire Onshape OpenAPI spec
- All Onshape API endpoints accessible
- Screenshot/Visualization for visual verification
- OAuth 2.0 flow with local callback server
- File I/O for exports and FeatureScript
- npx install: `npx --yes onshape-mcp`

### Strengths
- Most actively maintained (last commit yesterday, 674+ PRs)
- Dynamic API = future-proof
- Clean Rust implementation
- Screenshot capability
- HTTP transport for remote setups
- Apache 2.0 license

### Weaknesses
- Early development — author says "things may not always work"
- Low adoption (14 stars)
- Dynamic API = more token usage, more error-prone
- No CAD-specific agent logic
- Rust toolchain needed for dev

### Verdict for Bloom
Too raw for bloom. Dynamic approach means more token usage and more room for AI mistakes.

---

## 3. Casys-AI/mcp-onshape

### Works with Onshape Free Plan?
**No — explicitly requires paid plan.** README states: "This server needs a paid Onshape plan."

### Features (100 tools across 14 categories)
- Documents (12): Full CRUD + search + sharing + permissions + history
- Versions/Workspaces (8): Branching, merging, merge preview
- Part Studios (14): Features, body details, mass props, shaded views, compare/diff, rollback
- Parts (7): Per-part details, mass, shaded views, bend tables
- Assemblies (14): Full tree, instances, mates, BOM, exploded views
- Drawings (5): Create, views, geometry, modify, export (PDF/DXF/DWG)
- Export/Import (10): STL, STEP, GLTF, OBJ, Parasolid, SOLIDWORKS + import
- Configurations/Variables (6), Metadata (5), Releases/Revisions (6)
- Thumbnails (3), Comments (4), Users/Teams (3), Webhooks (3)
- 4 Interactive UI Viewers: 3D (three.js), mass, BOM, doc list

### Strengths
- Most comprehensive — 100 tools covering entire Onshape ecosystem
- PDM features (releases, revisions) — enterprise-grade
- Only MCP with drawing creation/export
- Category filtering, UI viewers
- Both stdio and HTTP modes

### Weaknesses
- Brand new (9 days old), 0 stars, 0 forks
- Requires paid Onshape
- No agent-specific features (no hints, no vision decomposition)
- Deno dependency

### Verdict for Bloom
Most feature-rich but paid plan + zero community validation. Best for Phase 2 when bloom upgrades to commercial Onshape.

---

## 4. ReshefElisha/jarvis-onshape-mcp — RECOMMENDED

### Works with Onshape Free Plan?
**Yes.** Uses standard API keys. Free plan limits apply (2,500 calls/year, public docs) but MCP works.

### Features (~60 tools)
- Everything from hedless (fork) + major additions:
- Truth-telling mutations: `{ok, status, feature_id, error_message, changes, hints}`
- Vision/Rendering: Multi-view PNG renders, image cropping, reference comparison
- Vision decomposition skill: Structured workflow to read drawings before building
- Drawing OCR: `extract_drawing_dimensions` via Tesseract
- Entity discovery: Deterministic face IDs, surface types, outward normals
- Per-feature geometric diffs: bbox/part count/mass deltas after each feature
- Parametric Variables: First-class Variable Studios, upsert-by-name
- FeatureScript escape hatch: Write custom features (helices, threads, shells)
- Hints rotation: Auto-suggest fixes for known failure patterns
- Assembly: 4 mate types + interference checks
- Claude Code plugin: `/plugin install github:ReshefElisha/jarvis-onshape-mcp`

### Strengths
- Most starred (153) — highest community adoption
- Purpose-built for AI agents — optimizes for LLM success
- Self-correcting hints = fewer wasted iterations
- Vision = AI can SEE what it's building
- Drawing OCR = can read engineering specs
- Claude Code plugin ready
- Knowledge base with CAD workflow guides
- MIT license

### Weaknesses
- Last commit April 2026 (4 months stale, but feature-complete)
- Claude Code specific — needs adaptation for other MCP clients
- No drawing creation (only OCR)
- No PDM features
- Depends on `uv` on PATH

### Verdict for Bloom
**Best option.** AI-agent-first design (vision, hints, self-correction, geometric diffs) is exactly what bloom needs.

---

## Comparison Matrix

| Feature | hedless | altendky | Casys-AI | jarvis |
|---------|---------|----------|----------|--------|
| Free Plan | Yes | Yes | No | Yes |
| Tool Count | 45 | Dynamic | 100 | ~60 |
| AI Vision | No | Screenshots | Shaded views | Multi-view+OCR+compare |
| Self-correction | No | No | No | Hints rotation |
| Geometric diffs | No | No | No | Per-feature |
| FeatureScript | Eval only | Via API | Eval | Eval + write custom |
| Assembly | Yes | Yes | Full | Yes + interference |
| Drawings | No | No | Create/export | OCR only |
| Export formats | 5 formats | Same via API | 7+ formats + import | 3 formats |
| PDM (releases) | No | No | Yes | No |
| Plugin install | Manual | npx | Deno | /plugin install |
| Language | Python | Rust | TypeScript | Python |
| Maintenance | Stale | Active | Brand new | Stable |
| Stars | 126 | 14 | 0 | 153 |

---

## Onshape Free Plan Limitations

1. **2,500 API calls/year** — enough for prototyping, not production
2. **All documents PUBLIC** — anyone can see designs
3. **Non-commercial use only** — can't use for profit
4. **Rate limits** — per-minute/per-day caps

For prototyping: free plan is fine. For manufacturing: upgrade to Standard ($1,500/yr) or Professional ($2,500/yr).

---

## Recommendation for Bloom

### Phase 1 — Now (Prototyping)
Use `jarvis-onshape-mcp` on Onshape free plan.
- Claude Code: `/plugin install github:ReshefElisha/jarvis-onshape-mcp`
- Antigravity/Codex: adapt MCP config to use `uv run onshape-mcp`
- Get API keys from dev-portal.onshape.com

### Phase 2 — Manufacturing
When designs go private/commercial:
1. Upgrade Onshape to Standard plan
2. Add `Casys-AI/mcp-onshape` for PDM features (releases, BOMs)
3. Or wait for altendky to mature (architecturally superior long-term)

### Why Onshape over Fusion 360?
- Cloud-native — no desktop app required
- Headless automation — any machine, any agent
- Better API surface for programmatic CAD
- No add-in install — just API keys
