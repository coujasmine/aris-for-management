---
name: theory-positioning
description: Anchor a management research manuscript or research question to its candidate parent theories, map the current academic conversation around each, and articulate the paper's position ("we extend X by...", "we challenge Y in that...", "we integrate X and Z to..."). Use when user says "what theory should I use", "position my paper", "I don't know how to frame this", "what tree should I hang this on".
argument-hint: [manuscript-path-or-research-question]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch, Skill, mcp__codex__codex
---

# Theory Positioning

The single most common reason a promising idea gets desk-rejected is **failure to identify the academic conversation it belongs to**. Reviewers ask: "what is this paper extending?" If the answer is "nothing specific" or "everything," the paper does not survive triage.

This skill takes a draft (or just a research question) and produces:
- 3-5 candidate **parent theories** the paper could anchor to
- For each: core propositions, active debates, recent variants, and what would be at stake by anchoring here
- A recommended primary anchor + secondary anchor
- Three positioning statements: **extend**, **challenge**, **integrate**

## Context: $ARGUMENTS

## Constants

- **OUTPUT_DIR = `theory-stage/`**
- **REPORT = `theory-stage/THEORY_POSITION.md`**
- **DOMAIN = `innovation-entrepreneurship`** — One of: `innovation-entrepreneurship`, `strategy`, `organization-theory`, `OB-HR`, `international-business`, `general-management`. Affects which theory families are surfaced first.
- **DIFFICULTY = `medium`** — `medium` (Claude only) / `hard` (Codex MCP cross-checks the anchor recommendation) / `nightmare` (Codex independently identifies parent theories from the manuscript).
- **REFRESH_LITERATURE = false** — When `true`, use WebSearch + Crossref to pull the 5-10 most-cited papers in each candidate theory from the past 5 years. Adds ~3 minutes per theory but produces sharper "current state of the conversation" notes.

> 💡 Override: `/theory-positioning "draft.md" — domain: strategy, difficulty: hard, refresh literature: true`

## Candidate Theory Library (innovation / entrepreneurship / strategy / organization)

The skill considers these as candidate parents. Not exhaustive — meant to seed the search. Add to this list as you encounter new ones.

### Strategy & resource-based theories
- **Resource-Based View (RBV)** — Wernerfelt 1984, Barney 1991; current debates around VRIN tautology, microfoundations
- **Dynamic Capabilities** — Teece, Pisano, Shuen 1997; Teece 2007; current debates around measurement, ordinary vs dynamic capabilities
- **Knowledge-Based View** — Grant 1996, Kogut & Zander 1992; absorptive capacity (Cohen & Levinthal 1990; Zahra & George 2002)
- **Real Options** — Bowman & Hurry 1993, McGrath 1997; current applications to entry, divestment, R&D investment

### Behavioral / cognitive theories
- **Upper Echelons** — Hambrick & Mason 1984, Hambrick 2007; current debates around TMT cognition, CEO characteristics, demographic vs cognitive proxies
- **Behavioral Theory of the Firm** — Cyert & March 1963; Greve 2003; aspiration-performance feedback, search behavior
- **Attention-Based View** — Ocasio 1997, 2011; how organizations allocate attention; current debates around micro-foundations
- **Managerial Cognition** — Walsh 1995, Helfat & Peteraf 2015; mental models, sensemaking
- **Sensemaking** — Weick 1995; identity, narrative, retrospective interpretation

### Institutional & social theories
- **Institutional Theory** — DiMaggio & Powell 1983, Meyer & Rowan 1977; isomorphism, legitimacy; current debates around institutional work, institutional complexity
- **Institutional Logics** — Thornton, Ocasio, Lounsbury 2012; multiple logics, hybridity
- **Social Network Theory** — Granovetter 1973, Burt 1992; structural holes, brokerage, embeddedness
- **Status & Stratification** — Podolny 1993; status as signal, status hierarchies
- **Categories & Identity** — Hsu & Hannan 2005; category fuzziness, identity audiences

### Entrepreneurship-specific
- **Opportunity Identification** — Shane & Venkataraman 2000; debates around discovery vs creation; Alvarez & Barney 2007
- **Effectuation** — Sarasvathy 2001; expert-entrepreneur logic; current debates against causation
- **Entrepreneurial Cognition** — Mitchell et al. 2007; scripts, biases, expert intuition
- **Bricolage** — Baker & Nelson 2005; resource-constrained creativity
- **Family Business / SEW** — Berrone, Cruz, Gomez-Mejia 2012; socioemotional wealth as objective function

### Innovation-specific
- **Absorptive Capacity** — Cohen & Levinthal 1990; Zahra & George 2002; debates around reconceptualization
- **Open Innovation** — Chesbrough 2003; current debates around mechanisms, outcomes, dark side
- **Ambidexterity** — Tushman & O'Reilly 1996; Raisch et al. 2009; exploration vs exploitation
- **Innovation Search** — March 1991 (exploration/exploitation); Katila & Ahuja 2002 (search depth/scope)
- **Technological Trajectories** — Dosi 1982; path dependence, lock-in

