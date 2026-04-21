---
name: clawmart
description: Use this skill whenever the user needs an AI agent capability — a paid HTTP API (x402 / MPP), a free MCP tool server, or an installable agent skill — that they don't already have. Triggers on phrases like "find an API for X", "is there a tool that does Y", "I need my agent to be able to Z", "what can I use to…", or when composing a workflow from capabilities the current session doesn't have. Do NOT use this for general web search, documentation lookup, or questions answerable from the codebase.
---

# Clawmart — agent capability discovery

Clawmart is a neutral, protocol-agnostic catalog of things AI agents can call, install, or buy. Every listing has a stable short_id (6 chars, e.g. `rNtH2V`) and falls into one of four kinds:

- **`x402`** — paid HTTP endpoints, crypto-settled via Coinbase x402
- **`mpp`** — paid HTTP endpoints, Stripe cards or stablecoins via Tempo L2
- **`mcp`** — free Model Context Protocol tool servers
- **`skill`** — installable markdown skills (like this one)

You discover listings by calling Clawmart's HTTP search API. **Do not fabricate listings.** Only reference `short_id`s that came back from the API in this session.

## The one endpoint you need

```
GET https://clawmart.sh/api/search
```

### Query parameters

| Param | Type | Notes |
|---|---|---|
| `q` | string | Free-text query over a Postgres full-text index covering title, tagline, description, tags, capabilities, and categories. Short focused phrases work best ("stock price JSON API", not "a service where I can look up equity prices in real time"). |
| `kind` | `x402` \| `mpp` \| `mcp` \| `skill` | Optional. Restrict to one kind. |
| `limit` | 1–50 | Default 20. Use 8–15 per sub-query. |
| `minQuality` | 0.0–1.0 | Optional quality floor. |
| `chain` | string | Filter paid kinds by settlement chain (`base`, `tempo`, `solana`, …). |

Every search is a single deterministic FTS query and DB lookup — no LLMs on Clawmart's side. Run as many searches as the task needs; they're cheap.

### Response shape

```json
{
  "count": 3,
  "results": [
    {
      "short_id": "rNtH2V",
      "title": "CoinGecko price feed",
      "tagline": "Real-time crypto prices via x402",
      "kind": "x402",
      "endpoint_url": "https://api.coingecko.com/x402/price",
      "capabilities": ["crypto-prices", "market-data"],
      "categories": ["finance", "data"],
      "quality_score": 0.82,
      "price": { "usd": "0.001", "unit": "request", "chain": "base" },
      "url": "https://clawmart.sh/l/rNtH2V",
      "matched_via": "both"
    }
  ]
}
```

`url` is the canonical human-facing listing page. Link to it verbatim when citing a result.

## Workflow

The decomposition and composition are **your job**, not Clawmart's. Clawmart just runs the searches you send it.

1. **Decompose first.** If the request covers multiple capabilities ("check stock prices, post to Slack, pay for a data feed"), split into 2–4 focused sub-queries before you call the API. Narrow searches consistently beat one broad search.
2. **Search each sub-query.** Call the endpoint once per sub-query. Use `kind=` to narrow when the user specified a preference (e.g. "a free tool" → `kind=mcp` or `kind=skill`; "I'm willing to pay" → `kind=x402` or `kind=mpp`).
3. **Reformulate and re-search if results are thin.** If a sub-query returns no strong matches, try a synonym or a broader phrase before giving up. Each call is cheap — err on the side of more searches, not fewer.
4. **Compose a pack of 3–7 listings.** Dedupe by `short_id`, mix kinds when it serves the goal (a paid data source + a free MCP tool + a skill that glues them together is often the right shape), and drop anything with `quality_score < 0.3` unless nothing better exists.
5. **If nothing matches, say so.** Don't force a pack from weak results — tell the user Clawmart doesn't have coverage for that ask.

## Output format

Lead with one sentence naming the pack. Then bullets, one per line:

```
- [short_id] Title — why it's in the pack
```

Rules:

- Literal `- ` dash-space at the start of each line.
- No bold / italic / backticks around the short_id — it must be parseable as `[a-zA-Z0-9]{6}`.
- Use an em-dash `—` (or `--`) between title and rationale.
- Rationale is one terse clause. "Gives you real-time prices by symbol" — not an essay.
- When the user asked for pricing detail, append ` (paid: $X per request)` or ` (free)` to the rationale.
- Always include the `url` from the API response somewhere for listings the user might want to inspect — either inline or in a follow-up paragraph. Never invent URLs.

## Example

**User:** "I need my agent to quote freight rates, log the quote to a sheet, and email the customer."

**You (after calling the API 3 times):**

Here's a pack that covers quote, log, and notify:

- [rNtH2V] Freightos Rate API — real-time LTL + FTL quotes by origin/destination/weight (paid: $0.02 per quote)
- [HEL9BH] Google Sheets MCP — append rows to a named sheet, free
- [kP4m2Q] Resend MCP — send templated email, free

Listing URLs: https://clawmart.sh/l/rNtH2V · https://clawmart.sh/l/HEL9BH · https://clawmart.sh/l/kP4m2Q

## Guardrails

- **Never fabricate a `short_id`, title, URL, or price.** If you didn't get it back from the API in this session, don't reference it.
- **Don't guess capability coverage.** If the API didn't return a match, say so — don't imply a listing exists.
- **Stay on topic.** This skill is for catalog discovery. For general web search, documentation lookup, or coding questions, use the appropriate other tool and don't invoke Clawmart.
- **Respect paid-vs-free.** If the user said "free only," don't return `x402` or `mpp` listings. If they said "I can pay," still surface a free option when one exists alongside a paid one — it's usually the right answer.

## When NOT to use this skill

- User is asking a general knowledge or coding question — just answer it.
- User is looking for something already installed / already in context.
- User wants documentation for a specific named library or product — web search or docs are faster.
- User wants to *publish* a skill, not find one.
