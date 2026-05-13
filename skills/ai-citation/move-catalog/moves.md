# Move Catalog — AI Citation Growth Hacks

Twelve moves the skill picks from when running the walkthrough. For each audit, the skill selects the top 5-7 based on (impact × ease) given the audit findings — skipping moves the site already has.

Each move below has the same structure:
1. **What** — one sentence
2. **Why** — research-cited reason
3. **Impact / Effort / Source**
4. **Concrete steps** — numbered, executable
5. **Templates** — copy-paste-able code or copy
6. **Real examples** — sites that did this well
7. **Common pitfalls** — what not to do
8. **How to verify** — what the next audit will show if successful
9. **Auto-close detector** (for delta tracking)

---

## Move 1 — Rewrite top-3 high-traffic pages with definition-first openings + stats per 150 words

**What:** Take the three pages with the most traffic (or organic impressions if GSC opted in) and rewrite the opening of each content block to lead with a definition + a specific stat with named source.

**Why:** Princeton/Georgia Tech GEO study (2024) — definition-first + statistical density together yield 30-115% citation lift.

| Impact | Effort | Source |
|---|---|---|
| High | Low (2-4 hours per page) | Princeton/GT GEO |

### Concrete steps

1. Pull the top-3 pages by traffic. If GSC opt-in: use Impressions in last 90 days. Else: use depth-from-homepage as proxy or ask user.
2. For each page, identify every content block (segment at H2/H3).
3. For each block scoring below 70 in the block rubric, rewrite the opening 2 sentences using the pattern below.
4. Add ≥1 specific stat with named source per 150 words. Use the **statistical-density template** below.
5. Ensure no leading pronouns in any block's first 60 words.

### Templates

**Definition-first opening pattern:**

```
<Subject> is <definition — 1 short clause>. <Specific stat with named source backing the definition>. <One sentence adding texture or contrast>.
```

**Example (transforming a real intro):**

```
BEFORE: "If you've been wondering how to get more leads, you're not alone.
There are many strategies out there. Let's dive in."

AFTER: "Inbound lead generation is the process of attracting prospects via
content rather than outbound prospecting. According to HubSpot's 2026 State
of Marketing report, inbound generates 54% more leads per dollar than
outbound. Companies adopting an inbound-first approach typically take 6-9
months to see leads-to-revenue parity."
```

**Statistical-density template (drop into any block):**

```
According to <named source> <year>, <metric> = <specific number>.
```

### Real examples

- HubSpot's own pricing/blog pages — every section opens with a definition
- Stripe Press — every product page leads with a one-sentence-what-it-is + a specific use case stat
- Cloudflare Learning Center — definition-first across hundreds of glossary entries

### Common pitfalls

- ❌ Adding "fluff" stats that don't directly support the claim ("73% of marketers like coffee" — irrelevant)
- ❌ Citing "studies show" without naming the study
- ❌ Putting the definition in sentence 3 instead of sentence 1 — defeats the purpose
- ❌ Rewriting all pages on the site — focus on the top 3 first; the lift compounds from there

### How to verify

Next audit will show those pages' passage_score moving up by ≥15 points and the % of blocks passing the 70+ threshold increasing.

### Auto-close detector

`rewrite-top-3-pages-definition-first`: re-score the named URLs; if average passage_score on those pages went up ≥15 pts → mark `closed`.

---

## Move 2 — Add FAQ schema to top-10 pages

**What:** Add `FAQPage` JSON-LD with 3-5 question/answer pairs to the top 10 high-traffic pages. Use question-shaped H2/H3 already on the page where possible; add if absent.

**Why:** Schema completeness directly drives Readiness Factor 4 (weight 10) AND it lets AI engines match user queries to your specific answers.

| Impact | Effort | Source |
|---|---|---|
| High | Low (15-30 min per page) | Shepard factors 4 + 5 |

### Concrete steps

1. List the top 10 pages by traffic (or priority pages from audit).
2. For each page, identify 3-5 natural questions the page answers. If question-shaped H2/H3 already exist, use those.
3. For each question, ensure the immediately-following text is a clean 40-60 word direct answer.
4. Generate JSON-LD using the template below.
5. Place the JSON-LD in `<head>` or just before `</body>`.
6. Validate using Google's Rich Results Test or `https://validator.schema.org/`.

