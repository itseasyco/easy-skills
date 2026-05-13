# AI Citation Readiness Factors

The Readiness Score (0-100) measures the likelihood the brand is **currently** being cited by AI engines. Weights are derived from Cyrus Shepard's May 2026 analysis of 54 AI citation experiments, narrowed to factors evaluable via WebFetch + WebSearch.

Each factor is scored **0-10**, then multiplied by its weight. Sum all weighted scores, divide by 10, → **0-100**.

```
Readiness Score = (Σ factor_score × weight) / 10
```

Weights sum to 100. The math is deterministic.

---

## The 12 factors

### Factor 1 — URL accessibility (weight: 12)

**What to check (per page):**

| Score | Criteria |
|---|---|
| 10 | HTTP 200, no auth/paywall, server-rendered HTML, canonical URL clean (no tracking params, no fragment-only routing) |
| 7 | HTTP 200, but heavy client-side rendering OR canonical URL has tracking-param bloat |
| 4 | HTTP 200, but key content gated behind login/paywall/age-gate/JS-required interstitial |
| 0 | HTTP error, 403/404/500/timeout, or robots.txt blocks the page |

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Shepard scored URL accessibility 9.5/10 — the single most predictive factor. AI engines need to fetch the page during live grounding. If they can't reach it, they can't cite it.

---

### Factor 2 — Search rank proxy (weight: 12)

**What to check (site-level):**

