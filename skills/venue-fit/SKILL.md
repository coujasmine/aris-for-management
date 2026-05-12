---
name: venue-fit
description: Score a management research manuscript's fit across 25+ FT50/UTD24 and adjacent journals in innovation, entrepreneurship, strategy, organization, and policy. Outputs a ranked submission priority list, alternates (where to resubmit after rejection), and explicit "do not submit here" venues with reasons. Use when user says "where should I submit", "venue fit", "AMJ or SMJ", "what journal".
argument-hint: [manuscript-path]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch, Skill, mcp__codex__codex
---

# Venue-Fit Scoring (Innovation / Entrepreneurship / Strategy / Organization)

Choosing the right submission target dominates 12 months of revision time. This skill scores a manuscript on six dimensions, matches against the preference profile of 25+ journals, and produces a ranked submission plan with alternates.

## Context: $ARGUMENTS

## Constants

- **OUTPUT_DIR = `venue-stage/`**
- **REPORT = `venue-stage/VENUE_FIT.md`**
- **DIFFICULTY = `medium`** — `medium` (Claude only) / `hard` (Codex MCP cross-checks top-3 recommendations) / `nightmare` (Codex reads the manuscript independently and produces its own ranking).
- **REFRESH_EDITORIAL = false** — When `true`, use WebSearch to pull current editor names, special issues, and recent (past 12 months) published articles to refresh each venue's preference profile. Adds ~2 minutes per top venue but produces sharper, current advice. Set true if the static profiles below are > 12 months old.
- **TOP_N = 5** — Number of primary recommendations
- **ALT_N = 3** — Number of alternates (post-rejection retargets)

> 💡 Override: `/venue-fit "draft.md" — difficulty: hard, refresh editorial: true`

## Six Scoring Dimensions

Each dimension is scored 1-10 from the manuscript.

1. **Theory strength** (D-THE) — How distinctive and developed is the theoretical contribution? (9-10 = generative new framework; 1-3 = atheoretical or trivially incremental)
2. **Methodological rigor** (D-MET) — Identification strategy, sample size, validation, robustness. (9-10 = quasi-experiment with multiple ID strategies; 1-3 = OLS on convenience sample)
3. **Phenomenon novelty** (D-PHE) — Is the empirical phenomenon itself new, rare, or theoretically interesting? (9-10 = unique data on a hard-to-observe phenomenon; 1-3 = well-studied phenomenon, marginal context shift)
4. **Internationalization** (D-INT) — Is the study cross-national, comparative, or institutionally distinctive? (9-10 = multi-country panel with institutional contrast; 1-3 = single-country, single-industry)
5. **Practitioner relevance** (D-PRA) — Can managers / entrepreneurs / policymakers act on the findings? (9-10 = direct decision implications; 1-3 = pure scholarship signal)
6. **Style fit** (D-STY) — Empirical-quant / empirical-qual / mixed / formal / conceptual / review

These six dimensions are then matched against each journal's preference profile (below).

## Journal Preference Profiles

Each profile has: tier, primary focus, preferred style, dimension weights, "what gets desk rejected here", current editor, typical lead time, "house keywords" (themes the journal has favored in the past 24 months).

> ⚠️ Editor names and recent-favored themes drift. Set `REFRESH_EDITORIAL = true` to verify against current journal website before final recommendation.

### Top-tier general management

| Code | Journal | Tier | Style | D-THE | D-MET | D-PHE | D-INT | D-PRA | Notes |
|---|---|---|---|---|---|---|---|---|---|
| AMJ | Academy of Management Journal | FT50 | emp-quant + qual | 9 | 8 | 7 | 6 | 5 | "Theoretical contribution unclear" = #1 desk reject |
| ASQ | Administrative Science Quarterly | FT50 | qual + emp | 10 | 7 | 9 | 6 | 4 | Sociology / institutional / process strong; theoretical novelty bar highest |
| AMR | Academy of Management Review | FT50 | theory only | 10 | n/a | n/a | 5 | 5 | No empirics. "Not generative enough" / "incremental" = killer |
| OS | Organization Science | FT50 | emp-quant + formal | 9 | 9 | 7 | 6 | 5 | Multi-method welcome; behavioral / org-design root |
| JOM | Journal of Management | FT50 | emp-quant + meta | 8 | 9 | 6 | 6 | 6 | Meta-analyses gold standard; reviews welcome |
| JMS | Journal of Management Studies | FT50 | qual + emp + theory | 9 | 7 | 8 | 8 | 5 | European/critical management; qualitative theory-building friendly |
| MS | Management Science | FT50/UTD24 | formal + emp | 8 | 10 | 7 | 6 | 5 | Identification strategy must be near-experimental; OB section relevant |

