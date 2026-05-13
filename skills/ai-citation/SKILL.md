---
name: ai-citation
description: Use when a user asks to audit, improve, or grow their likelihood of being cited by AI search engines (ChatGPT, Perplexity, Claude, Gemini, Google AI Overviews). Triggers on phrases like "audit our AI citation", "get cited by ChatGPT", "GEO audit", "generative engine optimization", "AI visibility", "raise our AI citation score", "citability check on <url>", "AI citation walkthrough", "monthly AI citation check". Use also when a user asks to score a single page for AI citability, run a brand-wide AI search visibility audit, walk through growth-hack moves to improve citation likelihood, or compare audits over time.
---

# AI Citation Audit and Growth

A self-contained skill that audits a brand's likelihood of being cited by AI search engines, walks the team through prioritized growth-hack moves to fix gaps, and tracks improvements over monthly delta runs.

The skill is grounded in research:

- **Cyrus Shepard's** May 2026 analysis of 54 AI citation experiments → drives the **Readiness Score**
- **Princeton / Georgia Tech / IIT Delhi** GEO research (2024) → drives the **Action Score**
- **Discovered Labs** early-2026 platform-citation data → drives the **Per-Platform Snapshot** projections

It produces a hybrid two-score model:

```
Composite AI Citation Score = 0.5 × Readiness Score + 0.5 × Action Score
```

Plus a Brand Surface Area Map (off-site presence on the platforms AI engines pull from) and per-engine projection gauges (ChatGPT / Perplexity / Gemini / AIO / Claude).

## When to use

**Use this skill when the user says:**

- "audit our AI citation" / "audit my AI citation" / "AI citation audit"
- "get cited by ChatGPT / Perplexity / Claude / Gemini / AI"
- "raise our AI citation score" / "improve our GEO score"
- "GEO audit" / "generative engine optimization audit" (only if AI/LLM context — see anti-trigger below)
- "AI visibility audit" / "AI search visibility check"
- "score this page for AI citation" / "is this page citable by AI"
- "citability check on <url>" / "how citable is <url>"
- "AI citation delta" / "compare our AI citation" / "monthly AI citation check"
- "AI citation walkthrough" / "growth-hack my AI citation" / "walk me through GEO fixes"

**Do NOT use when:**

- The user is asking pure traditional SEO questions ("how do I rank for X keyword on Google") unless they also mention AI / LLM / GEO / ChatGPT / Perplexity
- The user is asking a conceptual question ("what is GEO?") — answer directly, do not run the audit
- The user wants single-page line edits, not analysis — defer to editing tools
- The user says "GEO audit" in a geographic/geolocation context (geographic information system, location-based services) — require co-occurring **AI**, **LLM**, **citation**, **ChatGPT**, **Perplexity**, **Gemini**, or **Claude** token to confirm

## The skill has four modes

| Mode | Trigger | What it does |
|---|---|---|
| **audit** (default) | "audit our AI citation", "AI citation audit" | Brand-wide audit → HTML report + JSON snapshot + walkthrough offer + delta enrollment |
| **single-page** | "score this page", "citability check on <url>" | Degenerate audit — skips site crawl and off-site probe; focused passage report |
| **walkthrough** | "AI citation walkthrough", "walk me through GEO fixes" | Interactive playbook — top 5-7 moves with branching `done / show-me-how / skip` |
| **delta** | "AI citation delta", "compare our AI citation" | Load most recent snapshot → re-audit → compute deltas → render delta report |

## Procedure

The default flow is opinionated: **audit → report → offer walkthrough → offer delta enrollment**. Each step is below.

### Step 0 — Detect mode

Parse the trigger phrase and any URL/argument provided. Set `mode = audit | single-page | walkthrough | delta`. If mode is `audit` and no URL was provided, ask for the root URL (Q1 below).

### Step 1 — Gather minimum context

Scan the conversation first; do not re-ask anything the user already provided. Then ask up to 4 focused questions, **one at a time**.

