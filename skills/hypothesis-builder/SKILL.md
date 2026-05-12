---
name: hypothesis-builder
description: Audit and rebuild hypothesis arguments in a management research manuscript. Forces every hypothesis through a 3-layer chain (mechanism / boundary / counter-argument), detects common flaws (analogy substituting for causal mechanism, HARKing risk, scope creep, mechanism-outcome misalignment), and rewrites weak arguments. Use when user says "audit hypotheses", "fix my H1", "my hypotheses feel thin".
argument-hint: [manuscript-path-or-hypothesis-list]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill, mcp__codex__codex, mcp__codex__codex-reply
---

# Hypothesis Builder & Auditor

Most rejections at AMJ / ASQ / SMJ / JBV cite "logical leaps" or "thin theoretical mechanism" in hypothesis development. This skill audits each hypothesis against a strict 3-layer chain and rewrites weak arguments.

## Context: $ARGUMENTS

## Constants

- **TARGET_VENUE = `AMJ`** — Same set as `/contribution-stress-test`. AMR rarely uses formal hypotheses; for AMR drafts the skill audits propositions instead (set PAPER_TYPE = theory-paper).
- **PAPER_TYPE = `empirical-quant`** — `empirical-quant` (Hypotheses), `empirical-qual` (Propositions / Research Questions), `theory-paper` (Propositions), `meta-analysis` (Hypotheses), `mixed-methods` (both).
- **OUTPUT_DIR = `hypothesis-stage/`**
- **REPORT = `hypothesis-stage/HYPOTHESIS_AUDIT.md`**
- **REWRITE = `hypothesis-stage/HYPOTHESES_REVISED.md`** — Clean rewritten version of each hypothesis.
- **DIFFICULTY = `medium`** — `medium` (Claude only) / `hard` (Codex MCP adversary on weakest H) / `nightmare` (Codex reads manuscript directly).
- **MIN_PASSING_SCORE = 6**

> 💡 Override: `/hypothesis-builder "draft.md" — venue: SMJ, difficulty: hard`

## The 3-Layer Chain

Every hypothesis must defend three layers. Missing any layer is a structural flaw, not a stylistic one.

### Layer 1 — **Mechanism (Why)**

The causal process linking antecedent to outcome. Must be:
- **Named**: identify the mechanism by name (e.g., "information asymmetry", "status threat", "absorptive capacity", "legitimacy spillover")
- **Specified**: trace the mechanism step-by-step from A to B (A → step 1 → step 2 → B)
- **Grounded**: cite the foundational theoretical source for the mechanism
- **Distinguished**: explain why the proposed mechanism dominates plausible rival mechanisms

### Layer 2 — **Boundary (When / Where / Who)**

Scope conditions under which the relationship holds:
- **Contextual**: institutional environment, industry, region, organizational stage
- **Temporal**: short-term vs long-term, equilibrium vs disequilibrium
- **Construct-level**: at what range of the antecedent does the relationship hold? (Strong on the tails? Reverses at extremes?)
- **Boundary articulation must be testable** — vague boundaries ("in some contexts") are not boundaries.

### Layer 3 — **Counter-argument (Falsification & Defense)**

The opposite position taken seriously:
- **State the strongest reverse-direction argument** explicitly
- **State the strongest null argument** (why we might find no relationship)
- **Resolve** by either (a) explaining why our argument dominates, or (b) deriving a more nuanced prediction (interaction, threshold, curvilinearity)

This is the layer most authors skip. **Skipping it is the single highest-impact source of reviewer hostility** — reviewers feel the author has not engaged seriously with their own claims.

## Common Flaws Catalog

The skill scans for these patterns. Each detected flaw is logged with the offending sentence.

