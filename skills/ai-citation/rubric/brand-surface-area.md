# Brand Surface Area Map

Feeds **Action Score Factor 2** (weight 20). Measures how visible the brand is across the off-site platforms AI engines actually pull from.

Score each applicable platform 0/5/10. Skip platforms that don't apply to the brand's category (e.g., G2/Capterra only apply to SaaS). Average across applicable platforms → 0-10. Multiply by 10 → 0-100 for Action Score input.

---

## How to detect brand category (SaaS vs. non-SaaS vs. local-services vs. ecom vs. media)

Use the brand's `category_description` from `preferences.json`. Keyword heuristics:

| Category | Keywords in `category_description` |
|---|---|
| **SaaS** | software, SaaS, platform, app, tool, dashboard, automation, "service provider for", "for teams", "B2B" |
| **E-commerce** | shop, store, products, ecommerce, marketplace, retailer |
| **Media / publisher** | blog, magazine, newsletter, publication, content, media company |
| **Local services** | local, near me, city-specific, contractor, restaurant, salon, agency in <city> |
| **Other** | falls through to default — score all 6 platforms |

The skill auto-detects category from the description; user can override with `--category` flag.

---

## Platform applicability matrix

| Platform | SaaS | Ecom | Media | Local | Other |
|---|---|---|---|---|---|
| Wikipedia | ✓ | ✓ | ✓ | ✗ (usually) | ✓ |
| Reddit | ✓ | ✓ | ✓ | ✓ | ✓ |
| YouTube | ✓ | ✓ | ✓ | ✓ | ✓ |
| G2 / Capterra | ✓ | ✗ | ✗ | ✗ | ✗ |
| Quora | ✓ | ✓ | ✓ | ✓ | ✓ |
| Comparison / listicles | ✓ | ✓ | ✓ | ✓ | ✓ |

Local-services brands skip Wikipedia (notability threshold too high for most). Non-SaaS skip G2/Capterra.

---

## Scoring rubric (per platform)

### Wikipedia

**How to check:** Direct search for the brand's full disambiguated name on Wikipedia. Also check Wikidata for an entity.

| Score | Criteria |
|---|---|
| 10 | Full article with reliable third-party citations, regularly updated, no notability-flag templates |
| 5 | Stub article OR article exists but has notability/sourcing-flag templates OR Wikidata entity but no English Wikipedia article |
| 0 | No article and no Wikidata entity |

**AI engine impact:** ChatGPT cites Wikipedia 47.9% of the time per early-2026 data. This is the single highest-leverage off-site platform for ChatGPT visibility.

### Reddit

**How to check:** `site:reddit.com "<brand full name>"` AND `site:reddit.com "<brand + category phrasing>"`. Count distinct threads in past 12 months that pass the category-filter check.

| Score | Criteria |
|---|---|
| 10 | 20+ organic threads in past 12 months; active community discussion; brand mentioned in answers to category questions |
| 5 | 1-5 organic threads in past 12 months OR a brand-owned subreddit with low engagement |
| 0 | Zero threads found OR all mentions are spam/self-promotional |

**AI engine impact:** Perplexity cites Reddit 46.7% of the time. Critical for Perplexity visibility.

**Anti-pattern:** Do NOT seed Reddit via spam, sock-puppet accounts, or paid posting. The skill's walkthrough has explicit ethical guidance for the "seed Reddit presence" move (move #3 in `move-catalog/moves.md`).

### YouTube

**How to check:** `site:youtube.com "<brand>"`. Look for:
- Brand-owned channel + view counts on recent videos
- Third-party videos discussing the brand (reviews, tutorials, comparisons)
- Total brand mentions across discovered videos

| Score | Criteria |
|---|---|
| 10 | Brand channel with 3+ videos averaging 1k+ views OR 5+ third-party videos discussing brand |
| 5 | Some videos by others OR low-view brand channel (under 500 views per video average) |
| 0 | No videos discovered |

**AI engine impact:** YouTube overtook Reddit as the most-cited social platform in early 2026. Cross-engine value.

### G2 / Capterra (SaaS only)

**How to check:** Direct lookup on g2.com and capterra.com for the brand. Also TrustRadius and Software Advice as secondary checks.

| Score | Criteria |
|---|---|
| 10 | Active G2 listing with 20+ reviews in past 12 months, claimed profile, complete category placement |
| 5 | Listing exists but stale (under 20 reviews OR no reviews in past 12 months) OR Capterra-only presence |
| 0 | No listing on any major review platform |

**AI engine impact:** G2 is the most-cited SaaS review platform across ChatGPT, Perplexity, and Google AI Overviews.