### Templates

**FAQPage JSON-LD:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is <topic>?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "<40-60 word direct answer that matches the on-page content>"
      }
    },
    {
      "@type": "Question",
      "name": "How does <topic> work?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "<40-60 word direct answer>"
      }
    }
  ]
}
</script>
```

**Critical:** The `acceptedAnswer.text` MUST match content visible on the page (Google penalizes hidden FAQ content).

### Real examples

- Mailchimp's marketing glossary — every page has FAQPage schema with 3-5 questions
- Notion's docs — same pattern; pages rank for both the topic and the questions

### Common pitfalls

- ❌ Adding FAQ schema for questions the page doesn't actually answer in visible content
- ❌ Stuffing 20 questions into one page (Google's threshold is ~5-7 before perceived spam)
- ❌ Generating fake/marketing-y questions ("What makes our product the best?") — AI engines deprioritize these
- ❌ Forgetting to validate after deployment — broken JSON-LD scores 0

### How to verify

Next audit will show: Readiness Factor 4 (Schema completeness) jumping; Action Factor 7 (question-shaped headings) jumping if questions were added.

### Auto-close detector

`add-faq-schema-to-top-10-pages`: re-check JSON-LD on the 10 named URLs; if all 10 now have valid FAQPage schema → `closed`.

---

## Move 3 — Seed Reddit presence in 3 relevant subreddits (educational, NOT spam)

**What:** Build legitimate, organic presence on 3 subreddits where your category is actively discussed. Goal: get organically mentioned in answers without astroturfing.

**Why:** Perplexity cites Reddit 46.7% of the time. Zero Reddit presence = near-zero Perplexity visibility.

| Impact | Effort | Source |
|---|---|---|
| High | Medium (months of consistent engagement) | Perplexity citation share data |

### Concrete steps (the ethical playbook)

1. Identify 3 subreddits where your category is discussed (e.g., r/SaaS, r/marketing, r/startups for B2B SaaS).
2. **Build an account with real history first.** Use a real account that participates in unrelated content for 30+ days before mentioning your brand. New accounts that post brand content get filtered as spam by both moderators and Reddit's own spam systems.
3. **Comment first, post later.** Spend 2 weeks answering questions in the subreddit without mentioning your brand. Build karma and pattern recognition with the community.
4. When your brand is genuinely relevant to a question, mention it — **disclosing your affiliation** in the comment. Format: "Disclosure: I work at [Brand]. <substantive answer that includes the brand only where relevant>."
5. Continue contributing non-brand educational content alongside brand mentions at a 5:1 ratio minimum.

### Templates (sanitized — disclosure-first)

**The disclosure-first comment template:**

```
**Disclosure: I work at <Brand>.**

<3-5 sentence substantive answer to the question, including specific data,
real tradeoffs, and at least one mention of a competitor or alternative.>

<Optional: link to brand resource ONLY if directly relevant to the question>
```

### Real examples

- Linear (the project management SaaS) — engineers from Linear regularly answer in r/SaaS and r/startups with disclosed affiliation; brand gets cited in Perplexity for project management queries
- Vercel founders/employees — disclosed-affiliation answers in r/nextjs

### Common pitfalls (these will TANK your score and brand)

- ❌ Astroturfing — sock-puppet accounts, paid commenters, undisclosed-affiliation posts. Reddit's spam detection and community moderators catch these; ban risk is real.
- ❌ Drive-by promo — joining a subreddit to drop your link and leave. Filtered automatically.
- ❌ Generic AI-generated comments. Easily detected by community members and downvoted into oblivion.
- ❌ Posting the same content across multiple subreddits within hours. Cross-posting flags trigger ban.

**The skill refuses to generate astroturfing scripts.** If asked, return: "I won't generate inauthentic Reddit content. The legitimate playbook is in Move 3 of the catalog."

### How to verify

Next audit will show Reddit_score moving 0→5 or 5→10 as organic mentions accumulate. Set realistic timeline: 3-6 months of consistent engagement.

### Auto-close detector

`seed-reddit-presence`: re-run off-site probe; if Reddit_score moved 0→5 or 5→10 → `closed`.

---

## Move 4 — Get listed on G2 / Capterra + claim profile (SaaS only)

**What:** Create or claim the brand's listing on G2 and Capterra. Solicit real reviews from existing customers through approved channels.

**Why:** G2 is the most-cited SaaS review platform across ChatGPT, Perplexity, and Google AI Overviews. A claimed, active G2 profile is the cheapest off-site win for SaaS brands.

| Impact | Effort | Source |
|---|---|---|
| High | Low (1-2 hours to set up, ongoing for reviews) | Discovered Labs / industry research |

### Concrete steps

1. Check if a listing already exists on g2.com and capterra.com (also TrustRadius, Software Advice).
2. If existing: claim the profile (G2: "Claim this profile" button; verification via business email).
3. If not: create a new listing with category placement, description, features, pricing, screenshots.
4. Set up the customer review solicitation flow:
   - Add a "Review us on G2" CTA in your in-app NPS responses for promoters (NPS 9-10)
   - Add a CTA in post-onboarding emails after 30/60/90 days
   - Offer G2 gift card incentives for verified reviewers (G2 explicitly allows this with proper disclosure)
5. Aim for 20+ reviews in the first 6 months to clear the "established" threshold.

### Templates

**Customer review solicitation email (G2):**

```
Subject: Quick favor — would you share your <Brand> experience on G2?