### Strategy

| Code | Journal | Tier | Style | D-THE | D-MET | D-PHE | D-INT | D-PRA | Notes |
|---|---|---|---|---|---|---|---|---|---|
| SMJ | Strategic Management Journal | FT50 | emp-quant | 8 | 10 | 7 | 6 | 6 | Identification strategy is non-negotiable; firm-level focus |
| StrSci | Strategy Science | FT50 (recent) | formal + exp | 9 | 9 | 7 | 5 | 5 | INFORMS; behavioral strategy & decisions strong |
| GSJ | Global Strategy Journal | sub-FT50 | emp + theory | 7 | 7 | 7 | 10 | 6 | International strategy specifically; friendlier than SMJ |
| LRP | Long Range Planning | sub-FT50 | emp + practitioner | 6 | 7 | 7 | 7 | 9 | Practice-relevant strategy; more accessible |

### Entrepreneurship (your home turf for FT50)

| Code | Journal | Tier | Style | D-THE | D-MET | D-PHE | D-INT | D-PRA | Notes |
|---|---|---|---|---|---|---|---|---|---|
| JBV | Journal of Business Venturing | FT50 | emp-quant + qual | 8 | 8 | 9 | 6 | 7 | The entrepreneurship #1; new-venture / opportunity / cognition |
| SEJ | Strategic Entrepreneurship Journal | FT50 | emp-quant | 9 | 9 | 7 | 6 | 6 | Strategy × entrepreneurship intersect; theoretical bar high |
| ETP | Entrepreneurship Theory & Practice | FT50 | theory + qual + emp | 9 | 7 | 8 | 7 | 6 | Theory-heavy; family business strong; conceptual papers welcome |
| JBVI | J. Business Venturing Insights | sub-FT50 | short | 6 | 6 | 9 | 5 | 7 | Short-format (3-5K words); fast turnaround; insights/observations |
| SBE | Small Business Economics | sub-FT50 | emp-quant | 6 | 8 | 7 | 7 | 7 | Economics flavor; panel data + policy strong |
| ISBJ | Int'l Small Business Journal | sub-FT50 | qual + emp | 6 | 6 | 8 | 8 | 6 | European school; qualitative welcome |
| JSBM | Journal of Small Business Management | sub-FT50 | emp + practitioner | 5 | 6 | 7 | 6 | 8 | Practitioner-relevant SME research |
| FBR | Family Business Review | sub-FT50 | theory + emp | 8 | 7 | 8 | 7 | 6 | Family business specialist; conceptual strong |

### Innovation / Technology

| Code | Journal | Tier | Style | D-THE | D-MET | D-PHE | D-INT | D-PRA | Notes |
|---|---|---|---|---|---|---|---|---|---|
| RP | Research Policy | FT50 | emp + policy | 7 | 8 | 8 | 7 | 8 | Innovation policy / IP / S&T systems; econometric friendly |
| ICC | Industrial and Corporate Change | sub-FT50 | emp + theory | 7 | 7 | 8 | 7 | 6 | Evolutionary economics; long-run industry studies |
| I&I | Industry and Innovation | sub-FT50 | emp + qual | 6 | 7 | 8 | 7 | 6 | Sector / cluster / regional innovation |
| Tech | Technovation | sub-FT50 | emp + practitioner | 6 | 6 | 7 | 6 | 8 | Tech management; open access option |
| R&D | R&D Management | sub-FT50 | emp + practitioner | 5 | 6 | 7 | 6 | 8 | R&D process / capability / NPD |
| JPIM | J. Product Innovation Management | FT50 | emp + practitioner | 6 | 8 | 7 | 6 | 9 | Product innovation; strong methods + practice |

### Adjacent FT50 (might fit if the paper has the right angle)