| Flaw | Pattern | Fix |
|---|---|---|
| **Analogy as mechanism** | "Like A, B will also..." — uses surface similarity instead of causal process | Replace with a named, specified mechanism |
| **Mechanism-outcome misfit** | The proposed mechanism would actually predict a different outcome than what is hypothesized | Either change the mechanism or the predicted outcome |
| **HARKing risk** | Hypothesis is unusually narrow / specific in ways suspiciously matched to a known result | Pre-register, or state the broader theoretical claim first |
| **Scope creep** | The hypothesis applies to "organizations" but the data covers only "Chinese listed SMEs in 2019-2021" | Narrow the hypothesis to match the empirical scope, OR justify generalization explicitly |
| **Vague modifier** | "X may increase Y", "X tends to affect Y" — uncommitted language | Commit to a directional claim or specify the conditions |
| **Tautology** | The hypothesis is true by definition of the constructs | Sharpen the construct definitions |
| **Black-box mediation** | "X → Y via mediator M" without specifying why M | Defend the mediator independently |
| **Opportunistic moderation** | Moderator added post-hoc to rescue a weak main effect | Move to robustness analyses, not hypotheses |
| **Boundary as limitation** | Scope condition appears only in the limitations section, not in the hypothesis | Hoist into the hypothesis itself |
| **Citation laundering** | Citation does not actually support the claim ("Smith, 2018" cited for a claim Smith never made) | Verify each citation; remove or replace |

## Output Protocols

- Write `hypothesis-stage/HYPOTHESIS_AUDIT.md` (overwrite each run)
- Write `hypothesis-stage/HYPOTHESES_REVISED.md` (clean rewrite of every H)
- Append a memory entry under `## 我的论文项目`

## Workflow

### Phase 0: Extract hypotheses

Read the manuscript and locate every H / P / RQ:
1. Parse sentences matching `^Hypothesis \d+`, `^H\d+:`, `^Proposition \d+:`, `^P\d+:`, `^Research Question \d+:`, etc.
2. Also scan for sentences appearing right before `Method` / `Methodology` section headers
3. Extract conditionals ("If A, then B") and directional claims ("the higher X, the higher Y") even if not formally numbered

For each extracted hypothesis, capture:
- ID (H1, H2a, P3, etc.)
- Verbatim text
- Surrounding theoretical paragraph (the argument supporting it)
- Page / line / section reference

If no formal hypotheses exist (theory paper, qual study), instead extract:
- Propositions (theory paper)
- Research questions + theoretical claims (qual)
- Aggregate theoretical claims (review essay)

### Phase 1: Per-hypothesis audit

For each hypothesis, run the 3-layer chain and the flaws catalog.

Write per-hypothesis sections in `HYPOTHESIS_AUDIT.md`:

```markdown
## H1: [verbatim hypothesis text]

### Surrounding Argument (verbatim)
[copy the supporting paragraph(s)]

### Layer 1 — Mechanism
- **Named?** [Yes/No: the named mechanism is "..."]
- **Specified?** [Yes/No: trace A → step 1 → step 2 → B as found in the manuscript]
- **Grounded?** [Yes/No: the foundational source is ... / not cited]
- **Distinguished from rivals?** [Yes/No: rival mechanisms M' and M'' are addressed / ignored]
- **Layer 1 score**: X/10

### Layer 2 — Boundary
- **Contextual conditions stated?** [list them, or "missing"]
- **Temporal conditions stated?** [list, or "missing"]
- **Construct-range conditions stated?** [list, or "missing"]
- **Testable?** [Yes/No]
- **Layer 2 score**: X/10

### Layer 3 — Counter-argument
- **Reverse-direction argument addressed?** [quote the manuscript, or "absent"]
- **Null argument addressed?** [quote, or "absent"]
- **Resolution?** [paraphrase, or "no resolution offered"]
- **Layer 3 score**: X/10

### Flaws Detected
[list every flaw from the catalog with the offending sentence]

### Overall H1 Score: X/10
- ✅ ≥ 7: ready to submit
- ⚠️ 5-6: revise before submission
- 🔴 < 5: rewrite required

### Suggested Rewrite (Claude)
[a rewritten version of H1 + supporting paragraph that addresses the flaws]
```

### Phase 2: Cross-hypothesis check

After the per-H audit, check the **system** of hypotheses:

- **Theoretical coherence**: do the Hs share a unified mechanism story, or are they conceptually unrelated?
- **Argument independence**: does any H rely on another being true? (chained dependency is fragile)
- **Empirical contingency**: are all Hs testable with the available data, or do some require data the author does not have?
- **Hypothesis-finding alignment**: which Hs were supported / unsupported in the results? Are unsupported Hs honestly engaged in the discussion?
- **Hypothesis count**: above 6-7 hypotheses signals scope creep at most FT50 venues; consolidate.

Write a `## System-Level Audit` section.

### Phase 3: Cross-model adversary (DIFFICULTY ≥ hard)