For the top 5 brand+category queries (derived from the brand's category description in `preferences.json`), run a WebSearch and check whether the brand appears in the top 10 organic results.

| Score | Criteria |
|---|---|
| 10 | Brand appears in top-10 for all 5 queries |
| 8 | Top-10 for 4 of 5 |
| 6 | Top-10 for 3 of 5 |
| 4 | Top-10 for 2 of 5 |
| 2 | Top-10 for 1 of 5 |
| 0 | Not in top-10 for any query |

**Aggregation:** Site-level. Single score for the whole site.

**Reliability note:** WebSearch is sensitive to personalization and region. Run each query 2-3 times and average. Label this factor "noisy estimate" in the final report.

**Why it matters:** Shepard scored search rank 9.4/10. AI engines use organic rank as a quality prior — the higher you rank, the more likely you are quoted.

---

### Factor 3 — Authority / brand signal (weight: 12)

**What to check (site-level):**

Three sub-checks:
1. **Wikipedia article presence** — does the brand have a Wikipedia article? (Direct search.)
2. **`sameAs` graph depth** — count entries in the homepage's Organization schema `sameAs` array. Look for: LinkedIn, Crunchbase, X/Twitter, GitHub, Facebook, Wikipedia, Wikidata.
3. **Brand mention volume** — WebSearch for the brand's full disambiguated name; count distinct domains in the top 30 results that aren't owned by the brand.

| Score | Criteria |
|---|---|
| 10 | Wikipedia article exists + 5+ sameAs entries + 15+ distinct mentioning domains |
| 7 | Two of the three thresholds met |
| 4 | One of the three thresholds met |
| 0 | None of the thresholds met |

**Aggregation:** Site-level.

**Why it matters:** Shepard scored authority 9.2/10. Brand mentions correlate 3× stronger with AI visibility than backlinks (early-2026 industry data).

---

### Factor 4 — Schema completeness (weight: 10)

**What to check (per page):**

Inspect the page's JSON-LD blocks and Microdata. For each, record presence + basic validity (required fields populated).

Required schemas vary by page type:
- **Homepage / About:** Organization (with `sameAs`, `logo`, `url`, `name`)
- **Product / pricing pages:** Product or SoftwareApplication, Breadcrumb, AggregateRating where applicable
- **Article / blog:** Article (with `author`, `datePublished`, `dateModified`, `headline`), Breadcrumb
- **FAQ pages:** FAQPage with at least 3 valid Question/Answer pairs
- **How-to / docs:** HowTo with steps

| Score | Criteria |
|---|---|
| 10 | All applicable schemas present + valid |
| 7 | Most present, some validity gaps |
| 4 | Some present, many gaps |
| 0 | No JSON-LD or only generic WebPage schema |

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Content with proper schema shows 30-40% higher visibility in AI-generated answers (industry research). AI engines use schema as ground-truth for entity disambiguation.

---

### Factor 5 — Query-answer match (weight: 10)

**What to check (per page):**

Scan headings. Does the page have at least one H2 or H3 that is question-shaped (starts with "What", "How", "Why", "When", "Where", "Who", "Can", "Is", "Does", "Should"), with a direct 40-60 word answer in the immediately following text?

| Score | Criteria |
|---|---|
| 10 | 3+ question-shaped headings, each with a clean 40-60 word direct answer |
| 7 | 1-2 question-shaped headings with clean answers |
| 4 | Question headings present but answers are buried/long/vague |
| 0 | No question-shaped headings |

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Shepard scored query-answer match 9.2/10. AI engines match user queries to question-shaped content during retrieval.

---

### Factor 6 — Recency (weight: 8)

**What to check (per page):**

Extract `lastmod` from sitemap.xml, `dateModified` from Article schema, or HTTP `Last-Modified` header. Compute age in days from today.

| Score | Criteria |
|---|---|
| 10 | Updated within 60 days |
| 7 | Updated within 6 months |
| 4 | Updated within 1 year |
| 0 | No update signal or older than 1 year |

**Aggregation:** Page-level. Score is % of priority pages with score ≥7, mapped to 0-10.

**Why it matters:** SE Ranking research found pages updated within 2 months earn 28% more AI citations. AI cites content 25.7% fresher than traditional search.

---

### Factor 7 — AI bot allow-list (weight: 8)

**What to check (site-level):**

Fetch `robots.txt`. Look for explicit allow rules for these user agents:

- `GPTBot` (OpenAI training)
- `OAI-SearchBot` (ChatGPT search)
- `ClaudeBot` (Anthropic)
- `PerplexityBot` (Perplexity)
- `Google-Extended` (Google AI)

Also check for blanket `Disallow: /` directives or `Disallow:` rules that affect critical content paths.

| Score | Criteria |
|---|---|
| 10 | All 5 bots explicitly allowed (or no Disallow rules and no `User-agent: *` Disallow) |
| 7 | 3-4 bots allowed, no explicit blocks |
| 4 | Some bots blocked, others ambiguous |
| 0 | One or more AI bots explicitly blocked via Disallow |

**Aggregation:** Site-level.

**Why it matters:** Shepard scored this 8/10. If a bot can't crawl, it can't cite — period.

---

### Factor 8 — Fan-out / topical depth (weight: 8)

**What to check (per page):**

Count internal links from the page to other pages on the same domain. Specifically, count links that point to **topically related** pages (URLs sharing a path prefix or schema-typed as the same topic cluster).

| Score | Criteria |
|---|---|
| 10 | 10+ topical internal links, clear hub-and-spoke pattern visible |
| 7 | 5-9 topical internal links |
| 4 | 2-4 topical internal links |
| 0 | 0-1 internal links, orphan page |

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Shepard scored fan-out 9.3/10. AI engines use internal link graphs to assess topical authority and find related passages.

---

### Factor 9 — Preview controls (weight: 6)

**What to check (per page):**

| Control | Pass condition |
|---|---|
| Meta description | Present, 50-160 chars |
| OpenGraph tags | og:title, og:description, og:image, og:type all present |
| Twitter Card | summary or summary_large_image present |
| `noindex` | NOT present |
| `nosnippet` | NOT present |
| `max-snippet` | NOT set to 0 or low value |

Score = (controls passed / 6) × 10.

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Shepard scored preview controls 9.2/10. Engines use these signals to decide whether and how to preview/cite the page.

---

### Factor 10 — Mobile + render health (weight: 6)

**What to check (per page):**

| Control | Pass condition |
|---|---|
| `viewport` meta | Present with `width=device-width` |
| Response time | Page returns initial HTML in <3s on a cold fetch |
| No render-blocking | Content visible without JS execution |
| No layout-shift indicators | No `font-display: block` without fallback; no large `<img>` without dimensions |

Score = (controls passed / 4) × 10.

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Mobile and render health are baseline quality signals AI engines factor into citation decisions.

---

### Factor 11 — Canonicalization (weight: 4)

**What to check (per page):**

| Control | Pass condition |
|---|---|
| `<link rel="canonical">` | Present and points to a non-redirecting URL |
| Single canonical | Only one canonical per URL (not multiple) |
| Self-referential or upstream | Canonical is either self OR a sensible upstream version |

Score = (controls passed / 3) × 10.

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Duplicate-content confusion makes AI engines hedge on which version to cite.

---

### Factor 12 — Sitemap + (optionally) llms.txt (weight: 4)

**What to check (site-level):**

| Control | Pass condition |
|---|---|
| `sitemap.xml` exists | Reachable, returns valid XML |
| `lastmod` populated | Most URLs have `lastmod` dates |
| Referenced in robots.txt | `Sitemap:` directive present |
| `llms.txt` exists | Optional; presence noted but low-weight |

Score = (sitemap controls passed / 3) × 10. **`llms.txt` presence does not move the score** — it is mentioned in the report under "Don't waste time on" because Shepard scored it 2.0/10.

**Aggregation:** Site-level.

**Why it matters:** Sitemap is the standard way to declare URL surface and freshness to crawlers. `llms.txt` is hype-heavy and evidence-light; do not over-invest.

---

## Aggregation summary

| Score | Aggregation level | How |
|---|---|---|
| 1, 4, 5, 6, 8, 9, 10, 11 | Page-level | Average across priority pages, then weighted average across all priority + non-priority pages with 2× weight for priority |
| 2, 3, 7, 12 | Site-level | Single value for the whole site |

**Site readiness score** = weighted sum of page-level averages + site-level scores, all multiplied by their weights, summed, divided by 10.

## Anti-pattern callout

Do NOT inflate this score by:
- Installing an llms.txt (Shepard 2.0/10 — minimal lift)
- Spraying schema on every page without entity discipline (causes validity gaps that hurt Factor 4)
- Blocking competitor bots while allowing AI bots — common WAF mistake; check for inadvertent blocks of `Google-Extended` and `OAI-SearchBot`