**Q1 — Root URL.** Required for audit mode. Skip if already in context.

**Q2 — Brand identity + disambiguation.**

First, try to extract the brand name from the homepage's `<title>`, `og:site_name`, or Organization JSON-LD.

- If the extracted name is **≥3 tokens, unambiguous, or contains an uncommon proper noun**: skip the disambiguation prompts. Just ask for the one-sentence category description.
- Otherwise: prompt for disambiguation context:

> I've identified the brand as **"<name>"**. That's a common word — to keep off-site searches clean, give me a few additional phrasings:
>
> 1. **Full company name** (e.g., "Easy Labs Inc.")
> 2. **Category phrasing** (e.g., "Easy payment processor", "Easy PSP")
> 3. **Alternative names / handles** (e.g., "@useeasy", "easy.com") — optional
>
> Also: **what is the one-sentence category description?** (e.g., "payment service provider for SaaS companies")

**Q3 — Live AI citation probing opt-in (sticky per site).**

If `preferences.json` for this site does not have a recorded answer for `ai_probing`, ask:

> Want me to also probe live AI engines (ChatGPT/Perplexity/Gemini) to measure your actual citation rate?
>
> **A.** Yes — I have API keys, or I'll paste the answers manually
> **B.** Skip for this run
> **C.** No, don't ask again

**Q4 — Google Search Console integration (sticky per site).**

If `preferences.json` does not have a recorded answer for `gsc_integration`, ask:

> A more thorough report can use Google Search Console data to find queries you're already getting impressions for but not optimized for. Requires GSC API credentials.
>
> **A.** Yes — I'll provide credentials now (or read from `~/.config/ai-citation/gsc-credentials.json`)
> **B.** Skip for this run
> **C.** No, don't ask again

**Preference semantics — these are not negotiable:**