| Code | Journal | Tier | Style | D-THE | D-MET | D-PHE | D-INT | D-PRA | Notes |
|---|---|---|---|---|---|---|---|---|---|
| JIBS | Journal of International Business Studies | FT50 | emp + theory | 8 | 8 | 7 | 10 | 6 | Required for any IB / international context paper |
| MISQ | MIS Quarterly | FT50/UTD24 | emp + design | 8 | 9 | 8 | 6 | 7 | Digital innovation / platforms / IS-strategy interface |
| JAP | Journal of Applied Psychology | FT50 | emp-quant | 7 | 9 | 6 | 5 | 6 | Individual-level (entrepreneur cognition, OB) |
| OBHDP | Organizational Behavior and Human Decision Processes | FT50 | exp + emp | 8 | 9 | 6 | 4 | 5 | Experimental, decision-making focus |

## Output Protocols

- Write `venue-stage/VENUE_FIT.md` (overwrite each run)
- Append memory entry

## Workflow

### Phase 0: Read the manuscript

Read:
- Abstract
- Introduction (especially the hook and contribution paragraphs)
- A representative paragraph from the theory section
- Methods (sample, data, identification strategy)
- Headline findings
- One paragraph of discussion

Extract:
- Paper type (empirical-quant / qual / mixed / theory / meta / review)
- Sample (N, country, period, industry)
- Phenomenon (one sentence: what is the empirical setting?)
- Theoretical anchor (which parent theory does it engage?)
- Contribution claim (one sentence)
- Main finding (one sentence)

### Phase 1: Score the six dimensions

Assign D-THE / D-MET / D-PHE / D-INT / D-PRA each a 1-10 score, plus identify D-STY.

Justify each score with a 1-2 sentence reference to the manuscript. Do not inflate — these scores are the basis for every downstream recommendation.

### Phase 2: Match against venue profiles

For each venue, compute a fit score:

```
fit(venue) = Σ over dimensions [venue_weight(d) * min(manuscript_score(d), 10) ] / Σ venue_weight(d)
```

Penalize style mismatches sharply:
- Empirical paper to AMR: fit = 0 (AMR is theory-only)
- Pure theory paper to SMJ: fit cap = 4
- Qualitative paper to OBHDP: fit cap = 4

Penalize topic mismatches:
- Non-international paper to JIBS: fit cap = 4
- Non-IS paper to MISQ: fit cap = 3
- Non-family-business paper to FBR: fit cap = 3
- Non-product-innovation paper to JPIM: fit cap = 4

### Phase 3: Editorial refresh (if REFRESH_EDITORIAL = true)

For the top N venues from Phase 2, use WebSearch to verify:

1. **Current editor-in-chief and section editors** (changes ~ every 3 years)
2. **Recent special issues** (especially those with open calls or recent closes — they shape what passes desk review)
3. **Past-12-months high-cite articles** (signals what the journal is actively rewarding)
4. **Recent editorials** (editors often telegraph preferences in editorials and "from the editor" notes)

Tools to use (in order of preference):
- WebSearch for "<journal name> editor in chief 2026"
- WebFetch the journal's editorial page (Wiley / Elsevier / INFORMS / Sage / SMS)
- For special issues: WebSearch "<journal name> call for papers special issue"

Record the date of editorial info checked so the report self-discloses freshness.

### Phase 4: Cross-model adversary (DIFFICULTY ≥ hard)

**Skip if DIFFICULTY = medium.**

#### Hard mode

Send the top-3 ranked venues to Codex with a challenge prompt:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "high"}
  prompt: |
    Claude has scored this manuscript and recommended these top 3 venues:

    1. [VENUE_A] — fit X.X — rationale: [1 sentence]
    2. [VENUE_B] — fit X.X — rationale: [1 sentence]
    3. [VENUE_C] — fit X.X — rationale: [1 sentence]

    Manuscript summary:
    - Paper type: [...]
    - Sample: [...]
    - Phenomenon: [...]
    - Theory: [...]
    - Contribution: [...]
    - Main finding: [...]

    Your job:
    1. For each of the 3 venues, identify the single weakest dimension of fit
    2. Name one venue NOT in the top 3 that you would seriously consider
    3. Identify any "do not submit here" venue that Claude might have
       overlooked but is genuinely a poor fit
    4. Final recommendation: which venue would YOU target first, and why?

    Be skeptical of Claude's scoring. You are the second opinion.
```

#### Nightmare mode

Same pattern as `auto-review-loop` nightmare: let Codex read the manuscript via `codex exec` and produce an independent ranking with no priming from Claude's scores.

### Phase 5: Write the recommendation

Produce `VENUE_FIT.md`:

```markdown
# VENUE_FIT.md — $manuscript_id

