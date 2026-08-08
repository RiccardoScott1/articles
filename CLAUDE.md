# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal knowledge base / technical article site ("Dr. Riccardo Scott") built on [Quartz 4](https://quartz.jzhao.xyz/), a static site generator for digital gardens. It is a fork of the upstream Quartz repo with the author's content added in `content/`. The site is also an Obsidian vault (note the `.obsidian/` directory), so articles use Obsidian-flavored markdown including `[[wikilinks]]`.

## Commands

```bash
npm ci                      # install dependencies (Node >= 22 required, see .node-version)
npx quartz build --serve    # dev server with hot reload (http://localhost:8080)
npx quartz build            # production build into public/
npm run check               # typecheck (tsc --noEmit) + prettier --check
npm run format              # prettier --write
npm test                    # runs tsx --test (tests live in quartz/**/*.test.ts)
npx tsx --test quartz/util/path.test.ts   # run a single test file
```

## Deployment

Pushing to the `v4` branch (the default/main branch) triggers `.github/workflows/deploy.yaml`, which builds the site and deploys it to GitHub Pages at `riccardoscott1.github.io/articles`. There is no separate staging — a push to `v4` publishes.

## Architecture

Two clearly separated halves:

- **`content/`** — the articles (markdown + images). This is what changes most often. Organized as one folder per series (e.g. `Geospatial Series/`, `PostgreSQL Full Text Search/`, `RAG on a Web Domain/`, `RecommenderSystems/`), each with an `index.md` landing page and an `images/` subfolder. Files/folders named `private` or `templates` are ignored by the build; drafts (frontmatter `draft: true`) are filtered out by the `RemoveDrafts` plugin.

- **`quartz/`** — the framework (TypeScript/Preact). Mostly upstream Quartz code; only touch it for site behavior changes. The build pipeline is plugin-based and configured entirely in the two root config files:
  - `quartz.config.ts` — site metadata, theme, and the plugin chain: **transformers** (markdown processing: Obsidian/GitHub flavored markdown, KaTeX via `Plugin.Latex`, syntax highlighting, TOC, link resolution with `markdownLinkResolution: "shortest"`) → **filters** (`RemoveDrafts`) → **emitters** (page generation, RSS/sitemap, OG images).
  - `quartz.layout.ts` — which Preact components (from `quartz/components/`) render in each page region (search, explorer, graph view, backlinks, etc.).

When adding or modifying site functionality, the usual flow is: find or write a plugin/component under `quartz/plugins/` or `quartz/components/`, then wire it in via `quartz.config.ts` or `quartz.layout.ts`.

## Obsidian MCP server (`obsidian-vault`)

The vault is accessible to Claude Code through the **Semantic Notes Vault MCP** Obsidian plugin (`semantic-vault-mcp` by [aaronsb](https://github.com/aaronsb)), which runs an MCP server inside Obsidian itself — no external process.

**Install**: Obsidian → Settings → Community plugins → search "Semantic Notes Vault MCP" → install & enable. The plugin lives in `content/.obsidian/plugins/semantic-vault-mcp/` (git-ignored — the settings file contains the API key, never commit it).

**Run**: nothing to start manually. The server runs whenever Obsidian is open with the `content/` vault loaded, serving HTTP on loopback port 3001 (configurable in plugin settings, along with the API key and read-only mode). If MCP calls fail, the usual cause is Obsidian not running.

**Register with Claude Code** (already done for this project in `~/.claude.json`; for a new machine):

```bash
claude mcp add --transport http obsidian http://localhost:3001/mcp \
  --header "Authorization: Bearer <API key from plugin settings>"
```

**Use** — the server exposes action-based tools (`vault`, `graph`, `view`, `edit`, `workflow`, `system`, `bases`):

- `vault`: list/read/create/update/search; search supports `tag:`, `path:`, `content:` operators, quoted phrases, and `/regex/`.
- `graph`: `backlinks`/`forwardlinks`/`neighbors`/`traverse`/`statistics` etc. Note: link counts cover wikilinks only; tags are modeled as file-to-file *edges* (not nodes), included only when passing `followTags: true` or using the `tag-*` actions. For centrality/community analysis (PageRank, Louvain), export edges via a raw `traverse` and compute externally (e.g. networkx).
- Fallback when Obsidian is closed: read/grep `content/` directly.

## Content conventions

- Internal links use Obsidian wikilinks (`[[Page Name]]`), resolved with "shortest path" strategy — file names must be unambiguous across the vault.
- Dates default to git/filesystem modified time unless set in frontmatter (`CreatedModifiedDate` priority: frontmatter → git → filesystem).
- Math is written in LaTeX and rendered with KaTeX.
- Images belong in the series' own `images/` subfolder, not in a global asset directory.