Hi <Name>,

You mentioned in our last NPS survey that you'd recommend <Brand> to a peer.
That feedback means a lot — and other teams in <industry> looking at us would
benefit from hearing it directly.

Would you take 5 minutes to drop a review on G2? We'll send you a $25 gift
card as a thank-you (G2 allows this with disclosure on the review).

[Direct G2 review link]

Thanks,
<Sender>
```

### Real examples

- Linear, Notion, Vercel, Stripe — all have active G2/Capterra profiles with hundreds of reviews
- Their AI citations for "best <category>" queries reference these listings repeatedly

### Common pitfalls

- ❌ Fake reviews. G2 has detection — fake-flagged accounts lose all reviews and rankings.
- ❌ Soliciting reviews without disclosure. Both G2 and FTC have disclosure requirements.
- ❌ Only soliciting positive reviews (cherry-picking). G2 catches review-rate spikes from one channel.
- ❌ Ignoring negative reviews. Engage publicly; the response signal matters for buyers AND AI engines.

### How to verify

Next audit will show G2_score moving 0→5 (listing exists) or 5→10 (20+ reviews active).

### Auto-close detector

`get-listed-on-g2-capterra`: re-fetch G2/Capterra; if claimed listing with ≥1 review present → `closed` (but `partial` if reviews <20).

---

## Move 5 — Publish brand `sameAs` graph in Organization schema

**What:** Update the homepage's Organization JSON-LD to include a complete `sameAs` array listing every brand-controlled external identity: LinkedIn, Crunchbase, X/Twitter, GitHub, Facebook, Wikipedia, Wikidata.

**Why:** Authority signal Factor 3 weights sameAs depth at 12. This is the single cheapest entity-disambiguation signal for AI engines.

| Impact | Effort | Source |
|---|---|---|
| Medium | Low (30 minutes total) | Shepard factor 3 / sanitized Hobo_Web hack |

### Concrete steps

1. Find your homepage's Organization JSON-LD block (or create one if missing).
2. Add a `sameAs` array with every external identity you control.
3. Ensure each URL is canonical (no tracking params, no redirects).
4. Validate using `https://validator.schema.org/`.

### Templates