### Learning & adaptation
- **Organizational Learning** — Levitt & March 1988, Argote 2013; learning curves, transfer
- **Experiential Learning** — March 1991; near vs far transfer; learning from rare events
- **Routines & Capabilities** — Nelson & Winter 1982; Feldman & Pentland 2003 (routines as performative)

### Networks & external relations
- **Resource Dependence** — Pfeffer & Salancik 1978; debates around contemporary relevance
- **Transaction Cost Economics** — Williamson 1985; current debates around boundary conditions
- **Alliance & Network** — Gulati 1998, Powell, Koput, Smith-Doerr 1996

### Identity, emotion, micro-foundations
- **Organizational Identity** — Albert & Whetten 1985; Whetten 2006; identity audiences
- **Identity Work** — Ibarra 1999; entrepreneurial identity
- **Psychological Ownership / Passion** — Cardon et al. 2009; entrepreneurial passion
- **Emotion in Organizations** — Elfenbein 2007

If the manuscript clearly does not fit any of the above (e.g., it is in a sub-domain not covered), the skill should still attempt to identify the relevant theory family by searching the literature.

## Output Protocols

- Write `theory-stage/THEORY_POSITION.md` (overwrite each run)
- Append memory entry

## Workflow

### Phase 0: Extract the seed

From the manuscript (or research question):

1. **Phenomenon**: what observable thing is the paper about? (one sentence)
2. **Core claim**: what does the paper argue is true about that phenomenon? (one sentence)
3. **Already-invoked theories**: which theories does the manuscript currently cite or name? (extract from the theory section and introduction)
4. **Level of analysis**: individual, team, firm, industry, ecosystem, country
5. **Causal direction**: what is the antecedent and what is the outcome (or is it descriptive)?
6. **Style**: causal-explanatory / phenomenon-driven / theory-elaboration / theory-testing / theory-building

If only a research question is provided (no manuscript yet), the skill works from the question + DOMAIN constant. Output a "pre-anchor" version that helps the user decide what to write.

### Phase 1: Generate candidate parents

Produce 3-5 candidate parent theories, drawn from the library above plus any others surfaced by reading the manuscript.

For each candidate, write a 1-paragraph profile:

```markdown
### Candidate: [Theory Name]

**Origin paper(s)**: [Author Year]
**Core proposition**: [the theory's central claim, 1 sentence]
**Why this paper might anchor here**: [the connection point]
**What it would mean to anchor here**: [implications — what audience, what conversations, what stakes]
**Active current debates** (past 5 years):
  - Debate 1: [the controversy + recent papers]
  - Debate 2: [...]
**Where this paper could enter the debate**: [extension / challenge / integration]
**Risk of anchoring here**: [what could go wrong — e.g., crowded conversation, theoretical tautology accusations]
```

### Phase 2: Refresh recent literature (if REFRESH_LITERATURE = true)

For each candidate, search for the 5-10 most-cited recent (5-year) papers. Use:
- WebSearch: `<theory name> AMJ ASQ SMJ 2023..2026` or similar
- Crossref API for citation counts
- Google Scholar via WebFetch (carefully — rate limited)

Add to each candidate profile a `**Recent literature**` sub-section:

```markdown
**Recent literature** (verified $date):
- Author Year, Journal, "Title" — [why relevant to this debate]
- ...
```

Verify each citation actually exists before including (Crossref / DOI). Never fabricate.

### Phase 3: Compare candidates

Write a `## Comparison Matrix` table:

| Theory | Fit to Phenomenon | Fit to Core Claim | Audience Reach | Conversation Saturation | Debate Entry Cost |
|---|---|---|---|---|---|
| RBV | high | medium | very high | very saturated | high (must beat decades of work) |
| Dynamic Capabilities | medium | high | high | medium | medium |
| Absorptive Capacity | high | high | high | medium | low (open lanes) |
| ... | ... | ... | ... | ... | ... |

Each cell is scored low / medium / high with a 1-line rationale.

### Phase 4: Recommend an anchor

Pick a **primary** and **secondary** anchor.

**Primary**: the theory the paper should be framed against — its main "we extend X" partner.

**Secondary**: a theory that informs the mechanism or provides a counter-position to make the contribution sharper. Often the secondary is what the paper distinguishes itself from.

Write a `## Recommendation` section:

```markdown
### Primary Anchor: [Theory]
- **Why**: [2-3 sentences]
- **Recent papers to engage with**: [3-5 specific papers]
- **Predicted reviewer expectations**: [what reviewers steeped in this theory will demand]

### Secondary Anchor: [Theory]
- **Why**: [2-3 sentences]
- **Role**: [mechanism source / counter-position / boundary specifier]

### Why not [the runner-up]?
- [2 sentences explaining the trade-off]
```

### Phase 5: Write three positioning statements

These are the actual sentences the paper should use (in introduction + theory section). They are the operational output of the skill.