- **A (Yes):** take action this run; on future runs, re-confirm only if state changed (e.g., credentials expired)
- **B (Skip):** proceed without it this run; ask again next run
- **C (No, don't ask again):** write to `preferences.json`; never ask again for this site

Persist preferences at `<storage>/preferences.json` with this structure:

```json
{
  "site_slug": "example-com",
  "brand": {
    "primary": "Easy",
    "full_name": "Easy Labs Inc.",
    "category_phrasings": ["Easy payment processor", "Easy PSP"],
    "handles": ["@useeasy", "easy.com"],
    "category_description": "payment service provider for SaaS companies"
  },
  "preferences": {
    "ai_probing":      { "answered": "C", "answered_at": "2026-05-13", "scope": "site" },
    "gsc_integration": { "answered": "C", "answered_at": "2026-05-13", "scope": "site" },
    "target_engine":   "perplexity",
    "cadence_days":    30
  }
}
```

Per-site by default. Global override at `~/.ai-citation/preferences.json` if user copies the file.

### Step 2 — Discover surface area (audit mode only)

1. Fetch the root URL. Confirm reachability. Extract canonical domain, primary language, brand name (cross-check Q2).
2. Try `/sitemap.xml`. If present, sample up to **25 pages** weighted toward: homepage + product/feature pages + blog/glossary/docs (configurable via the `--max-pages` flag — see "Slash-command flags" below).
3. If no sitemap: BFS from homepage, 2 levels deep, dedupe, cap at 25.
4. Also fetch: `robots.txt` (once, cached) and `llms.txt` (if present — noted but low-weight).

### Step 3 — On-page audit (per page)

For each sampled page, fetch and extract:

- **Technical:** HTTP status, render-blocking signals, robots/auth walls, canonical URL, mobile viewport meta, JSON-LD schemas, OpenGraph completeness
- **Structural:** heading hierarchy, paragraph length distribution, tables, lists, FAQ schema presence
- **Passage-level:** segment content into blocks at H2/H3; score each block against the block rubric (see `rubric/block-rubric.md`)
- **Freshness:** publish date, last-modified, age (60-day threshold)
- **Query-answer match:** at least one question-shaped H2/H3 with a clean 40-60 word direct answer

Apply the **Readiness Score** rubric in `rubric/readiness-factors.md` (12 factors, weights sum to 100) and the page-level inputs to the **Action Score** rubric in `rubric/action-factors.md` (8 factors, weights sum to 100).

### Step 4 — Off-site presence probe (audit mode only)

Build a query set from Q2 disambiguation data:

```
queries = [
  '"<full company name>"',
  '"<brand>" <category phrasing>',
  '"<alternate phrasing>"',
  '"<domain>"',
  '@<handle>',
]
```

Run each over six off-site platforms via WebSearch:

1. **Reddit** — `site:reddit.com <query>` — critical for Perplexity (46.7% citation share)
2. **YouTube** — `site:youtube.com <query>` — top social citation source in early 2026
3. **Wikipedia** — direct article check — critical for ChatGPT (47.9%)
4. **G2 / Capterra / TrustRadius** — SaaS only
5. **Quora** — Q&A footprint
6. **Comparison/listicle sites** — `"best <category>" "<brand>"`

For non-SaaS brands, skip G2/Capterra. For local-services brands, skip Wikipedia.

**Match-quality weighting:**

- Exact full-name match → 1.0
- Brand + category match → 0.8
- Alternate phrasing match → 0.7
- Bare brand + unrelated context → **0.0 (discarded)**

**False-positive guard (mandatory for ambiguous brand names):** for every result, check whether category-description tokens appear in result title/snippet. Drop if not. Print this transparency note in the final report:

> "Off-site probe used N query variants and dropped K results that didn't match the category context."

Score each platform 0/5/10 per the rubric in `rubric/brand-surface-area.md`. Produce the **Brand Surface Area Map**.

### Step 5 — Optional live AI citation probing

Only if `preferences.ai_probing.answered == "A"` for this run.

**Before running:** print an estimated cost based on the number of seed prompts × the number of available engines × estimated tokens per response. Get explicit user confirmation before incurring API costs:

> "Live probing will run {{prompt_count}} seed prompts against {{engine_count}} engines. Estimated cost: ~${{cost_estimate}}. Continue? (y/n)"

If the user declines, fall back to printing the seed prompts for manual probing.

1. Detect available engines: check for OpenAI API key (env or `~/.config/ai-citation/openai-key`), xAI key, Anthropic key, Perplexity key. Use whichever MCPs are installed.
2. Derive a seed prompt set (~10 prompts) from category + competitors + product features. Examples:
   - "best <category> tool for <use case>"
   - "<competitor 1> vs <competitor 2>"
   - "<specific feature> for <audience>"
   - "alternatives to <competitor>"
3. Query each available engine. Record whether the brand appears in the answer.
4. Compute actual citation rate per engine. Store under `scores.per_platform_measured.<engine>` in the snapshot.
5. If no APIs available, print the seed prompts and offer:

> No engine APIs detected. I can give you the seed prompts to paste into ChatGPT/Perplexity/etc. manually, then you drop the answers back to me and I'll score them. Want to do that?

### Step 6 — Compute scores

Apply the rubrics. Both rubric files have explicit aggregation formulas.

```
Composite = 0.5 × Readiness + 0.5 × Action
```

Compute the **Per-Platform Snapshot** projections per `rubric/platform-preferences.md`. If a measured rate exists for an engine, use it instead of the projection (flag with ✓).

### Step 7 — Prioritized recommendations

Cluster findings into a flat list. Score each by **(impact × ease)** where impact is the rubric-derived score lift and ease is the inverse of effort (low/medium/high).

Sort descending. Tag each: `on-page | off-site | technical | content`. Split into time buckets:

- **Week 1** — trivial fixes, low effort, immediate lift
- **Month 1** — meaningful fixes, low-to-medium effort
- **Quarter 1** — strategic moves, medium-to-high effort

The top 5-7 items become the candidate set for the walkthrough.

### Step 8 — Render report

Use `templates/audit-report.html.template` as the HTML scaffold.

Write three artifacts to `<storage>/`:

- `<date>-report.html` — the human-readable deliverable
- `<date>-baseline.json` — durable structured snapshot (see Step 8 schema)
- (Print a markdown summary in chat with composite score, top 3 wins, top 3 gaps, link to the HTML file)

**Storage location resolution:**

1. If the audited URL maps to a local git repo (e.g., the user is in that repo right now) → write to `<repo>/.ai-citation/snapshots/<site-slug>/` so the snapshots get checked in
2. Otherwise → `~/.ai-citation/snapshots/<site-slug>/`
3. Always print the absolute path so the user can find it

**Site slug:** primary registered domain, dots → dashes (`example.com` → `example-com`).

**Snapshot JSON schema:**

```json
{
  "schema_version": "1.0",
  "methodology_version": "1.0",
  "run": {
    "id": "2026-05-13T14:22:03Z",
    "type": "baseline",
    "previous_snapshot": null
  },
  "site": {
    "slug": "example-com",
    "root_url": "https://example.com",
    "brand": { /* from preferences.json */ },
    "pages_audited": 25,
    "pages_list": ["/", "/pricing", ...]
  },
  "scores": {
    "composite": 63,
    "readiness": 58,
    "action": 68,
    "per_platform": {
      "chatgpt": 71, "perplexity": 49, "gemini": 64, "ai_overviews": 60, "claude": 67
    },
    "per_platform_measured": {
      "perplexity": { "rate": 0.20, "measured_at": "2026-05-13", "prompts_tested": 10 }
    }
  },
  "factor_scores": {
    "readiness": { "url_accessibility": 9, "search_rank_proxy": 6, ... },
    "action":    { "passage_citability": 72, "brand_surface_area": 45, ... }
  },
  "brand_surface_area": {
    "wikipedia": 0, "reddit": 5, "youtube": 3, "g2": 7, "quora": 0, "listicles": 4
  },
  "per_page": [...],
  "recommendations": [
    { "id": "rec_001", "move": "rewrite-top-3-pages-definition-first",
      "impact": 4, "effort": "low", "status": "open" }
  ]
}
```

### Step 9 — Offer walkthrough

After the report renders, prompt:

> "You scored **{{composite}}/100**. The top 5-7 moves would lift you to an estimated **{{projection}}/100** in 30 days. Want me to walk you through them now?
>
> **A.** Yes — start the walkthrough
> **B.** Save for later — I'll write the plan to a file
> **C.** No"

- **A:** Enter walkthrough mode (see "Walkthrough state machine" below) using the recommendations in memory
- **B:** Generate `walkthrough-plan.md` using `templates/walkthrough-plan.md.template` with statuses `pending` for each move
- **C:** Skip to Step 10

### Step 10 — Offer delta enrollment (the nudge)

Detect available reminder channels in order:

1. **Google Calendar MCP** present? (look for `mcp__claude_ai_Google_Calendar`)
2. **Linear MCP** present? (look for `mcp__claude_ai_Linear`)
3. **`schedule` skill** installed?
4. None — fallback to `.ics` file + `NEXT-REVIEW.md`

Show only the available options:

> "You scored {{composite}}/100. Recommended cadence: monthly. Pick a reminder channel:
>
> **A.** Google Calendar — recurring event with the slash command in the description
> **B.** Linear — recurring issue assigned to you
> **C.** .ics file — calendar invite dropped in the snapshot folder
> **D.** Cron via the `schedule` skill — autonomous re-runs (most hands-off)
> **E.** None — just write NEXT-REVIEW.md and trust me to remember"

On choice, set up the reminder with this body:

> "Run `/ai-citation delta <site>` to compare against your <date> baseline.
> Baseline composite: <score>/100. Snapshot: <path>."

**Always** write `NEXT-REVIEW.md` using `templates/next-review.md.template` regardless of channel chosen — it is the always-emit fallback.

### Step 11 — Final printout

Markdown summary in chat: composite score, paths to all artifacts (report HTML, snapshot JSON, walkthrough plan if generated, NEXT-REVIEW.md), reminder channel used (or "none chosen"), exact slash command to run the next check.

## Walkthrough state machine

Entered from Step 9 of the audit funnel **or** directly via trigger ("walk me through GEO fixes"). If invoked standalone with no recent snapshot, prompt to run audit first or load a baseline JSON.

For each of the top 5-7 moves (selected from the catalog in `move-catalog/moves.md` by audit findings):

1. **Introduce the move** with its name, *what*, *why* (with research citation), estimated impact (e.g., `+4 composite points`), effort (low/medium/high).

2. **Prompt for branch:**

   > "**A.** Already have this — mark done, log evidence, move on
   > **B.** Show me how — go deep on this one
   > **C.** Skip — note reason, move on"

3. **If A (done):** ask for optional evidence URL/screenshot; record as `status: done` with `evidence` field; move on.

4. **If B (show me how):** load the playbook block for that move from `move-catalog/moves.md` and walk through:
   - Concrete steps (numbered, copy-pasteable)
   - Templates (JSON-LD blocks, comment templates, outreach templates)
   - Real examples (1-2 sites that did this well)
   - Common pitfalls (what NOT to do)
   - How to verify it worked
   - **Optional:** if Linear MCP available, offer to file a tracking issue with the playbook embedded
   - User can interrupt with "next" / "stop" / "skip the rest"

5. **If C (skip):** ask "skip permanently for this site, or just for this run?" Persist to `preferences.json` if permanent.

At the end:

- Save `walkthrough-plan.md` via `templates/walkthrough-plan.md.template` with statuses per move
- Print summary in chat: how many marked done, how many in progress, how many skipped
- Offer: "Want me to enroll you in monthly delta tracking? (yes / no)" if not already enrolled

### Move selection logic

The top 5-7 moves are selected from `move-catalog/moves.md` by:

1. **Filter out moves the site already has** (e.g., if `brand_surface_area.g2 == 10`, skip move 4)
2. **Filter out moves disqualified by category** (e.g., move 4 only for SaaS, move 6 only if Wikipedia notability check passes)
3. **Filter out moves the user already permanently skipped** (per `preferences.json`)
4. **Score remaining moves by (impact × ease)**
5. **Apply target_engine bias** if `preferences.target_engine` is set (e.g., Perplexity target → bump Reddit-related moves up)
6. **Take top 5-7**

## Delta mode procedure

Triggered by: "AI citation delta", "compare our AI citation", "monthly AI citation check", `/ai-citation delta <site>`.

1. Load the most recent snapshot for the site slug from `<storage>/`. If none, refuse:

   > "No baseline found for this site. Run `/ai-citation audit <url>` first to establish a baseline."

2. Run **Steps 2-7** of the audit fresh. Use the same `preferences.json` (brand identity, GSC opt-in, AI probing opt-in stay sticky).

3. Compute deltas:
   - **Composite delta** = current - baseline (sign matters)
   - **Per-factor deltas** — flag movers: ≥5 points up = **win**, ≥5 points down = **regression**
   - **Recommendation status diff** — apply auto-close detectors from `move-catalog/moves.md` to each open recommendation
   - **New findings** — issues present in current but not baseline
   - **Persistent gaps** — issues present in both
   - **Brand Surface Area movement** per platform

4. Write `<date>-delta.json` with the same schema plus a `delta` block. Set `run.type = "delta"` and `run.previous_snapshot = <baseline filename>`.

5. **CRITICAL:** if `methodology_version` differs between baseline and current, the report MUST refuse direct numeric comparison and instead surface a "methodology has changed — re-baseline recommended" banner. Comparisons are informational only.

6. Render `<date>-delta-report.html` using `templates/delta-report.html.template`.

7. If `Action Score` did NOT improve since baseline, re-offer the walkthrough:

   > "Your Action Score is flat ({{action_baseline}} → {{action_current}}). The walkthrough should help find what got stuck. Want to run it now?"

### Auto-close detector logic

For each open recommendation, apply the detector defined for that move in `move-catalog/moves.md`. Status options:

- `closed` — detector confirms the move is done (e.g., FAQ schema now present on all 10 named pages)
- `partial` — partial progress detected (e.g., 6 of 10 pages now have schema)
- `open` — no progress detected
- `regressed` — the move was previously detected as done but is now undone

**Default fallback:** for any move without an explicit auto-close detector, status defaults to `open` and is surfaced in the delta report under "Awaiting manual verification" with a prompt to mark `done` / `partial` / `skipped` during the next walkthrough. The skill MUST NOT mark such moves `closed` based on inference.

## Anti-patterns the skill refuses to execute

The walkthrough's playbooks include educational-only patterns for legitimate growth. The skill explicitly refuses these:

1. **Astroturfing Reddit / Quora / forums** — sock-puppet accounts, undisclosed-affiliation posts, paid commenters. If asked, respond:

   > "I won't generate inauthentic Reddit/Quora content. The legitimate playbook is in Move 3 of the catalog — disclosure-first, real account history, educational answers."

2. **Fake reviews** for G2 / Capterra / TrustRadius / Trustpilot / etc. If asked, refuse and point to the customer-solicitation template in Move 4.

3. **Wikipedia article seeded by the brand itself with marketing language** — gets flagged or deleted. Move 6 has the legitimate playbook (AfC submission, neutral tone, third-party-cited claims).

4. **Blank-page-with-schema-only growth hacks** (e.g., the well-known Hobo_Web 36-hour Perplexity citation experiment). Mention in passing as a documented short-term tactic, but flag explicitly: works short-term, ages poorly, AI engines are increasingly filtering thin-content pages. Teach the sanitized version: rich schema on **real** content.

5. **Aggressive WAF rules that block AI bots while pretending they don't** — recommend explicit allow-listing per Move 7.

6. **LLMs.txt over-investment** — Cyrus Shepard's analysis scored llms.txt 2.0/10. The skill MUST surface this in every audit's "Don't waste time on" callout.

The walkthrough's "Common pitfalls" section for each move must include the bad version alongside the good version, so the team learns the boundary.

## Quick reference

| Step | What | Skip = |
|---|---|---|
| 0 | Detect mode (audit / single-page / walkthrough / delta) | Wrong procedure dispatched |
| 1 | Gather minimum context (URL, brand identity, sticky prefs) | Generic results, polluted off-site probe on ambiguous brands |
| 2 | Discover up to 25 priority pages via sitemap or BFS | Missed primary content surfaces |
| 3 | On-page audit per page (passage rubric + Readiness factors) | Cannot compute scores |
| 4 | Off-site probe across 6 platforms with disambiguation | BSAM is unscored; can't recommend off-site moves |
| 5 | Optional live AI citation probing if opted in | Projections only, no measured ground-truth |
| 6 | Compute composite + per-platform projections | No deliverable |
| 7 | Prioritize recommendations by impact × ease, bucket 30/60/90 | Unprioritized advice dump |
| 8 | Render HTML report + JSON snapshot | No artifact for handoff or future delta |
| 9 | Offer walkthrough on top 5-7 moves | Audit becomes "advice" — no execution loop |
| 10 | Offer delta enrollment with channel detection | No accountability — audit drift |
| 11 | Final printout with paths and slash commands | User loses track of artifacts |

## Common mistakes

- **Skipping Q2 disambiguation on ambiguous brand names.** "Easy", "Apple", "Wave" — without disambiguating phrasings + category filter, the off-site probe returns garbage. The Q2 procedure trips the prompt when the extracted brand is **<3 tokens, OR is a common English word, OR lacks an uncommon proper noun**. When in doubt, ask.
- **Recommending llms.txt as a high-impact move.** Shepard scored it 2.0/10. Mention it in the "Don't waste time on" callout, not the action plan.
- **Confusing projections with measurements.** Per-Platform Snapshot values are projections derived from factor mix. Only call them "measured" when live AI probing actually ran. Always flag measured rates with ✓.
- **Auto-closing recommendations without re-measurement.** The delta engine must re-measure (re-fetch pages, re-probe off-site) — never trust the user said "we did it" without verification.
- **Generating fake content for off-site moves.** Refuse. Point to the disclosed-affiliation playbook in Move 3.
- **Treating the composite score as the only number that matters.** Both Readiness AND Action have to move for real progress. A site can game Readiness via short-term tactics and stall on Action; the report should call this out.
- **Ignoring the storage location.** Always print the absolute path of the snapshot folder. Users get lost.
- **Forgetting the always-emit NEXT-REVIEW.md.** Whether or not the user picks a reminder channel, this file is the bottom-line fallback. Write it every time.

## Red flags — STOP and restart

| Thought | Reality |
|---|---|
| "The brand name is fine, no need to disambiguate" | Check token count and commonness. "Easy" with no disambiguation = polluted probe. |
| "I'll just give them generic GEO advice in chat" | That's not this skill. Produce the HTML report. |
| "Two out of three preferences answered is enough" | All three Q3/Q4 answers matter. Sticky semantics depend on them. |
| "I can infer the user wants Perplexity since they mentioned it once" | Ask explicitly which engine matters most — bias the walkthrough on it. |
| "I'll set up the calendar event without offering options" | Offer the menu (A/B/C/D/E). Some users prefer .ics, some Linear. |
| "Schema sprays everywhere will boost their score" | False. Validity matters more than presence. Targeted, valid schemas only. |

## What this skill is not

- **Not a traditional SEO audit tool.** It complements but doesn't replace Ahrefs, Semrush, or Search Console. Surface AI-search-specific issues; don't recompute keyword difficulty.
- **Not a content generation tool.** The walkthrough teaches the team to write better content; the skill itself doesn't ghostwrite blog posts or generate astroturfing scripts.
- **Not a paid-citation broker.** "Pay to be cited by AI" doesn't exist as a legitimate offering. Be skeptical of any vendor claiming otherwise.

## Slash-command flags

The skill accepts optional flags passed as `key=value` arguments after the trigger phrase:

| Flag | Default | Effect |
|---|---|---|
| `--max-pages` | 25 | Cap on pages crawled in Step 2 |
| `--rebrand` | (off) | Force re-prompt of brand disambiguation in Q2, ignoring `preferences.json` |
| `--priority-urls` | (none) | Comma-separated list of URLs to treat as priority pages (weighted 2× in rollup) |
| `--category` | (auto-detect) | Override SaaS/ecom/media/local-services category detection |
| `--cadence` | 30 (days) | Override the default monthly review cadence |
| `--enroll-cron` | (off) | Enroll site in autonomous cron-driven delta runs (requires `schedule` skill) |
| `--target-engine` | (auto/asked) | Bias walkthrough prioritization toward one engine (chatgpt / perplexity / gemini / aio / claude) |

Example: `/ai-citation audit https://example.com --max-pages=50 --target-engine=perplexity`

## Reference files (loaded on demand by the procedure)

- `rubric/readiness-factors.md` — 12 Readiness factors with weights and per-factor scoring criteria
- `rubric/action-factors.md` — 8 Action factors with weights and scoring criteria
- `rubric/block-rubric.md` — 6-criterion block-level passage rubric
- `rubric/brand-surface-area.md` — 6-platform off-site presence rubric
- `rubric/platform-preferences.md` — Per-engine projection formulas
- `move-catalog/moves.md` — 12-move catalog with playbook blocks and auto-close detectors
- `templates/audit-report.html.template` — Audit report HTML scaffold
- `templates/delta-report.html.template` — Delta report HTML scaffold
- `templates/walkthrough-plan.md.template` — Walkthrough output markdown
- `templates/next-review.md.template` — Always-emit reminder markdown