**Organization with sameAs:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Easy Labs Inc.",
  "url": "https://itseasy.co",
  "logo": "https://itseasy.co/logo.png",
  "description": "Payment service provider for SaaS companies",
  "sameAs": [
    "https://www.linkedin.com/company/easy-labs",
    "https://twitter.com/useeasy",
    "https://github.com/easylabs",
    "https://www.crunchbase.com/organization/easy-labs",
    "https://en.wikipedia.org/wiki/Easy_Labs",
    "https://www.wikidata.org/wiki/Q123456789"
  ]
}
</script>
```

### Real examples

- Stripe, Vercel, Linear — all maintain comprehensive sameAs arrays on homepage Organization schema
- Their AI engine entity recognition is correspondingly tight (consistent attribution across answers)

### Common pitfalls

- ❌ Including URLs you don't actually control (just a profile mentioning you)
- ❌ Stale URLs that 404 or redirect — these hurt validity score
- ❌ Multiple Organization blocks with conflicting sameAs arrays
- ❌ The "blank-page-only-schema" growth hack (Hobo_Web's original 36-hour Perplexity citation experiment). This skill MENTIONS but DOES NOT TEACH that approach: it works short-term, ages poorly, ethics-debt accrues, and AI engines are increasingly filtering thin-content pages.

### How to verify

Next audit will show Readiness Factor 3 (Authority/brand signal) score jumping.

### Auto-close detector

`publish-sameas-graph`: re-fetch homepage Organization JSON-LD; if sameAs array has ≥5 valid entries → `closed`.

---

## Move 6 — Stand up a Wikipedia article (if eligible)

**What:** Create a Wikipedia article about the brand, properly sourced and notable-enough to survive deletion review.

**Why:** ChatGPT cites Wikipedia 47.9% of the time. A Wikipedia article is the single highest-leverage ChatGPT visibility move.

| Impact | Effort | Source |
|---|---|---|
| Very High | High (notability is the hard gate) | ChatGPT citation share |

### Eligibility check (the notability gate)

The brand qualifies for a Wikipedia article ONLY if it has:
- **5+ pieces of significant, independent coverage** in reliable third-party sources (major publications, not press releases, not company-owned content)
- A **multi-year operating history** (rarely accepted under 2 years)
- A **distinguishable category position** (not "yet another <category> tool")

If any of these fails, this move is **disqualified** and the skill substitutes a lower-tier move (a strong Crunchbase profile + brand mentions on `sameAs` from related Wikipedia entities for now, with a "circle back when notability threshold is met" note).

### Concrete steps (assuming eligible)

1. **Audit existing coverage.** Pull up the brand's coverage in major publications (TechCrunch, Forbes, NYT, WSJ, trade publications). You need 5+ pieces.
2. **Choose a writer.** Articles written by the brand itself get tagged with conflict-of-interest banners. Hire a Wikipedia-experienced contractor or pitch a third-party writer interested in the category.
3. **Draft following the Wikipedia Manual of Style.** Neutral tone, no marketing language, every claim cited.
4. **Submit via Articles for Creation (AfC).** Don't publish directly. AfC reviewers are more lenient with first-time submissions and provide feedback.
5. **Respond to revision requests.** Most articles need 1-3 rounds before publication.

### Templates

**Wikipedia article skeleton (neutral tone):**

```wiki
'''<Brand>''' is a <category> founded in <year> and headquartered in
<location>. As of <date>, the company has raised <funding> and serves
<customer count or industries>. <Brand> was founded by <founders> who
previously worked at <prior companies>.<ref>...</ref>

== History ==
[Founding, milestones, funding rounds — every claim cited]

== Products and services ==
[Neutral description of what the company offers — every claim cited]

== Reception ==
[Coverage from third-party sources — direct quotes with citations]
```

### Real examples

- Notion's Wikipedia article — sourced, neutral, robust
- Linear's Wikipedia article — passed AfC after one revision round

### Common pitfalls

- ❌ Writing the article yourself with marketing language — gets tagged or deleted
- ❌ Citing your own blog posts as "third-party sources" — disqualified
- ❌ Submitting before notability threshold is met — article gets nominated for deletion within days
- ❌ Aggressive editing after publication ("conflict of interest" editor warnings cascade)

### How to verify

Next audit will show Wikipedia_score moving from 0 to 5 (stub) or 10 (full article with citations).

### Auto-close detector

`stand-up-wikipedia-article`: re-check Wikipedia search for brand; if article exists and is not a redirect/stub → `closed`.

---

## Move 7 — Add AI bots to robots.txt allow-list

**What:** Update robots.txt to explicitly allow GPTBot, OAI-SearchBot, ClaudeBot, PerplexityBot, and Google-Extended.

**Why:** Many sites accidentally block AI bots via aggressive WAF rules or blanket `User-agent: *` disallows. Allowing AI bots is the lowest-effort, lowest-risk move possible.

| Impact | Effort | Source |
|---|---|---|
| Low (but trivial) | Trivial (5 minutes) | Shepard factor 7 |

### Concrete steps

1. Pull current `robots.txt`.
2. Add the allow-list block below.
3. Deploy.
4. Verify each bot's allow rule using `https://www.robotstxt.org/checker.html` or curl.

