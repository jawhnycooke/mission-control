---
name: research
description: Deep-dive a stock or ETF in the portfolio. Pulls fundamentals and recent news from the Bigdata.com MCP, anchors the read to Jawhny's investing philosophy, and writes a thesis doc into investments/outputs/research/{ticker}-YYYY-MM-DD.md. The dashboard links the new note on its next regeneration. Use when the user says "/research NVDA", "/research VOO", "deep dive AIQ", or asks for a thesis on any ticker. Reads from the Bigdata.com MCP; writes only to investments/outputs/research/.
---

# `/research [ticker]`

Equity research on demand. Reads Bigdata.com, writes a markdown thesis. No buy/sell recommendation — the note informs, Jawhny decides.

## Argument parsing

- `/research <TICKER>` → run the workflow for `<TICKER>`. Normalize to uppercase.
- `/research` with no args → print usage: `Usage: /research <TICKER>`.
- Multiple tickers in one call → run them sequentially, one file per ticker.

## Resolution

1. Read `investments/inputs/holdings.csv` to confirm whether the ticker is in the portfolio. If yes, surface that fact in the note. If no, still run, but lead with `NOT IN PORTFOLIO` so context is honest.
2. Resolve the ticker via `mcp__bigdata_search.find_securities`. Prefer the US-listed security (`XNAS:` or `ARCX:`). For ETFs, pass `security_types=["ETF"]` to disambiguate (AIQ resolves to the Global X AI & Technology ETF, not the UK-listed AIQ Ltd.).
3. Capture the `rp_entity_id` and `security_type` (COMPANY vs ETF) from the result.

## Reads

1. `investments/inputs/holdings.csv` — portfolio membership check.
2. `investments/data/holdings.json` — current price, day change, position size, P&L (if held).
3. `memory/investments/philosophy.md` — Jawhny's investing posture. **Anchor every conclusion to this file.** If the file is empty or skeleton-only, surface that and continue.
4. Bigdata.com MCP:
   - `bigdata_company_tearsheet` with `rp_entity_id`, `company_type="Public"`, `sections` chosen by what's available (start broad if it's a name unfamiliar to you; narrow with `["financial_statements","key_metrics","analyst_ratings"]` for known names).
   - `bigdata_search` in **fast mode** with explicit entity filter (`any_of: [rp_entity_id]`) and `document_type` filter (INCLUDE: `NEWS`, `TRANSCRIPT`, `FILING`, `INVESTMENT-RESEARCH` as relevant). Smart mode can pick the wrong entity for ambiguous tickers — always use fast here.

## Writes

`investments/outputs/research/{TICKER}-{YYYY-MM-DD}.md`. If a file already exists for today and the ticker, append a `## Update {HH:MM PT}` section rather than overwriting.

The dashboard's research section reads the directory on next regeneration of `investments/outputs/dashboard.html` (either Block 2's daily refresh or an ad-hoc `python investments/scripts/render_dashboard.py`).

## Output format

```markdown
# {TICKER} — {Company / Fund Name}

**As of:** {YYYY-MM-DD HH:MM PT}
**Position in portfolio:** {shares × current_price = $value, unrealized P&L} or `NOT IN PORTFOLIO`
**Bigdata entity:** {rp_entity_id} · {security_type} · {primary listing}

## Snapshot
{Price, day change, 1Y change, 52W range, market cap. One paragraph or a short table.}

## Thesis (FACTS)
{Numbers and verifiable statements from the tearsheet and filings. Cite each.}

## Read-through (ANALYSIS)
{What the facts imply. Connect to Jawhny's philosophy in memory/investments/philosophy.md.
Lead with the strongest read; bury caveats only after the primary point.}

## Risks (FACTS + ANALYSIS)
{Concrete risks from filings, analyst notes, or guidance changes. No hand-waving.}

## Speculation
{Anything beyond what the data supports. Label it. Keep it short.}

## What would change the read
{The next 1-3 data points that would force a re-underwrite. Specific catalysts,
specific thresholds, with dates if known.}

## Sources
- Bigdata.com — [Tearsheet for {Company}]({url})
- {Source name} — [{Headline}]({url}) ({date})
- ...
```

**Distinguish facts, analysis, and speculation.** Mix them and the note loses value. Cite every fact with an inline source. The Sources section at the bottom is the structured re-list — not a substitute for inline citations.

**No buy/sell recommendation.** The note ends at "what would change the read." Jawhny decides.

## Voice

Match `investments/CLAUDE.md`: rigorous analyst, no fluff, never round or hand-wave numbers, no em dashes. Be confident in the read; be honest about what's speculation.

## File references

- Spec: `spec/mission-control-spec.md` §4.5
- Philosophy anchor: `memory/investments/philosophy.md`
- Current portfolio: `investments/inputs/holdings.csv`, `investments/data/holdings.json`
- Output location: `investments/outputs/research/{TICKER}-{YYYY-MM-DD}.md`
- Domain voice: `investments/CLAUDE.md`
- Bigdata branding: Always "Bigdata.com" with link to https://bigdata.com when referenced in output.