**Anti-pattern:** Do NOT solicit fake reviews or compensate reviewers without disclosure. G2/Capterra both have detection mechanisms and will deprioritize or remove offending listings.

### Quora

**How to check:** `site:quora.com "<brand>"` and `site:quora.com "<category phrasing>"`. Look for the brand cited in answers to category questions.

| Score | Criteria |
|---|---|
| 10 | Brand cited in answers to 5+ category questions; high-upvote answers; brand-affiliated experts answering |
| 5 | Brand has some Q&A presence OR brand-owned space with low activity |
| 0 | Zero mentions found |

**AI engine impact:** ChatGPT and Perplexity both pull from Quora for niche category questions.

### Comparison / listicle sites

**How to check:** `"best <category>" "<brand>"` AND `"top <category> tools" "<brand>"`. Count appearances in major third-party listicles.

| Score | Criteria |
|---|---|
| 10 | Featured in major listicles for the category (e.g., Forbes, TechCrunch, leading category-specific publications) |
| 5 | Listed in 1-3 "best of" articles on smaller publications or aggregator sites |
| 0 | Absent from all category listicles |

**AI engine impact:** Listicles are the canonical "recommendation" surface AI engines mine for comparison answers.

---

## Disambiguation discipline (critical for short brand names)

For brands with ambiguous names (under 3 tokens, OR common English words, OR lacking an uncommon proper noun):

1. ALL platform searches MUST use disambiguated queries from `preferences.json` (full company name + category phrasing + handles)
2. For every result, check whether category-description tokens appear in result title/snippet. Drop if not.
3. Report the dropped count transparently in the report:

> "Off-site probe used 5 query variants and dropped 23 results that didn't match the category context (e.g., 'Easy Mac', 'Easy Off') as false positives."

### Worked example — `"Easy"` (payment service provider for SaaS)

Brand identity recorded in `preferences.json`:

```json
{
  "brand": {
    "primary": "Easy",
    "full_name": "Easy Labs Inc.",
    "category_phrasings": ["Easy payment processor", "Easy PSP"],
    "handles": ["@useeasy", "easy.com"],
    "category_description": "payment service provider for SaaS companies"
  }
}
```

**Reddit probe — queries actually issued:**

```
site:reddit.com "Easy Labs Inc."               # weight 1.0
site:reddit.com "Easy" payment processor       # weight 0.8
site:reddit.com "Easy PSP"                      # weight 0.7
site:reddit.com "easy.com"                      # weight 0.7
site:reddit.com @useeasy                        # weight 0.7
```

**Results kept (category tokens — payment / SaaS / PSP — present in snippet):**
- "Anyone tried Easy Labs for payment processing?" (r/SaaS) — kept, weight 1.0
- "Easy PSP vs Stripe?" (r/startups) — kept, weight 0.7

**Results dropped (no category tokens in snippet — false positives):**
- "Easy Mac is the worst pasta" (r/food)
- "Easy Off oven cleaner review" (r/CleaningTips)
- "Easy mode in Dark Souls" (r/gaming)
- "Easy" English subtitles for X movie (r/movies)
- ... 19 more

**Transparency note printed in report:**

> "Off-site probe issued 5 query variants across 6 platforms and dropped 23 results that did not match the category context (payment / SaaS / PSP). Examples of dropped: 'Easy Mac', 'Easy Off', 'Easy Mode'. Full kept-list saved to snapshot."

---

## What the BSAM does NOT measure

- **Sentiment of mentions** — surfaced in the report as decorative context only; not factored into the numerical score (sentiment analysis is noisy and not in scope for v1)
- **Backlinks** — different measure; the report can note backlink count from external SEO tools if available, but BSAM is about mention surface specifically
- **Social media engagement** (likes, shares, follows) — engagement metrics aren't reliable AI citation signals; mention frequency is

---

## Per-platform output in the report

The audit report renders BSAM as a radar chart across the 6 platforms (or the 5 applicable ones). Each gap (0 score) becomes a candidate recommendation in the prioritized action plan:

- **Wikipedia 0 → 5** = "Create a Wikipedia article (notability check required)" — move #6 in catalog
- **Reddit 0 → 5** = "Seed Reddit presence in 3 relevant subreddits" — move #3
- **YouTube 0 → 5** = "Publish 3 explainer videos for top categories" — move #9
- **G2 0 → 5** = "Get listed on G2 / Capterra + claim profile" — move #4
- **Comparison 0 → 5** = "Outreach to top-3 third-party listicles" — move #10