```markdown
### Extension Statement
"We extend [primary anchor] by showing that [specific extension], a gap that prior work — most notably [author year] and [author year] — has not addressed because [reason]."

### Challenge Statement (optional)
"We challenge [secondary anchor or part of primary] in that [specific challenge]. Whereas prior work argues [paraphrase prior position], we argue [our position]."

### Integration Statement (optional, if applicable)
"We integrate insights from [theory A] and [theory B] to develop a [unified framework / boundary specification / mechanism], addressing the limitation in [theory A] that [...] and the limitation in [theory B] that [...]."
```

Generate 2-3 variants of each statement for the author to pick from.

### Phase 6: Cross-model adversary (DIFFICULTY ≥ hard)

**Skip if DIFFICULTY = medium.**

#### Hard mode

Send the primary/secondary recommendation to Codex:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "high"}
  prompt: |
    A management research paper is being positioned. Claude recommends:

    Primary anchor: [theory + 1-sentence rationale]
    Secondary anchor: [theory + 1-sentence rationale]

    Manuscript snapshot:
    - Phenomenon: [...]
    - Core claim: [...]
    - Level of analysis: [...]

    As a senior $DOMAIN reviewer, evaluate this anchoring choice:

    1. Is the primary anchor the strongest available, or is there a better one?
    2. Is the secondary anchor doing real work, or is it decorative?
    3. What theory does the manuscript IGNORE that a reviewer would
       demand it engage with?
    4. Predict the most likely reviewer complaint about the proposed
       positioning.

    Return: agreement / partial / disagreement with Claude's recommendation,
    plus the alternative anchoring you would propose.
```

#### Nightmare mode

Let Codex read the manuscript directly via `codex exec` and produce its own independent anchoring recommendation, with no priming.

Compare Claude's and Codex's recommendations side by side in the final report.

### Phase 7: Write the report

```markdown
# THEORY_POSITION.md — $manuscript_id

Generated: $timestamp
Domain: $DOMAIN
Difficulty: $DIFFICULTY
Literature freshness: $date_or_static

## Seed
- Phenomenon: $phenomenon
- Core claim: $core_claim
- Already-invoked theories: [...]
- Level of analysis: $level
- Causal direction: $direction
- Style: $style

## Candidate Parent Theories
[Phase 1 + 2 output, all candidates]

## Comparison Matrix
[Phase 3 table]

## Recommendation
[Phase 4 output: primary, secondary, why not runner-up]

## Positioning Statements (drop into intro / theory section)
[Phase 5 output: extension / challenge / integration variants]

## Adversary Notes (DIFFICULTY ≥ hard)
[Codex's independent recommendation if different]

## Recommended Next Action
[one of:
  "Insert the extension statement into the introduction. Then run /hypothesis-builder."
  "Engage the secondary anchor's recent literature before drafting the theory section."
  "Conflict between Claude and Codex anchoring — user decision required."
  "Run /contribution-stress-test once anchoring is locked in."
]
```

### Phase 8: Memory hook

Append to `memory/MEMORY.md`:

```markdown
- [$manuscript_id theory position, $timestamp]: primary = $primary, secondary = $secondary. See theory-stage/THEORY_POSITION.md
```

Also create / append `memory/theories/<primary>.md` with this paper's positioning relative to the theory.

## Key Rules

- **No theory salad**: invoke 3-5 candidates internally, but the manuscript should anchor on **one** primary. Multiple primary anchors confuses reviewers.
- **Recent > foundational**: the foundational paper establishes the conversation, but the manuscript must engage what is being said *now*. Cite the foundational paper, then cite recent (5-year) papers in the same conversation.
- **Beware crowded conversations**: anchoring to RBV or Upper Echelons is high-audience but high-saturation — the bar for marginal contribution is very high. Anchoring to a less-trafficked theory (e.g., Attention-Based View, Effectuation) opens lanes but reduces audience.
- **The secondary anchor pulls weight**: a good secondary anchor sharpens the contribution by giving the paper something to push against. If the secondary is purely decorative, drop it.
- **Audience proximity**: the chosen anchor determines who reviews the paper. Reviewers steeped in RBV will accept different evidence than reviewers steeped in Behavioral Theory. Pick the anchor whose reviewers your evidence will persuade.
- **Honesty about ignored theories**: if there is a theory a reviewer is likely to demand engagement with, the manuscript must either engage it or explain why it does not apply. Pretending an obvious adjacent theory does not exist is fatal.
- **Citation verification**: every paper named in this skill's output must be verified via Crossref / DOI lookup. Never invent author names, years, or paper titles.

## Bridging to other skills

- Anchor locked in → run `/contribution-stress-test` to audit the contribution against the chosen anchor's standards
- Anchor still uncertain after this skill → the underlying research question may be too broad; consider running a literature mapping pass first
- Anchor revealed a previously-ignored adjacent theory → update the manuscript's theory section before running `/hypothesis-builder`
- Multiple revisions of anchor over time → audit trail lives in `memory/theories/<theory>.md`