### Templates

**Append to `robots.txt`:**

```
# AI crawlers — explicit allow

User-agent: GPTBot
Allow: /

User-agent: OAI-SearchBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /
```

### Real examples

Most major SaaS companies have already updated their robots.txt — check your favorites for the canonical version.

### Common pitfalls

- ❌ Adding an `Allow: /` without a corresponding `User-agent:` line above it (rule doesn't apply)
- ❌ Blocking AI bots intentionally to "protect content" — this kills citation visibility for zero defensive benefit (the content is already public)
- ❌ Forgetting to also remove competing `Disallow: /` rules elsewhere in the file

### How to verify

Next audit will show Readiness Factor 7 (AI bot allow-list) score jumping to 10.

### Auto-close detector

`add-bots-to-robots-allowlist`: re-fetch robots.txt; if all 5 bots have explicit Allow rules → `closed`.

---

## Move 8 — Refresh top-5 evergreen pages with current dates + updated stats

**What:** Update the top 5 evergreen content pages with current-year dates, refreshed statistics, and a published `dateModified`.

**Why:** Pages updated within 60 days earn 28% more AI citations (SE Ranking research). This is a process change, not a one-time fix.

| Impact | Effort | Source |
|---|---|---|
| Medium | Low (1 hour per page) | SE Ranking research |

### Concrete steps

1. Identify the top 5 evergreen pages (pricing, product overviews, top-traffic blog posts, glossary entries).
2. For each, audit stats: find anything dated more than 6 months old; replace with current sources.
3. Update `<title>` and H1 if they include years (e.g., "The 2025 Guide to X" → "The 2026 Guide to X").
4. Set `dateModified` in Article schema OR push a meaningful sitemap.xml `lastmod` update.
5. Add a visible "Last updated" line in the page header.
6. Set up a quarterly refresh cadence (calendar reminder via the skill's delta tracking nudge).

### Templates

**Visible "Last updated" pattern:**

```html
<p class="last-updated">Last updated <time datetime="2026-05-13">May 13, 2026</time></p>
```

**Article schema dateModified:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "<Page title>",
  "datePublished": "2024-03-15",
  "dateModified": "2026-05-13",
  "author": {"@type": "Person", "name": "<Author>"}
}
</script>
```

### Real examples

- Backlinko's evergreen guides — visible "Last updated" line, dateModified always within 60 days
- Cloudflare Learning Center — refreshed quarterly with stat updates

### Common pitfalls

- ❌ "Updating" without actually changing content (just bumping the date). AI engines (and Google) detect this.
- ❌ Refreshing only sitemap.xml `lastmod` without touching `dateModified` schema — mixed signals.
- ❌ Refreshing without finding actually new stats — leaves stale stats with new dates, which is worse than leaving the original date alone.

### How to verify

Next audit will show Recency factor score moving up and the % of priority pages updated-within-60-days increasing.

### Auto-close detector

`refresh-top-5-evergreen-pages`: re-fetch the 5 named URLs; if `lastmod`/`dateModified` on all 5 is within 60 days → `closed`.

---

## Move 9 — Build a YouTube presence (3 explainer videos)

**What:** Publish 3 explainer videos on YouTube covering top categories. Brand-owned channel, complete metadata, transcripts published.

**Why:** YouTube overtook Reddit as the top-cited social platform in early 2026. Cross-engine value.

| Impact | Effort | Source |
|---|---|---|
| High | High (1-2 weeks of production) | Early-2026 platform-citation data |

### Concrete steps

1. Identify 3 top categories where your brand has expertise (look at top-traffic blog topics).
2. Script a 5-8 minute explainer per category. Use the structure: **Hook → Problem → Definition → Walkthrough → Recap**.
3. Record. Use whatever production level you can sustain — consistent voice/quality matters more than cinematic polish.
4. Upload with:
   - **Title:** Question-shaped, includes the category phrase
   - **Description:** First 200 chars are a definition + key stat (AI engines often extract from these)
   - **Tags:** Category terms + variants
   - **Captions / transcripts:** Auto-generate then human-review; transcripts are what AI engines actually extract
   - **End screen:** Link to your site's relevant pillar page
5. Submit videos to YouTube as a brand-owned channel; complete the About page with sameAs links back to your site.

### Templates

**YouTube description template:**

```
<Category topic> is <definition in 1-2 sentences>. According to <named source>,
<key stat>. In this video, we walk through:

00:00 — The core concept
01:30 — Three common pitfalls
04:00 — A walkthrough example
07:00 — Recap and next steps

🔗 Read the full guide: <link to pillar page>

About the channel:
<Brand> helps <audience> <accomplish goal>. Visit <domain>.
```

### Real examples

- Cloudflare's developer YouTube channel
- HubSpot's marketing-academy YouTube videos — high citation rate in AI answers

### Common pitfalls

- ❌ Uploading without transcripts — AI engines can't extract anything; you've made a video for humans, not for machines
- ❌ Short Shorts (under 60 seconds) — these rarely get cited; AI engines pull from longer, structured content
- ❌ Stuffing keywords in title/description — algorithm and human viewers both deprioritize
- ❌ Generic stock-footage videos with VO — AI engines can't extract distinctive content from these

### How to verify

Next audit will show YouTube_score moving 0→5 or 5→10 as videos accumulate views.

### Auto-close detector

`build-youtube-presence`: re-run YouTube probe; if brand channel exists with ≥3 videos averaging 100+ views → `closed`.

---

## Move 10 — Get included in top-3 third-party listicles ("best <category>")

**What:** Outreach to the editors of top-3 "best <category>" articles in your niche. Pitch your brand for inclusion with concrete differentiators and proof points.

**Why:** Listicles are the canonical "recommendation" surface AI engines mine for comparison answers.

| Impact | Effort | Source |
|---|---|---|
| High | Medium (a few weeks of outreach + content production) | Industry research |

### Concrete steps

1. Run `"best <category>" "<brand>"` to confirm absence, then `"best <category>"` to find the top-3 third-party listicles.
2. For each listicle, find the editor or author. LinkedIn + the site's contact page.
3. Craft a personalized pitch: specific value of your product, 1-2 concrete differentiators, customer proof points (logos, quoted reviewers).
4. Offer them something useful in exchange: an interview with your founder, original data they can cite, a co-marketing campaign.
5. Follow up once. If no response after 2 weeks, move on.

### Templates

**Listicle inclusion pitch:**

```
Subject: Suggestion for your "Best <category>" article

Hi <Name>,

Loved your <date> piece on the best <category>. I noticed you didn't include
<Brand> — would you consider adding us?

What makes <Brand> different from the X options you covered:
1. <Specific differentiator with proof point>
2. <Second differentiator>

If it'd help, I can connect you with <our founder / a customer / our data
team> for original perspective. We also have <specific dataset / case study>
you might find useful for your readers.

Either way, thanks for the great piece.

<Sender>
```

### Real examples

- New SaaS startups regularly land in G2's category-specific listicles through this approach
- TrustRadius "Best of" articles update based on outreach + verified usage

### Common pitfalls

- ❌ Generic mass-emailing — gets ignored. Personalize every pitch.
- ❌ Demanding inclusion or paying for placement on legit publications (some editors will refuse to engage with brands that paid for placement elsewhere)
- ❌ Pitching for inclusion in a listicle that's a thinly disguised affiliate-money grab — won't move AI citation needle
- ❌ One-and-done — the highest-impact moves are getting on multiple listicles, not just one

### How to verify

Next audit will show Comparison_score moving 0→5 (1-3 listicles) or 5→10 (major listicles).

### Auto-close detector

`outreach-listicles`: re-run listicle probe; if brand appears in ≥1 new listicle since baseline → `partial`, ≥3 → `closed`.

---

## Move 11 — Mine GSC for impressions-but-no-clicks queries (GSC opt-in required)

**What:** Pull Google Search Console data, find queries where the brand has impressions but very few clicks, then rewrite the relevant pages to better answer those queries.

**Why:** These queries are AI engine candidates already — the brand is showing up in search for them. The rewrite is high-leverage because the discovery is already happening.

| Impact | Effort | Source |
|---|---|---|
| High | Medium (a few hours of analysis + rewrites) | @hridoyreh / @gouthamjay8 (May 2026) |

### Concrete steps

1. **Requires GSC opt-in (Q4 = A in audit setup).** If not opted in, this move is unavailable.
2. Pull GSC Performance data for last 90 days.
3. Filter to queries with: impressions > 100, clicks/impressions < 2%, position > 10.
4. For each high-impression-low-click query, identify the page being shown and audit:
   - Does the page actually answer this query?
   - Is there a question-shaped H2 matching the query language?
   - Is the answer a clean 40-60 word block?
5. Rewrite the relevant section to match query intent more directly.

### Templates

**GSC mining query (paste into GSC API tool or use Search Console Insights):**

```
Date range: Last 90 days
Filters:
  - Impressions > 100
  - CTR < 2%
  - Average position > 10
Sort by: Impressions descending
Output: Query, Page, Impressions, CTR
```

### Real examples

- Many high-velocity content teams use this loop weekly. The pattern is well-documented in the SEO community.

### Common pitfalls

- ❌ Rewriting pages without checking whether the impressions are commercial-intent or informational
- ❌ Trying to rank a single page for 20 different queries — diluting intent
- ❌ Stuffing query language verbatim into the page (Google penalizes; AI engines don't care)

### How to verify

Next audit will show the rewritten pages' passage_score increasing AND, on the next GSC pull, CTR increasing on the targeted queries.

### Auto-close detector

`mine-gsc-queries`: needs GSC re-pull comparison. **No automatic detector** — defaults to `open` per the default-fallback rule; surfaced for manual mark on next walkthrough.

---

## Move 12 — Pillar+spoke topical cluster build-out for top revenue topic

**What:** Build a hub-and-spoke topical cluster for your highest-revenue topic: one comprehensive pillar page + 5-7 deep spoke pages, all bidirectionally linked.

**Why:** Internal answer architecture is Action Factor 8 (weight 5) AND a major signal for Readiness Factor 8 (fan-out / topical depth, weight 8). Cluster structure tells AI engines you have topical authority.

| Impact | Effort | Source |
|---|---|---|
| Medium | High (1-2 weeks per cluster) | Industry research |

### Concrete steps

1. Identify your highest-revenue topic. The one your customers actually pay for.
2. Plan a pillar page: comprehensive, 2,000-4,000 words, covers the topic end to end with skimmable structure.
3. Plan 5-7 spoke pages: each diving deep into a sub-topic (e.g., if pillar is "<category> guide," spokes are "<category> for <use case>", "<category> vs <competitor>", "how to <do specific thing> in <category>").
4. Write the pillar first. Use the block rubric throughout (definition-first, fact-rich, question-shaped headings).
5. Write each spoke. Each spoke links back to the pillar in the first paragraph AND to 2-3 sibling spokes.
6. Pillar links forward to ALL spokes from a "Deep dives" section.

### Templates

**Pillar page structure:**

```
H1: <Topic> — A Comprehensive Guide for <Audience>

H2: What is <topic>?  [Definition + stat]
H2: How does <topic> work?  [Mechanism explanation]
H2: Why does <topic> matter for <audience>?  [Use case + ROI stat]
H2: Common approaches to <topic>  [Comparison]
H2: How to implement <topic>  [Step-by-step]
H2: Common pitfalls  [What to avoid]
H2: Deep dives  [Links to all spoke pages]
  - <Spoke 1 title>
  - <Spoke 2 title>
  ...
H2: FAQ  [FAQPage schema target]
```

### Real examples

- HubSpot's marketing methodology guide and its 50+ spokes
- Stripe's Payments Documentation — pillar (overview) + spokes (Cards, ACH, Wallets, Subscriptions, etc.)

### Common pitfalls

- ❌ Writing the spokes first and trying to retrofit a pillar afterward — leads to thin pillars
- ❌ Skipping bidirectional linking — defeats half the purpose
- ❌ Trying to build 3 clusters in parallel — too much; do one well first
- ❌ Generic AI-generated pillars that don't reflect actual brand expertise

### How to verify

Next audit will show Readiness Factor 8 (fan-out/topical depth) and Action Factor 8 (answer architecture) both moving up. Once the cluster is indexed, search rank for the topic also moves.

### Auto-close detector

`pillar-spoke-cluster`: no automatic single-signal detector. Surfaced for manual mark; verified through the increase in factors 8R and 8A on the next audit.
