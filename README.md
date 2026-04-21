# Clawmart skill

The official [Clawmart](https://clawmart.sh) discovery skill for AI agents.

Teaches your agent to find paid APIs, free MCP tool servers, and installable skills in the Clawmart catalog by hitting a single deterministic search endpoint — **no LLMs on Clawmart's side**. The agent does the query decomposition and composition; Clawmart just runs Postgres full-text search and returns matches.

## Install

```bash
npx skills add christianlovescode/clawmart-skill
```

Or pick a specific agent / scope:

```bash
npx skills add christianlovescode/clawmart-skill -a claude-code -g
```

Works with Claude Code, Cursor, Codex, OpenCode, and 40+ other harnesses supported by [`skills`](https://skills.sh).

## What your agent gets

When invoked, the skill teaches your agent to:

1. **Decompose** a multi-capability request into 2–4 focused sub-queries.
2. **Search** each sub-query against `GET https://clawmart.sh/api/search` (free, unauthenticated, cacheable).
3. **Compose** a pack of 3–7 complementary listings across the four listing kinds:
   - `x402` — paid HTTP endpoints (crypto-settled)
   - `mpp`  — paid HTTP endpoints (Stripe cards + stablecoins)
   - `mcp`  — free Model Context Protocol tool servers
   - `skill` — installable markdown skills
4. **Link** each pick to its listing page at `https://clawmart.sh/l/<short_id>`.

## Why a skill, not an MCP server

Clawmart's catalog is a plain HTTP API. There's no state, no auth, no long-running connection — so an MCP server would add a process without adding capability. A skill tells the agent *when* and *how* to hit the API, which is the only useful layer.

## License

MIT
