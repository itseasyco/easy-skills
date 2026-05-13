# Per-Platform Snapshot (AI Engine Projections)

These projections show **which AI engines are most/least likely to cite the brand given current factor scores**. They are NOT inputs to the composite Readiness/Action scores — they are derived projections shown alongside, to help teams prioritize moves by engine target.

Each projection is 0-100. If the user opted into **live AI citation probing** (Step 5 of audit) and provided API keys, the actual measured citation rate replaces the projection for that engine and is flagged "✓ measured" in the report.

---

## Projection formulas

Each formula uses normalized inputs (each input is 0-100):

### ChatGPT

```
ChatGPT_projection = 0.50 × Wikipedia_score
                   + 0.20 × Quora_score
                   + 0.20 × Web_authority_score   # Factor 3 from readiness-factors.md
                   + 0.10 × Recency_score          # Factor 6 from readiness-factors.md
```

**Why these weights:** ChatGPT cites Wikipedia 47.9% of the time. Authority + recency are tie-breakers.

### Perplexity

```
Perplexity_projection = 0.50 × Reddit_score
                      + 0.20 × Recency_score
                      + 0.20 × Web_authority_score
                      + 0.10 × Schema_completeness_score   # Factor 4 from readiness-factors.md
```

**Why these weights:** Perplexity cites Reddit 46.7% of the time and weights recency heavily.

### Google AI Overviews

```
AIO_projection = 0.50 × Search_rank_proxy_score    # Factor 2 from readiness-factors.md
               + 0.30 × Schema_completeness_score
               + 0.20 × Answer_architecture_score   # Action Factor 8
```

**Why these weights:** AI Overviews heavily favors content already ranking in top-10 organic. Schema + answer architecture are the differentiators among top-rankers.

### Gemini

```
Gemini_projection = 0.40 × Search_rank_proxy_score
                  + 0.20 × Schema_completeness_score
                  + 0.20 × Web_authority_score
                  + 0.20 × Recency_score
```

**Why these weights:** Gemini behaves similarly to AI Overviews but with more weight on brand authority and recency.

### Claude

```
Claude_projection = 0.40 × Passage_citability_score   # Action Factor 1
                  + 0.30 × Authority_quotations_score # Action Factor 4
                  + 0.30 × Structural_readability     # composite of Action Factors 7 + 8
```

**Why these weights:** Claude favors well-structured, nuanced passages with authority backing. Less rank-driven than Google-family engines.

---

## Where each input comes from

| Input | Source rubric | Factor # |
|---|---|---|
| Wikipedia_score | brand-surface-area.md | (Wikipedia row) |
| Quora_score | brand-surface-area.md | (Quora row) |
| Reddit_score | brand-surface-area.md | (Reddit row) |
| Web_authority_score | readiness-factors.md | Factor 3 |
| Recency_score | readiness-factors.md | Factor 6 |
| Schema_completeness_score | readiness-factors.md | Factor 4 |
| Search_rank_proxy_score | readiness-factors.md | Factor 2 |
| Answer_architecture_score | action-factors.md | Factor 8 |
| Passage_citability_score | action-factors.md | Factor 1 |
| Authority_quotations_score | action-factors.md | Factor 4 |
| Structural_readability | composite | (Action Factors 7 + 8) / 2 |

All inputs are normalized to 0-100 before applying the projection formula.

---

## How projections are displayed in the report

A row of 5 gauges, color-coded:
- **Green:** ≥70 — engine is likely to cite
- **Yellow:** 40-69 — borderline; specific moves would push it green
- **Red:** <40 — engine is unlikely to cite without significant work

Each gauge labeled with the engine name and the score. If `measured` rate is available, gauge shows both projected and measured side-by-side with a delta arrow.

---

## How projections drive walkthrough prioritization

When the walkthrough picks the top 5-7 moves, it considers per-engine targeting:

- If user has indicated they care most about **Perplexity** (high-intent buyers research there), moves that lift Reddit_score get bumped in priority
- If user cares most about **ChatGPT**, Wikipedia and authority-mention moves get bumped
- If user cares most about **AIO** / **Gemini**, organic-rank and schema moves get bumped

The walkthrough asks once at the start:

> "Which AI engine matters most to you right now?
> A. ChatGPT
> B. Perplexity
> C. Google AI Overviews / Gemini
> D. Claude
> E. All equally"

The chosen target is stored in `preferences.json` under `target_engine` and re-used on subsequent runs.

---

## Honesty about projections

The report must explicitly state:

> "These are projections derived from current factor scores using public industry research weights. They are not direct measurements. To replace projections with actual citation rates, opt into live AI probing on your next audit run."

If `measured` rate is available, the report adds:

> "**Measured** rate for <engine> is based on N seed prompts run against the engine's API on <date>. This replaces the projection."
