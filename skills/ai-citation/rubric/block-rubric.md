# Block-Level Passage Rubric

This rubric scores individual content blocks (text segments bounded by H2 or H3 headings). It is the foundation of **Action Score Factor 1: Passage citability**.

A block is the unit AI engines extract and cite. The optimal length for AI citation is **134-167 words** (industry research from passage-mining analyses).

---

## The 6 criteria

Each block is checked against six pass/fail criteria. Block score = (criteria passed / 6) × 100. A block **passes the 70+ threshold** if it scores ≥70 (i.e., ≥5 of 6 criteria passed).

| # | Criterion | Pass condition | Anti-example |
|---|---|---|---|
| 1 | **Definition-first opening** | First sentence uses the pattern `<subject> is/refers to/means…` OR opens with a direct answer to an implied question | "If you've ever wondered why X matters, the answer might surprise you…" |
| 2 | **Self-contained** | First 60 words name the subject explicitly. No leading pronouns (`It`, `This`, `They`, `These`). Block is understandable without prior context. | "It works by routing requests through edge servers…" (no subject named) |
| 3 | **Word count fit** | Block is 134-167 words, OR contains sub-block extractable passages of 50-200 words bounded by clear paragraph breaks | Block is 12 words (too short) or 800 words wall-of-text (no extractable sub-block) |
| 4 | **Statistical density** | At least 1 specific stat per 150 words. A stat is a % / $ / named study / specific count / timeframe with measurement | "Many companies report significant improvements" (vague, no stat) |
| 5 | **Answer-first structure** | Direct answer in first 1-2 sentences; supporting detail/elaboration follows | "Background context… then more context… and finally the answer in sentence 7" |
| 6 | **No vague quantifiers** | Avoids unsourced "many," "most," "a lot," "studies show," "experts agree" — every quantifier has a name attached | "Studies show that 70% of users…" (no named study) |

---

## Examples

### High-scoring block (passes all 6 — score 100)

```
Content delivery networks (CDNs) are distributed server systems that cache and
serve web content from locations geographically close to end users. A CDN reduces
latency by 50-70% on average by serving assets from edge servers rather than a
single origin server, according to Cloudflare's 2025 State of the Edge report.
The three largest CDN providers as of 2026 are Cloudflare (serving approximately
20% of all websites per BuiltWith data), Amazon CloudFront, and Akamai
Technologies. CDNs are particularly impactful for video and image delivery,
where edge caching can cut bandwidth costs by 40-60% compared to origin-only
serving.
```

**Why it works:**
- ✅ Definition-first ("CDNs are distributed server systems…")
- ✅ Self-contained (subject named in first 4 words)
- ✅ Word count fit (87 words — extractable as a clean passage)
- ✅ Stats: 50-70%, 20%, 40-60% (3 stats in 87 words = 1 per 29 words, well above threshold)
- ✅ Answer-first (definition → impact → who → use case)
- ✅ Named sources: Cloudflare 2025 report, BuiltWith. No vague quantifiers.

### Low-scoring block (fails most — score 17)

```
If you've ever wondered why some websites load faster than others, the answer
might surprise you. There's this amazing technology that has been around for a
while now. It's changed the way we think about web performance, and many experts
agree it's now essential. Let me explain how it works and why you should care
about it for your business.
```

**Why it fails:**
- ❌ Definition-first — no, opens with rhetorical hook
- ❌ Self-contained — uses "this," "it," "this technology" with no subject named
- ✅ Word count fit (53 words — borderline but extractable)
- ❌ Stats — zero
- ❌ Answer-first — promises explanation in the next paragraph; no answer here
- ❌ Vague quantifiers — "many experts agree" with no named experts

---

## How to segment blocks

Walk the rendered DOM (or markdown):

1. Find each H2 and H3 boundary
2. All content between H_n and the next H_n (of equal or lower depth) is the block
3. Skip blocks with no text (e.g., a heading immediately followed by another heading)
4. Skip pure navigation/ToC blocks
5. Skip pure list-only blocks under 30 words (lists are scored separately at the structural level)

---

## Rewrite suggestions (used by report)

For every block scoring below 70, the report includes a suggested rewrite. Use this template:

```markdown
**Current opening (first 2 sentences):**
> [original text]

**Problem:** [identify which criteria failed — e.g., "buried answer, no facts, vague quantifiers"]

**Suggested rewrite:**
> [<Subject> is <definition>. <Specific stat with named source>. <Second supporting sentence with another stat or quote>.]

**Additional improvements:**
- Add a question-shaped H3 above this block
- Split long paragraph into 2-3 short ones (2-4 sentences each)
- Add an authority quote with name + credential
```

---

## What this rubric does NOT score

- **Tone** — formal vs. conversational doesn't matter for AI extraction
- **Length of the page overall** — only block length
- **SEO keyword density** — orthogonal to citability
- **Engagement metrics** (CTR, dwell time) — outside scope, AI engines don't use these for citation
