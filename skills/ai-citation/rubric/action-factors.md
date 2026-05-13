# Citability Action Factors

The Action Score (0-100) measures what the team can **fix this week** to improve citation likelihood. Weights are derived from Princeton/Georgia Tech/IIT Delhi GEO research (2024).

Each factor is scored **0-100**, then multiplied by its weight. Sum all weighted scores, divide by 100, → **0-100**.

```
Action Score = (Σ factor_score × weight) / 100
```

Weights sum to 100.

---

## The 8 factors

### Factor 1 — Passage citability (weight: 25)

**What to check (per page):**

Segment each page's main content at H2/H3 boundaries → score each block using `rubric/block-rubric.md` (6 criteria, pass threshold ≥70).

| Score | Criteria |
|---|---|
| 100 | ≥80% of blocks score ≥70 |
| 80 | 60-79% of blocks score ≥70 |
| 60 | 40-59% of blocks score ≥70 |
| 40 | 20-39% of blocks score ≥70 |
| 0 | <20% of blocks score ≥70 |

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Princeton's GEO study found definition-first, fact-rich passages get 30-115% more citations. This is the single biggest fixable lever.

---

### Factor 2 — Brand surface area off-site (weight: 20)

**What to check (site-level):**

Use the Brand Surface Area Map sub-rubric in `rubric/brand-surface-area.md` to score 6 off-site platforms (Wikipedia, Reddit, YouTube, G2/Capterra, Quora, comparison/listicles). SaaS-only platforms are skipped for non-SaaS brands.

Average the scores across applicable platforms × 10 → 0-100.

**Aggregation:** Site-level.

**Why it matters:** Brand mentions correlate 3× stronger with AI visibility than backlinks. Each platform skews different AI engines (Reddit → Perplexity, Wikipedia → ChatGPT).

---

### Factor 3 — Statistical density (weight: 12)

**What to check (per page):**

Count specific statistics in the main content. A statistic is:
- A percentage with a named source ("73% of marketers report, per HubSpot 2026...")
- A dollar amount with a named context ("$4,500 average monthly cost")
- A named study citation ("according to the 2026 SE Ranking report")
- A specific count ("integrates with 340+ tools")
- A timeframe with measurement ("implementation takes 6-8 weeks")

Compute stats-per-500-words.

| Score | Criteria |
|---|---|
| 100 | 5+ stats per 500 words |
| 80 | 3-4 stats per 500 words |
| 60 | 2 stats per 500 words |
| 40 | 1 stat per 500 words |
| 0 | <1 stat per 500 words |

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** Princeton's GEO study: adding statistics increases citation by 40%.

---

### Factor 4 — Authority quotations (weight: 10)

**What to check (per page):**

Count direct quotations from named external authorities. A qualifying quote must have:
1. A direct quote (in quotation marks)
2. A named source person OR a named study/organization
3. A credential or affiliation (title, organization, year)

| Score | Criteria |
|---|---|
| 100 | 2+ qualifying quotes per priority page on average |
| 80 | 1 qualifying quote per priority page on average |
| 60 | Quotes present on some pages, missing on others |
| 40 | A few quotes total across the site |
| 0 | No qualifying quotes |

**Aggregation:** Site-level (computed as average across priority pages).

**Why it matters:** IIT Delhi GEO study (2024) — adding authority quotations increases citation rate by 115% in certain categories.

---

### Factor 5 — Freshness cadence (weight: 10)

**What to check (site-level):**

% of priority pages with `lastmod` or `dateModified` within the past 60 days.

| Score | Criteria |
|---|---|
| 100 | ≥80% of priority pages updated within 60 days |
| 80 | 60-79% |
| 60 | 40-59% |
| 40 | 20-39% |
| 0 | <20% |

**Aggregation:** Site-level.

**Why it matters:** Pages updated within 2 months earn 28% more AI citations (SE Ranking research). This is fixable in a single sprint.

---

### Factor 6 — Schema actionability (weight: 10)

**What to check (site-level):**

Count high-leverage missing schemas the team could ship this week. A missing schema is "high-leverage" if:
- It is the standard schema type for the page (e.g., FAQPage on an FAQ, Article on a blog post)
- Required fields are populatable from existing page content (no new content authoring needed)
- It is supported by all major AI engines (Organization, Article, Product, FAQPage, HowTo, Breadcrumb, sameAs)

| Score | Criteria |
|---|---|
| 100 | 0 high-leverage gaps — schema is fully covered |
| 80 | 1-2 high-leverage gaps |
| 60 | 3-4 high-leverage gaps |
| 40 | 5-7 high-leverage gaps |
| 0 | 8+ high-leverage gaps |

**Inverse direction:** higher gaps = lower score. The score reflects "how complete is the actionable schema layer."

**Aggregation:** Site-level.

**Why it matters:** Schema is the cheapest, highest-leverage move available. Most teams have huge gaps here.

---

### Factor 7 — Question-shaped headings (weight: 8)

**What to check (per page):**

% of H2 and H3 headings that are question-shaped (start with What/How/Why/When/Where/Who/Can/Is/Does/Should).

| Score | Criteria |
|---|---|
| 100 | ≥50% of H2/H3 are question-shaped |
| 80 | 30-49% |
| 60 | 15-29% |
| 40 | 5-14% |
| 0 | <5% or no H2/H3 |

**Aggregation:** Page-level. Average across priority pages.

**Why it matters:** AI engines match user queries to question-shaped headings during retrieval. This is a cheap restructure with measurable lift.

---

### Factor 8 — Internal answer architecture (weight: 5)

**What to check (site-level):**

Does the site have a clear hub-and-spoke topical cluster structure? Specifically:
- A "pillar" page that defines the core topic comprehensively
- 5+ "spoke" pages that dive deep into sub-topics
- Bidirectional internal linking (pillar links to spokes, spokes link back to pillar and to each other)

| Score | Criteria |
|---|---|
| 100 | 3+ pillar+spoke clusters, well-linked |
| 80 | 2 clusters or one strong cluster |
| 60 | 1 cluster, partially linked |
| 40 | Some related content, no clear cluster structure |
| 0 | No discernible topical clustering |

**Aggregation:** Site-level.

**Why it matters:** Pillar-spoke architecture signals topical authority and gives AI engines multiple anchor points within the same topic.

---

## Aggregation summary

| Score | Aggregation level | How |
|---|---|---|
| 1, 3, 4, 6, 7, 8 | Page-level | Average across priority pages |
| 2, 5 | Site-level | Single value for the whole site |

**Site action score** = weighted sum of page-level averages + site-level scores, all multiplied by their weights, summed, divided by 100.

## Time-to-fix estimates (used by walkthrough prioritization)

| Factor | Typical time to fix on a real site |
|---|---|
| 1. Passage citability (top 3 pages) | 2-4 hours per page |
| 2. Brand surface area | Variable: G2 listing = 1 hour; Wikipedia = weeks; Reddit seeding = ongoing |
| 3. Statistical density | 1-2 hours per page |
| 4. Authority quotations | 30 min per page if sources are known |
| 5. Freshness cadence | 1 hour per page; cadence is a process change |
| 6. Schema actionability | 15-60 min per missing schema |
| 7. Question-shaped headings | 1 hour per page (restructure) |
| 8. Internal answer architecture | 1-2 weeks for a real cluster build-out |

The walkthrough uses (impact × ease) to surface the highest-leverage low-effort moves first.
