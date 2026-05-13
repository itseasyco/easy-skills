# Design — `ai-citation` skill for easy-skills

**Date:** 2026-05-13
**Author:** Andrew Fisher (brainstormed with Claude Code)
**Status:** Approved design — ready for implementation plan
**Target location:** `skills/ai-citation/` in the `easy-skills` plugin

---

## 1. Purpose and scope

A self-contained Claude Code skill that helps marketing teams, agencies, and solo builders raise their **AI Citation Score** — the likelihood their site is cited or quoted by AI engines (ChatGPT, Perplexity, Gemini, Google AI Overviews, Claude).

The skill replaces the standalone `geo-citability` page-scorer and consolidates passage-level scoring, brand-wide auditing, off-site presence mapping, an interactive growth-hack walkthrough, and monthly delta tracking into one package. It is built without dependencies on other skills; any functionality previously delegated is rebuilt inline.

### Primary user

Designed for "agency-grade deliverable" so the same output serves:
- In-house marketing teams running it on their own site
- SEO/GEO agencies running it for clients
- Solo founders running it on scrappy sites

### Out of scope (deliberate non-goals)

- Replacing traditional SEO tools (Ahrefs, Semrush, etc.) — this skill complements them, not replaces them
- Automating Reddit/YouTube/Wikipedia content creation — the skill walks users through these moves, refuses to astroturf
- Black-hat citation hacks (sock-puppet accounts, fake reviews, blank-page-with-schema-only) — flagged as anti-patterns; the legitimate sanitized version is what the playbook teaches

---

## 2. Skill identity

### Name
`ai-citation`

### Trigger phrases

**Brand-wide audit triggers (default):**
- "audit our AI citation", "audit my AI citation", "AI citation audit"
- "get cited by ChatGPT", "get cited by Perplexity", "get cited by Claude", "get cited by Gemini", "get cited by AI"
- "raise our AI citation score", "improve our GEO score"
- "GEO audit", "generative engine optimization audit"
- "AI visibility audit", "AI search visibility check"

**Single-page invocation (degenerate case of audit — skips crawl and off-site):**
- "score this page for AI citation", "is this page citable by AI"
- "citability check on <url>", "how citable is <url>"

**Sub-mode triggers (jump past audit):**
- "AI citation delta", "compare our AI citation", "monthly AI citation check" → delta mode
- "AI citation walkthrough", "growth-hack my AI citation", "walk me through GEO fixes" → walkthrough mode