Generated: $timestamp
Editorial info freshness: $date_or_static
Difficulty: $DIFFICULTY

## Manuscript Snapshot
- Paper type: $type
- Sample: $sample
- Phenomenon: $phenomenon
- Theoretical anchor: $theory
- Contribution claim: $contribution
- Main finding: $finding

## Dimension Scores
| Dimension | Score | Justification |
|---|---|---|
| Theory (D-THE)            | X/10 | ... |
| Method (D-MET)            | X/10 | ... |
| Phenomenon (D-PHE)        | X/10 | ... |
| Internationalization (D-INT) | X/10 | ... |
| Practitioner (D-PRA)      | X/10 | ... |
| Style                     | $type | ... |

## Top 5 Recommendations

### 1. [VENUE] — fit X.X / 10 — submit first
- **Why a fit**: [2-3 sentences citing dimensions]
- **Why this venue first**: [1 sentence on strategic logic]
- **Risks**: [the 1-2 dimensions where this paper is weaker than typical accepted paper]
- **Current editorial signals**: [if REFRESH_EDITORIAL was true]
- **Estimated review time**: [from public journal data]
- **What to sharpen in cover letter**: [1-2 specific things]

### 2. [VENUE] — fit X.X / 10 — submit if #1 rejects
[same structure]

... [5 total]

## Alternates (after first-round rejection at top venue)

### A1. [VENUE] — fit X.X / 10
- **Why this is the right rebound**: [...]
- **What to revise before rebounding here**: [...]

[3 alternates]

## ⚠️ Do NOT Submit Here

| Venue | Reason |
|---|---|
| AMR | Has empirics — AMR is theory-only |
| ... | ... |

## Codex Adversary Notes (if DIFFICULTY ≥ hard)
[verbatim from Phase 4]

## Recommended Path
[one of:
  "Submit to [V1]. If rejected, revise per [V1] reviewer feedback before rebounding to [A1]."
  "Run /contribution-stress-test first — current contribution is too weak for any top-5 recommendation."
  "Run /hypothesis-builder first — methodological weaknesses are downstream of unclear hypotheses."
]
```

### Phase 6: Memory hook

Append to `memory/MEMORY.md` under `## 我的论文项目`:

```markdown
- [$manuscript_id venue fit, $timestamp]: top recommendation $VENUE (fit X.X). Alternates: [A1, A2, A3]. See venue-stage/VENUE_FIT.md
```

Also append a one-line note to `memory/venues/` (create `<venue>.md` if it does not exist) recording when this paper was evaluated for that venue.

## Key Rules

- **No score inflation**: dimension scores should reflect what a senior reviewer would say, not what the author hopes. If unsure, score lower.
- **Style match is a hard gate, not a tiebreaker**: a perfect dimension match cannot rescue a style mismatch. AMR will not accept your empirical paper no matter how strong the theory.
- **Topic-specific journals deserve separate evaluation**: JIBS, MISQ, FBR, JPIM have hard topical gates. Use the caps in Phase 2.
- **The "right" venue is not always the highest-tier**: an AMJ aim with low odds + 18 months of revision can be worse than a JBV aim with strong odds + 9 months. Factor in revision realism.
- **Editorial signals drift**: any recommendation is only as fresh as the editorial info it is based on. If the static profile is > 12 months old, set REFRESH_EDITORIAL = true.
- **One venue at a time**: management journals enforce single-submission ethics. The "Top 5" is a *priority list*, not a parallel submission plan.
- **Cover letter is downstream**: knowing the venue determines what to highlight in the cover letter. A weak fit dimension can sometimes be reframed in the cover letter.

## Bridging to other skills

- All top-5 venues have low fit (< 5) → the manuscript may not be ready for FT50; run `/contribution-stress-test` and `/hypothesis-builder` first
- Editorial refresh reveals a special issue match → consider the special-issue route instead of the regular submission queue
- Top venue selection is locked → run `/auto-review-loop` with that venue's reviewer persona to validate readiness before submitting

## Maintenance

The venue profiles above were synthesized from journal websites, recent editorials, and past 24-month publication patterns. Re-validate annually:
- New journals added to FT50 (Strategy Science was the most recent addition)
- Editor-in-chief changes
- Major editorial direction shifts (e.g., JOM increasing its meta-analysis intake)

To update: edit this SKILL.md directly, commit the change to your repo. Future runs use the new profiles.