**Skip if DIFFICULTY = medium.**

#### Hard mode

For each hypothesis that Claude scored ≥ 6, send to Codex for an attack:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "high"}
  prompt: |
    You are a senior $TARGET_VENUE reviewer who is highly skeptical of
    convenient hypothesis development. The author claims this hypothesis
    has a strong argument:

    H[N]: [verbatim]

    Supporting paragraph: [verbatim]

    Claude has graded this as X/10. Your job is to attack the grade.

    1. Identify the weakest step in the causal chain
    2. Propose the most plausible rival mechanism that the author did not address
    3. Construct the strongest reverse-direction argument the author would
       struggle to defeat
    4. Decide: would you, as a senior reviewer, demand a major revision of
       this hypothesis at R&R stage 1? Yes / No / Maybe

    Return a 0-10 score (independent of Claude's grade) and a 2-sentence
    rewrite suggestion.
```

For hypotheses Claude scored < 6, send to Codex with a different prompt:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "high"}
  prompt: |
    Author and Claude have flagged this hypothesis as weak (score < 6).
    Your job is to attempt a **rescue**: write the best possible version
    of this hypothesis, including:
    - A named, specified mechanism
    - Defensible boundary conditions
    - Pre-emptive counter-argument resolution

    If the hypothesis is unrescuable (the underlying claim is genuinely
    unsupportable), say so directly and recommend dropping it.

    H[N]: [verbatim]
    Manuscript context: [paragraph]
```

#### Nightmare mode

Same as `auto-review-loop` nightmare: let Codex read the manuscript via `codex exec` and produce an independent audit.

Save raw Codex output under `## Adversary Audit` in `HYPOTHESIS_AUDIT.md`.

### Phase 4: Write the revised hypothesis set

Produce `HYPOTHESES_REVISED.md` — a clean, submission-ready version:

```markdown
# Revised Hypotheses

## H1 — [topic]
**Hypothesis**: [rewritten formal statement]

**Mechanism**: [named, specified, with citation]

**Boundary**: [contextual, temporal, range conditions]

**Counter-argument resolution**: [the strongest reverse-direction argument and why our argument dominates]

**Test**: [how it would be tested with the manuscript's data]

---
```

If any hypothesis is unrescuable, mark it `DROP` and explain in 1-2 sentences why.

### Phase 5: Memory hook

Append to `memory/MEMORY.md` under `## 我的论文项目`:

```markdown
- [$manuscript_id hypothesis audit, $timestamp]: $TARGET_VENUE — N hypotheses, average score X.X/10, flaws: [3 most common types]. See hypothesis-stage/HYPOTHESIS_AUDIT.md
```

## Key Rules

- **One hypothesis at a time**: never audit two hypotheses in the same response — that breeds shortcuts. Audit each in its own pass.
- **Quote verbatim**: the audit is of the manuscript's actual text, not Claude's improved paraphrase. Always cite the line you are scoring.
- **Mechanism > Mediation**: a named mechanism is not the same as a measured mediator. Many authors confuse the two — flag this when it occurs.
- **Boundary is not limitation**: limitations are about scope of the empirical study; boundary conditions are about the theory's claim. Hoist any "limitation" that is actually a boundary condition into the hypothesis itself.
- **Counter-argument honesty**: writing "some might argue X but Y" without seriously engaging X is worse than not raising it. Either engage the reverse argument substantively or remove it.
- **Hypothesis count discipline**: > 6 hypotheses is usually scope creep. Consider consolidating to 3-5 primary + supplementary moderation analyses.
- **Pre-registration alignment**: if the study was pre-registered, ALL hypotheses in the manuscript must match the pre-reg. Any addition is a HARKing red flag.
- **Citation verification**: each citation supporting a hypothesis must be verified to actually support the claim. Use Crossref / DOI lookup, never trust memory.

## Bridging to other skills

- High flaw rate in mechanism layer → run `/contribution-stress-test` first (the contribution claim itself may be the upstream problem)
- All hypotheses score ≥ 7 → run `/auto-review-loop` to validate against a full reviewer simulation
- Pattern of "scope creep" flags across multiple Hs → run `/venue-fit` — the manuscript may be aimed at the wrong journal
- Multiple "boundary missing" flags → boundary conditions may need their own paragraph in the theory section; flag for manuscript edit