**Anti-triggers (don't fire):**
- Pure traditional SEO questions ("how do I rank for X keyword on Google") — unless they also mention AI/LLM/GEO
- Conceptual GEO questions ("what is GEO?") — answer directly, don't run audit
- Single-page tweaks where the user clearly wants line edits, not analysis
- "GEO audit" used in geographic/geolocation contexts — require co-occurring AI/LLM/citation/ChatGPT/Perplexity/etc. token

---

## 3. File structure

### Skill source tree

```
skills/ai-citation/
├── SKILL.md                          # main skill file with procedure + rubric
├── rubric/
│   ├── readiness-factors.md          # the 12 Shepard-derived AI Citation Readiness factors with weights
│   ├── action-factors.md             # the 8 Action factors (passage quality, off-site, freshness)
│   ├── block-rubric.md               # block-level passage rubric (6 criteria)
│   ├── brand-surface-area.md         # off-site platform scoring rubric
│   └── platform-preferences.md       # per-platform projection formulas
├── templates/
│   ├── audit-report.html.template    # styled HTML report (audit + single-page modes share)
│   ├── delta-report.html.template    # delta comparison report
│   ├── walkthrough-plan.md.template  # markdown plan saved at end of walkthrough
│   └── next-review.md.template       # the always-emit fallback reminder
├── move-catalog/
│   └── moves.md                      # the 12-move catalog with playbook blocks
└── examples/
    └── sample-audit-report.html      # example output for docs
```

### Runtime storage (snapshots persist between runs)

```
.ai-citation/snapshots/<site-slug>/
├── preferences.json                  # sticky prefs (brand info, GSC opt-in, AI probing opt-in, cadence)
├── 2026-05-13-baseline.json          # full audit snapshot
├── 2026-05-13-report.html
├── 2026-06-13-delta.json
├── 2026-06-13-delta-report.html
├── walkthrough-plan.md               # latest playbook with statuses
└── NEXT-REVIEW.md                    # always-emit fallback nudge file
```

### Storage location resolution

1. If audited URL maps to a local git repo → write to `<repo>/.ai-citation/snapshots/<slug>/` (gets checked in)
2. Otherwise → `~/.ai-citation/snapshots/<slug>/`
3. Always print the path used so the user can find it

Site slug = primary registered domain, dots → dashes (`example.com` → `example-com`).

---

## 4. Procedure (the audit funnel)

The skill follows an opinionated front-to-back flow: audit → report → walkthrough offer → delta enrollment. The funnel mirrors the actual work: audit reveals problems → walkthrough fixes them → delta proves it worked.

### Step 0 — Mode detection

Parse trigger phrase + arguments. Set `mode = audit | single-page | walkthrough | delta`. If `audit` and no URL provided, prompt the user for the root URL.

### Step 1 — Gather minimum context

Scan conversation first to avoid re-asking. Ask up to 4 focused questions, one at a time:

**Q1 — Root URL** (required for audit mode; skip if already in context)

**Q2 — Brand identity + disambiguation context**

Skill extracts brand candidate from `<title>` / `og:site_name` / Organization schema. If the brand name is **≥3 tokens, unambiguous, or contains an uncommon proper noun**, skip the disambiguation prompts and just ask for the one-sentence category description.

Otherwise prompt:

> I've identified the brand as **"<name>"**. That's a common word — to keep off-site searches clean, give me a few additional phrasings I can use:
>
> 1. **Full company name** (e.g., "Easy Labs Inc.")
> 2. **Category phrasing** (e.g., "Easy payment processor", "Easy PSP")
> 3. **Alternative names / handles** (e.g., "@useeasy", "easy.com") — optional
>
> Also: **what is the one-sentence category description?** (e.g., "payment service provider for SaaS companies") — used to derive seed prompts for live AI probing.

**Q3 — Live AI citation probing opt-in** (offered once per site; preference remembered):

> Want me to also probe live AI engines (ChatGPT/Perplexity/Gemini) to measure your actual citation rate?
> **A.** Yes — I have API keys, or I'll paste the answers manually
> **B.** Skip for this run
> **C.** No, don't ask again

**Q4 — Google Search Console integration** (offered once per site; preference remembered):

> A more thorough report can use Google Search Console data to find queries you're already getting impressions for but not optimized for. This requires GSC API credentials.
>
> **A.** Yes — I'll provide credentials now (or you can read them from `~/.config/ai-citation/gsc-credentials.json`)
> **B.** Skip for this run
> **C.** No, don't ask again

**Preference persistence semantics:**
- **A (Yes)** → take action this run; on future runs, re-confirm only if state changed (e.g., creds expired)
- **B (Skip)** → proceed without it this run; ask again next run
- **C (No, don't ask again)** → write to preferences.json; never ask again for this site

Per-site by default. Global override at `~/.ai-citation/preferences.json` if the user copies the file.

### Step 2 — Discover surface area (audit mode only)

1. Fetch root URL → confirm reachability, extract canonical domain, primary language, brand name
2. Try `/sitemap.xml` → if present, sample up to 25 pages (configurable via `--max-pages`) weighted toward: homepage + product/feature pages + blog pages + glossary/docs
3. If no sitemap → BFS from homepage 2 levels deep, dedupe, cap at 25
4. For each page, also fetch: `robots.txt` once (cached), `llms.txt` if exists (noted but low-weight)

### Step 3 — On-page audit (per page)

For each page, extract and score:

- **Technical:** HTTP status, render-blocking signals, robots/auth walls, canonical URL, mobile viewport meta, JSON-LD schemas present, OpenGraph completeness
- **Structural:** heading hierarchy integrity (H1>H2>H3), paragraph length distribution, table count, list count, FAQ schema presence
- **Passage-level:** segment content into blocks at H2/H3; score each block against the block rubric (Section 6 below)
- **Freshness:** publish date, last-modified, age (60-day threshold per SE Ranking research)
- **Query-answer match:** at least one question-shaped H2/H3 followed by a 40-60 word direct answer

### Step 4 — Off-site presence probe (audit mode only)

Builds query set from Q2 disambiguation data:

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

**Match-quality weighting:**
- Exact full-name match → 1.0
- Brand + category match → 0.8
- Alternate phrasing match → 0.7
- Bare brand + unrelated context → 0.0 (discarded)

**False-positive guard:** for every result, check whether category-description tokens appear in result snippet/title. Drop if not. Print transparency note in report:

> "Off-site probe used N query variants and dropped K results that didn't match the category context."

Output: a **Brand Surface Area Map** with mention counts, sentiment signal where visible (decorative — surfaced in the report but NOT factored into the Action Score; see Section 5 BSAM rubric for what is scored), gap flags, and "biggest absence" priorities.

### Step 5 — Optional live AI citation probing (only if opted in at Q3)

For each available engine (detect via env keys / MCPs):

1. Derive a seed prompt set (~10 prompts) from category + competitors + product features. Examples: "best <category> tool", "<competitor> alternatives", "<feature> for <use case>"
2. Query each engine, record whether brand appears in answer
3. Compute *actual citation rate* per engine and store it
4. If no APIs available, print the seed prompts and tell the user: "paste these into ChatGPT/Perplexity/etc., then drop the answers back to me — I'll score them"

### Step 6 — Compute scores

See Section 5 (Scoring rubric).

### Step 7 — Prioritized recommendations

Cluster all findings into a flat list, score each by **(impact × ease)**:

- **Impact** = estimated composite-score lift from rubric weights
- **Ease** = inverse of effort estimate (low/medium/high)

Sort descending. Tag each: `on-page | off-site | technical | content`. Split into time buckets: **Week 1 / Month 1 / Quarter 1**. The top 5-7 items become the candidate set for the walkthrough.

### Step 8 — Render report

- HTML report → `<storage>/<date>-report.html` (self-contained, no external assets)
- JSON snapshot → `<storage>/<date>-baseline.json` (or `-delta.json` in delta mode)
- Markdown summary printed in chat with composite score, top 3 wins, top 3 gaps, link to HTML file

### Step 9 — Offer walkthrough (the funnel)

After report is rendered, the skill prompts:

> "You scored **<composite>/100**. The top 5 moves would lift you to an estimated **<projection>/100** in 30 days. Want me to walk you through them now?
> **A.** Yes — start the walkthrough
> **B.** Save for later — I'll write the plan to a file
> **C.** No"

- **A** → enters walkthrough mode (Section 7) using the recommendations already in memory
- **B** → writes `walkthrough-plan.md` with the top moves but doesn't run interactive flow
- **C** → skip to Step 10

### Step 10 — Offer delta enrollment (the nudge)

Skill probes available reminder channels in order of preference:

1. Google Calendar MCP present?
2. Linear MCP present?
3. `schedule` skill installed?
4. None — fallback to `.ics` file + plain text

Skill presents only the available channels and asks user to pick one (or none). Default cadence: 30 days. On pick, sets up the reminder with this body:

> "Run `/ai-citation delta <site>` to compare against your <date> baseline. Baseline: <composite-score>. Snapshot: <path>."

**Always** writes `NEXT-REVIEW.md` regardless of choice — slash command, baseline score, snapshot path, "paste this into your calendar" block.

### Step 11 — Final printout

Markdown summary: composite score, paths to all artifacts (report HTML, snapshot JSON, walkthrough plan if generated, NEXT-REVIEW.md), reminder channel used (or "none chosen"), exact slash command to run next check.

### Edge cases

- **Site blocks bots / WebFetch fails** → degrade to a manual-input mode where user pastes content; flag the technical issue (blocking AI crawlers is a Readiness Score hit anyway)
- **Site has 1000+ pages** → cap at 25 by default; honor `--max-pages` override; sample by category
- **Single-page mode** → skip Steps 2 & 4; report focuses on passage rewrites; can still offer walkthrough for that single page
- **API rate limits during off-site probing** → skill caches WebSearch results in snapshot folder for 24h; subsequent runs reuse them
- **Brand name ambiguity not caught by heuristic** → user can re-run with `--rebrand` flag to update disambiguation

---

## 5. Scoring rubric (the two-score model)

### Composite AI Citation Score (0-100)

```
Composite = 0.5 × Readiness Score + 0.5 × Action Score
```

Headline number. Equal weight because the two halves serve different purposes — predict vs. fix.

### Score 1 — AI Citation Readiness Score (0-100)

**Measures:** likelihood the brand is *currently* cited. Shepard-derived weights, narrowed to factors evaluable via WebFetch + WebSearch.

| # | Factor | Weight | How measured |
|---|---|---|---|
| 1 | URL accessibility | 12 | HTTP 200; no auth/paywall; not JS render-only; canonical URL clean |
| 2 | Search rank proxy | 12 | For top 5 brand+category queries, brand in top-10 organic (via WebSearch) |
| 3 | Authority / brand signal | 12 | Wikipedia presence + `sameAs` graph depth + brand mention volume across web |
| 4 | Schema completeness | 10 | Organization, Product/Article, FAQ, HowTo, Breadcrumb, sameAs presence + validity |
| 5 | Query-answer match | 10 | At least one question-shaped H2/H3 with 40-60 word answer per page |
| 6 | Recency | 8 | % of priority pages with last-modified within 60 days |
| 7 | AI bot allow-list | 8 | robots.txt explicitly allows GPTBot, ClaudeBot, PerplexityBot, Google-Extended, OAI-SearchBot |
| 8 | Fan-out / topical depth | 8 | Internal linking depth + count of related sub-pages per topic cluster |
| 9 | Preview controls | 6 | Meta description, OG tags, no noindex/nosnippet/max-snippet:0 |
| 10 | Mobile + render health | 6 | viewport meta; reasonable response time; no obvious layout shift |
| 11 | Canonicalization | 4 | Single canonical per URL; no duplicate-content footprint |
| 12 | Sitemap + (optionally) llms.txt | 4 | sitemap.xml valid; llms.txt noted but low-weight (Shepard 2.0) |

Each factor scored 0-10, multiplied by weight, summed, divided by 10 → 0-100.

### Score 2 — Citability Action Score (0-100)

**Measures:** what can be *fixed this week*. Princeton/Georgia Tech/IIT Delhi GEO research-backed.

| # | Factor | Weight | How measured |
|---|---|---|---|
| 1 | Passage citability | 25 | % of content blocks scoring 70+ on the block rubric |
| 2 | Brand surface area off-site | 20 | Brand Surface Area Map score (sub-rubric) |
| 3 | Statistical density | 12 | Specific stats per 500 words; named studies/numbers/dollars (Princeton: +40%) |
| 4 | Authority quotations | 10 | Quoted experts with names + credentials (IIT Delhi: +115% in some categories) |
| 5 | Freshness cadence | 10 | % of priority pages updated within 60 days |
| 6 | Schema actionability | 10 | Count of high-leverage missing schemas the team could ship this week |
| 7 | Question-shaped headings | 8 | % of H2/H3 in question form |
| 8 | Internal answer architecture | 5 | Hub-and-spoke topical clusters; pillar+spoke linking present |

### Block-level passage rubric (feeds Action factor 1)

For every content block (segmented at H2/H3), check:

| Criterion | Pass condition |
|---|---|
| Definition-first opening | Block starts with `<subject> is/refers to/means…` in first sentence |
| Self-contained | First 60 words name subject explicitly; no leading pronouns |
| Word count fit | Block is 134-167 words OR sub-block extraction yields 50-200 word passages |
| Statistical density | ≥1 specific stat per 150 words (% / $ / named study / count) |
| Answer-first structure | Direct answer in first 1-2 sentences; supporting detail after |
| No vague quantifiers | Avoids "many," "most," "a lot," "studies show" without named source |

Block score = (criteria passed / 6) × 100. Block passes the 70+ threshold if ≥70.

### Brand Surface Area Map (feeds Action factor 2)

| Platform | 0 | 5 | 10 |
|---|---|---|---|
| Wikipedia | No article | Stub or weak article | Full article with reliable-source citations |
| Reddit | Zero threads found | 1-5 organic mentions in past 12mo | Active community discussion / 20+ mentions |
| YouTube | No videos | Some videos by others or low-view brand channel | Multiple high-view videos discussing brand |
| G2 / Capterra (SaaS only) | No listing | Listing exists | 20+ reviews, recent activity |
| Quora | Zero mentions | Some Q&A presence | Brand cited in answers to category questions |
| Comparison/listicle sites | Absent | Listed in 1-3 "best of" articles | Featured in major listicles for the category |

Skill detects SaaS vs. non-SaaS from the audit and only scores applicable platforms. Average across applicable.

### Per-Platform Snapshot (projection, not score input)

Same raw data, projected through different weightings:

| Engine | Projection formula |
|---|---|
| ChatGPT | 0.50 × Wikipedia + 0.20 × Quora + 0.20 × Web-authority + 0.10 × Recency |
| Perplexity | 0.50 × Reddit + 0.20 × Recency + 0.20 × Web-authority + 0.10 × Schema |
| Google AI Overviews | 0.50 × Search-rank + 0.30 × Schema + 0.20 × Answer-architecture |
| Gemini | 0.40 × Search-rank + 0.20 × Schema + 0.20 × Brand-signal + 0.20 × Recency |
| Claude | 0.40 × Passage-quality + 0.30 × Authority-quotations + 0.30 × Structure |

If user opted into live AI probing, *actual* observed citation rate replaces the projection for that engine, flagged "✓ measured".

### Rollup

```
For each page:
  block_scores = [score_block(b) for b in blocks]
  page_action_partial = weighted avg of action factors using block_scores
  page_readiness_partial = weighted avg of readiness factors

site_readiness = avg(page_readiness_partial across priority pages) + site-level factors (3, 7)
site_action = avg(page_action_partial) + site-level factors (2, 5)
composite = 0.5 × site_readiness + 0.5 × site_action
```

**Aggregation semantics — which factors are page-level vs site-level:**

| Score | Page-level factors (averaged across priority pages) | Site-level factors (scored once for the whole site) |
|---|---|---|
| Readiness | 1, 4, 5, 6, 8, 9, 10, 11 | 2 (search rank — inherently site-level), 3 (brand signal), 7 (robots.txt allow-list), 12 (sitemap + llms.txt) |
| Action | 1, 3, 4, 6, 7, 8 | 2 (off-site BSAM), 5 (freshness cadence) |

Authoritative source: `rubric/readiness-factors.md` aggregation summary.

Site-level factors are added to the averaged page-level partial via weighted blending so the final score stays normalized to 0-100.

**Priority pages weighted 2×:** homepage, top-3 product/feature pages, top-3 highest-traffic content pages.

**How "highest-traffic" is determined:**
- If GSC opted in (Q4 = A) → use actual impressions data
- Otherwise → fall back to depth-from-homepage as a traffic proxy (shallower = higher priority), with manual override available via `--priority-urls` flag

### Calibration & honesty

The report must:
1. **Cite the weight sources** — link Shepard's analysis for Readiness, Princeton/GT/IIT Delhi for Action. Don't pretend our weights are first-party research.
2. **Flag the LLMs.txt anti-pattern** — Shepard scored it 2.0/10. The report includes a "Don't waste time on" callout that names llms.txt, generic "schema everywhere" sprays without entity discipline, and bot-blocking via aggressive WAFs.

---

## 6. Reports (HTML deliverables)

### Audit report (`<date>-report.html`)

Self-contained HTML, openable from finder. Sections:

1. **Header** — composite score gauge, brand identity block, audit date, "next review due" date
2. **Executive summary** — 3 sentences: where you stand, why it matters, top recommendation
3. **Two-score breakdown** — Readiness Score with factor sub-bars, Action Score with factor sub-bars
4. **Per-Platform Snapshot** — 5 gauges (ChatGPT, Perplexity, Gemini, AIO, Claude). Measured rates flagged ✓.
5. **Brand Surface Area Map** — radar chart across the 6 platforms
6. **Top wins (passes)** — what's already working
7. **Top gaps (fails)** — what's broken, ranked by impact × ease
8. **Per-page table** — every page audited, sortable by composite, block-pass-rate, freshness
9. **Per-block findings (collapsible)** — every below-70 block with current text + suggested rewrite
10. **Prioritized 30/60/90-day action plan**
11. **"Don't waste time on" callout** — LLMs.txt and other anti-patterns
12. **Footer** — research citations, snapshot path, next-review CTA

### Delta report (`<date>-delta-report.html`)

Same shell, but with before/after panels for every numeric score. Sections added:

- **Composite delta banner** — +X / -X points, with directional arrow
- **Wins** — items closed since baseline (re-measured, not self-reported)
- **Regressions** — items that got worse
- **Still open** — carried-forward recommendations with original impact estimate
- **New findings** — issues present this run that weren't visible at baseline

### Brand identity in HTML reports

Header always shows: `<full company name>` · `<category description>` · `<audited URL>` · audit date. Disambiguation phrasings used during off-site probe shown in a small "search variants" tooltip.

---

## 7. Growth-hack walkthrough

### Entry conditions

- From Step 9 of audit funnel ("Walk through top 5 moves now?")
- Or directly via trigger ("walk me through GEO fixes")

If invoked standalone with no recent snapshot, skill prompts to run audit first or load a baseline JSON.

### Walkthrough state machine

For each of the top 5-7 prioritized moves:

```
1. Skill introduces the move
   - WHAT it is (one sentence)
   - WHY it matters (one sentence, research-cited)
   - IMPACT: estimated score lift (e.g., "+4 composite points")
   - EFFORT: low / medium / high
2. Skill prompts:
   A. Already have this — mark done, log evidence, move on
   B. Show me how — go deep on this one
   C. Skip — note reason, move on
3. If A: skill asks for evidence URL (optional; helps delta tracking)
4. If B: enter deep walkthrough
5. If C: skill asks "skip permanently for this site, or just for this run?" — sticky preference
```

### Deep walkthrough block (when user picks B)

Each move has a playbook with:

1. **Concrete steps** (numbered, copy-pasteable)
2. **Templates** — e.g., JSON-LD FAQ schema with placeholders
3. **Examples from real sites** — 1-2 sites that did this well, drawn from research
4. **Common pitfalls** — what NOT to do
5. **How to verify it worked** — what the next audit will show
6. **Optional: create tracking ticket** — if Linear MCP available, offer to file an issue with playbook embedded

User can interrupt at any point with "next" / "stop" / "skip the rest".

### Move catalog (initial 12 — skill picks top 5-7 based on biggest impact × ease per audit)

| # | Move | Impact | Effort | Source |
|---|---|---|---|---|
| 1 | Rewrite top-3 high-traffic pages with definition-first openings + stats per 150 words | High | Low | Princeton/GT GEO |
| 2 | Add FAQ schema to top-10 pages with question-shaped H2s and 40-60 word answers | High | Low | Shepard factor 4+5 |
| 3 | Seed Reddit presence in 3 relevant subreddits (educational, NOT spam — manual playbook) | High | Medium | @recap_david / Perplexity 46.7% |
| 4 | Get listed on G2 / Capterra + claim profile (SaaS only) | High | Low | Discovered Labs research |
| 5 | Publish brand sameAs graph in Organization schema (LinkedIn, Crunchbase, X, GitHub, Wikipedia) | Medium | Low | Hobo_Web growth hack, sanitized |
| 6 | Stand up a Wikipedia article if eligible (notability checklist included) | Very High | High | ChatGPT 47.9% citation share |
| 7 | Add ClaudeBot, GPTBot, PerplexityBot, Google-Extended explicitly to robots.txt allow-list | Low | Trivial | Shepard factor 7 |
| 8 | Refresh top-5 evergreen pages with current-year dates + updated stats | Medium | Low | SE Ranking 25.7% freshness lift |
| 9 | Build a YouTube presence — 3 explainer videos for top categories | High | High | Early-2026 YouTube > Reddit |
| 10 | Get included in top-3 third-party listicles ("best <category>") via outreach | High | Medium | Comparison-site research |
| 11 | Mine GSC for impressions-but-no-clicks queries, rewrite pages to match (only if GSC opted in) | High | Medium | @hridoyreh tactic |
| 12 | Pillar+spoke topical cluster build-out for top revenue topic | Medium | High | Internal answer architecture |

### Output: `walkthrough-plan.md`

Contains:
- Brand identity block + audit date
- Composite score baseline + projected score after these moves
- One section per move with status: `done` / `in-progress` / `skipped` (with reason)
- For each "show me how" move: the playbook block embedded inline
- Linear issue URLs if created
- "Verify after next audit" checklist at the bottom

### Anti-patterns the skill refuses to execute

- Astroturfing Reddit — playbook explicitly requires educational answers, real account history, disclosure where appropriate
- Generating fake testimonials for G2/Wikipedia
- Blank-page-with-schema-only trick (Hobo_Web) — mentioned but flagged risky; sanitized version (rich schema on real content) is what's taught

Skill prints a refusal block if asked to generate fake reviews, sock-puppet accounts, or astroturf scripts.

---

## 8. Delta tracking mechanics

### Snapshot format (`baseline.json` and `delta.json`)

Stable schema across versions for diffing reliability:

```json
{
  "schema_version": "1.0",
  "methodology_version": "1.0",
  "run": {
    "id": "2026-05-13T14:22:03Z",
    "type": "baseline | delta",
    "previous_snapshot": "2026-04-13-baseline.json"
  },
  "site": {
    "slug": "example-com",
    "root_url": "https://example.com",
    "brand": { "primary": "...", "full_name": "...", "category_phrasings": [...], "handles": [...], "category_description": "..." },
    "pages_audited": 25,
    "pages_list": ["/", "/pricing", "..."]
  },
  "scores": {
    "composite": 63,
    "readiness": 58,
    "action": 68,
    "per_platform": { "chatgpt": 71, "perplexity": 49, "gemini": 64, "ai_overviews": 60, "claude": 67 },
    "per_platform_measured": {
      "perplexity": { "rate": 0.20, "measured_at": "2026-05-13", "prompts_tested": 10 }
    }
  },
  "factor_scores": {
    "readiness": { "url_accessibility": 9, "search_rank_proxy": 6 },
    "action":    { "passage_citability": 72, "brand_surface_area": 45 }
  },
  "brand_surface_area": {
    "wikipedia": 0, "reddit": 5, "youtube": 3, "g2": 7, "quora": 0, "listicles": 4
  },
  "per_page": [
    {
      "url": "/pricing",
      "block_count": 8,
      "blocks_passing_70": 3,
      "passage_score": 52,
      "freshness_days": 14,
      "issues": ["no_definition_first_opening", "vague_quantifiers", "missing_faq_schema"]
    }
  ],
  "recommendations": [
    {
      "id": "rec_001",
      "move": "rewrite-top-3-pages-definition-first",
      "impact": 4,
      "effort": "low",
      "status": "open"
    }
  ]
}
```

### Delta computation (`/ai-citation delta <site>`)

1. Load most recent snapshot for site_slug
2. Run Steps 2-7 of audit fresh
3. Compute:
   - **Composite delta:** new − old
   - **Per-factor deltas:** flag movers (≥5 points up = win, ≥5 down = regression)
   - **Recommendation status diff:** which ones the new audit shows as resolved (re-measured, not self-reported)
   - **New findings:** issues present in new snapshot not in old
   - **Persistent gaps:** issues present in both
   - **Brand surface area movement:** per platform
4. Write `<date>-delta.json` (same schema, `run.type = "delta"`, `delta` block populated)
5. Render `<date>-delta-report.html`

### Auto-close logic (recommendation status detection)

The delta engine doesn't trust "the user said they did it." It re-measures:

- `rewrite-top-3-pages-definition-first`: re-score named pages; if passage_score on those pages went up ≥15 pts → `closed`
- `add-faq-schema-to-top-10-pages`: re-check JSON-LD; if 10 pages now have FAQPage → `closed`
- `add-bots-to-robots-allowlist`: re-fetch robots.txt; if all five AI bots (GPTBot, OAI-SearchBot, ClaudeBot, PerplexityBot, Google-Extended) now allowed → `closed`
- `seed-reddit-presence`: re-run off-site probe; if Reddit score moved 0→5 or 5→10 → `closed`
- Etc.

**Default fallback rule:** for any move that does not have an explicit auto-close detector defined, status defaults to `open` and is surfaced in the delta report under "Awaiting manual verification" with a prompt for the user to mark `done` / `partial` / `skipped` during the next walkthrough run. Implementation MUST NOT mark such moves `closed` based on inference.

Status options: `closed | partial | open | regressed`. Walkthrough plan auto-updates with new statuses on each delta run.

### The nudge (channel selection at end of audit)

After report rendered, skill probes available reminder channels and presents only those installed:

```
You scored 63/100. Recommended cadence: monthly.

Pick a reminder channel:
  A. Google Calendar — recurring event with slash command in description
  B. Linear — recurring issue assigned to you
  C. .ics file — calendar invite dropped in snapshot folder
  D. Cron via the `schedule` skill — autonomous re-runs (most hands-off)
  E. None — just write NEXT-REVIEW.md and trust me to remember
```

Options A-D only shown if the corresponding integration is detected.

### `NEXT-REVIEW.md` (always emitted)

```markdown
# Next AI Citation Review — <site>

**Baseline score:** <composite>/100 (run <date>)
**Next review due:** <date + 30 days>
**Snapshot path:** <path>

## To run the next check

```
/ai-citation delta <site>
```

## To add to your calendar manually

[Google Calendar add link]
[Outlook add link]
[Apple Calendar (.ics file in this folder)]

## Open recommendations (carry forward)

- [ ] <move> — +<impact> pts est.
- [ ] ...
```

### Cadence options

- Monthly (default)
- Bi-weekly (active iteration)
- Quarterly (mature site)

Asked once at delta enrollment, persisted in `preferences.json`. Skill prints "next-due" date in every report.

### Honesty about limits

Every reminder setup prints:

> "This reminder fires the nudge. It cannot make the audit happen — that requires you to run the command. If the discipline isn't there, pick option D (cron) and let the schedule skill run autonomously."

---

## 9. Anti-patterns and ethics

The skill must explicitly refuse or warn against:

1. **Astroturfing** — fake Reddit posts, sock-puppet accounts, manufactured discussion. Refuse to generate scripts for these.
2. **Fake reviews** — G2, Capterra, Trustpilot, Wikipedia astroturf. Refuse.
3. **Blank-page-with-schema-only growth hacks** — mention the Hobo_Web case but flag as risky and age-poorly; teach the sanitized version (rich schema on real content).
4. **Aggressive AI bot blocking via WAF** — mention as Readiness-Score-killer, recommend selective allow-listing not blanket blocks.
5. **LLMs.txt over-investment** — Shepard 2.0/10. Mention in passing, deprioritize, don't make it a top recommendation.

The walkthrough's "common pitfalls" sections must include the bad version of each move alongside the good version.

---

## 10. Acceptance criteria (what "done" looks like)

The skill is shippable when:

1. **Brand-wide audit** works end-to-end on a real public site (Easy Labs' own site as the dogfood target) and produces a coherent HTML report under 3 minutes of wall time
2. **Single-page mode** produces a focused passage-level rewrite report
3. **Walkthrough mode** can be entered standalone with a loaded snapshot, walks through 5+ moves with branching, saves a `walkthrough-plan.md`
4. **Delta mode** correctly computes deltas between two snapshots, auto-closes resolved recommendations via re-measurement, renders delta HTML
5. **Reminder enrollment** works for at least three channels (Google Calendar, .ics, NEXT-REVIEW.md); other channels degrade gracefully
6. **Brand disambiguation** correctly handles the "Easy" ambiguous-brand test case
7. **Anti-patterns** are refused with clear refusal blocks
8. **All research citations** in the report link to the correct sources (Shepard, Princeton, GT, IIT Delhi, SE Ranking)
9. **README** in the easy-skills repo updated to list `ai-citation` alongside `premortem`

---

## 11. Out-of-scope for v1

- Automated content generation for off-site moves (Reddit posts, YouTube scripts) — too risky, may add as opt-in v2
- Integration with paid SEO tools (Ahrefs, Semrush) — possible v2
- A SaaS-hosted version (skill is local-only)
- Multi-language audits — English-only for v1; non-English sites can still be audited but scoring rubric may misfire on definition-first patterns

---

## 12. Open risks / known limitations

1. **Search rank proxy is noisy** — WebSearch results vary by personalization/region; mitigate by averaging multiple query variants and labeling factor 2 "noisy estimate" in the report
2. **Ambiguous brand names beyond Easy** — heuristic is "3+ tokens or uncommon proper noun." Real-world failure cases may need an explicit "force disambiguation" flag
3. **WebFetch rate limits** — 25-page crawl × per-page schema/robots fetches can hit limits; cache aggressively
4. **Live AI probing cost** — when opted in, API costs accrue; warn user before running with estimate
5. **Schema-version drift in snapshots** — include `schema_version` (for JSON shape) AND `methodology_version` (for scoring formula / weight changes) from v1; write migration block on first schema change; delta reports MUST refuse to compare across `methodology_version` mismatches and instead recommend the user re-baseline
6. **Move catalog is opinionated** — the 12 moves reflect May 2026 research; needs periodic refresh as platforms evolve

---

## 13. References

Cyrus Shepard, "AI Citation Ranking Factors Analysis" (Zyppy Signal, 2026-05-07) — 54-experiment analysis distilled to 23 weighted factors.
Pranjal Aggarwal et al., "GEO: Generative Engine Optimization" (Princeton, 2024) — 30-115% visibility lift via citation/quotation/statistics tactics.
SE Ranking 2026 research — 25.7% freshness lift; 28% citation lift for sub-60-day content.
Discovered Labs, "AI Citation Patterns" (2026) — ChatGPT 47.9% Wikipedia, Perplexity 46.7% Reddit, YouTube as top social citation source in early 2026.
@neilpatel, @CyrusShepard, @aleyda, @Hobo_Web, @recap_david, @hridoyreh, @gouthamjay8 — X discussions May 2026.
@HowToAI_ (X, 2026-05-13) — GEO-SEO open-source Claude Code skill reference.

---

## 14. Implementation guidance for the next agent

The next step is `superpowers:writing-plans` to convert this design into a step-by-step implementation plan. Key implementation priorities for the planning agent:

1. **Start with the scoring engine** — rubric/* files + a pure-function scorer take no I/O and can be tested with fixtures
2. **Then the audit runner** — Steps 0-8 of the procedure, with WebFetch/WebSearch as the I/O layer
3. **Then the report renderer** — templates/* are pure transformations of snapshot JSON
4. **Then the walkthrough state machine** — depends on the audit runner producing recommendations
5. **Then delta** — depends on snapshot stability and the audit runner
6. **Finally the nudge** — MCP-detection layer + channel-specific adapters

The SKILL.md itself should be the orchestration layer that ties these together, kept under 500 lines via progressive disclosure into the rubric/, templates/, and move-catalog/ subfolders.
