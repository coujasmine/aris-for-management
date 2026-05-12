---
name: contribution-stress-test
description: Hard-test the theoretical contribution of a management research manuscript against the Whetten (1989) building-blocks framework extended by Corley & Gioia (2011). Forces every cell — What / How / Why / Who-Where-When / Originality / Utility — to defend itself, predicts the most likely reviewer rejection memo, and produces a per-cell score with concrete revision instructions. Use when user says "stress test contribution", "Whetten audit", "is my contribution AMJ-worthy".
argument-hint: [manuscript-path-or-paragraph]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill, mcp__codex__codex, mcp__codex__codex-reply
---

# Contribution Stress Test (Whetten 1989 + Corley & Gioia 2011)

The dominant cause of FT50 desk-rejects is unclear or under-developed theoretical contribution. This skill forces a manuscript's contribution claim through the *building blocks of theory* framework and outputs a per-cell scorecard plus a predicted reviewer rejection memo.

> **Whetten (1989)** *AMR*: "What constitutes a theoretical contribution?"
> **Corley & Gioia (2011)** *AMR*: "Building theory about theory building."

## Context: $ARGUMENTS

## Constants

- **TARGET_VENUE = `AMJ`** — One of `AMJ`, `ASQ`, `AMR`, `SMJ`, `JBV`, `SEJ`, `ETP`, `JOM`, `JMS`, `OS`, `RP`, `MS`, `Strategy Science`. AMR is theory-paper-only.
- **PAPER_TYPE = `empirical-quant`** — `empirical-quant` / `empirical-qual` / `mixed-methods` / `meta-analysis` / `theory-paper` / `review-essay`. Affects how strictly each cell is graded.
- **OUTPUT_DIR = `contribution-stage/`**
- **REPORT = `contribution-stage/CONTRIBUTION_AUDIT.md`**
- **DIFFICULTY = `medium`** — `medium` (Claude self-audit) / `hard` (Codex MCP adversary, requires `.mcp.json` codex entry) / `nightmare` (Codex reads manuscript directly via `codex exec`).
- **MIN_PASSING_SCORE = 6** — Below this, the cell is flagged red and must be revised before submission.

> 💡 Override: `/contribution-stress-test "draft.md" — venue: ASQ, difficulty: hard`

## Output Protocols

- Write `contribution-stage/CONTRIBUTION_AUDIT.md` (overwrite each run)
- Append a one-line summary to `memory/MEMORY.md` under "我的论文项目"

## Workflow

### Phase 0: Locate the contribution paragraph

Read the manuscript. Find the paragraph(s) that articulate the theoretical contribution. Common locations:
1. Last 1-2 paragraphs of the introduction
2. Opening of the discussion section ("This study contributes...")
3. A standalone "Theoretical contributions" subsection in the discussion

If multiple candidate paragraphs exist, treat them as a single composite contribution claim. Quote the verbatim text into `CONTRIBUTION_AUDIT.md` under `## Extracted Contribution Claim` so the audit is transparent.

If no clear contribution paragraph exists, that **itself is a finding** — write a brief note and proceed to generate the test questions based on the abstract + intro hook.

### Phase 1: Generate stress-test questions (per cell)

Produce 4-6 questions per cell. Each question must:
- Be directly answerable from the manuscript (yes/no, or with a specific paragraph reference)
- Be the kind of question a senior $TARGET_VENUE reviewer would actually ask

#### Cell W1 — **What** (which constructs / variables / phenomena belong in the theory?)

- W1.1 What are the focal constructs, and how are they operationalized?
- W1.2 Which constructs were **deliberately excluded**, and what is the justification?
- W1.3 Are the constructs already established in the literature, or newly introduced? If new, is the construct validation reported?
- W1.4 Are competing or rival constructs (that could explain the same phenomenon) acknowledged?
- W1.5 For empirical papers: do the measures match the conceptual definitions (face validity)?

#### Cell H1 — **How** (what is the structural relation among the constructs?)

- H1.1 Direct effect, mediation, moderation, or interaction? Specified clearly?
- H1.2 Causal direction: is the temporal ordering defensible?
- H1.3 Is non-linearity considered (e.g., inverted U, threshold)? Why or why not?
- H1.4 For mediation: are the steps theoretically motivated *before* being tested?
- H1.5 For moderation: is the interaction grounded in theory, not opportunistic?

#### Cell Y1 — **Why** (what mechanism explains the relationship?)

⚠️ **This is the #1 R&R kill-zone.** Most rejections cite "thin theoretical mechanism."

- Y1.1 Is the mechanism named and specified, not merely implied?
- Y1.2 Is the mechanism causally connected to both the antecedent and the outcome?
- Y1.3 Are **rival mechanisms** identified and ruled out (theoretically or empirically)?
- Y1.4 Does the manuscript distinguish "the mechanism we propose" from "what the data show happens"?
- Y1.5 Is the mechanism falsifiable — i.e., what evidence would refute it?

#### Cell WWW — **Who / Where / When** (boundary and scope conditions)

- WWW.1 To which organizations / industries / regions does the theory apply?
- WWW.2 What institutional, cultural, or temporal conditions are required?
- WWW.3 Where would the theory **fail**, and why?
- WWW.4 For Chinese-context studies: is the role of context theorized, or just acknowledged as a "limitation"?
- WWW.5 Are the boundaries tested empirically (subgroup analysis, moderation by context)?

#### Cell ORIG — **Originality** (per Corley & Gioia 2011)

Pick the most accurate label for the contribution and defend it:
- **Incremental** — refines/extends an existing theory
- **Revelatory** — reveals a phenomenon or relationship not previously seen
- **Revolutionary** — overturns a prior framework

- ORIG.1 Which label fits, and what is the evidence?
- ORIG.2 If incremental: is the increment **non-trivial** (not just a context replication)?
- ORIG.3 If revelatory: is the phenomenon genuinely new, or just newly named?
- ORIG.4 What specific prior paper does this extend / challenge / replace?

#### Cell UTIL — **Utility** (scientific + practical)

- UTIL.1 Scientific utility: what new research questions does this open?
- UTIL.2 Practical utility: what can a manager / entrepreneur / policymaker now do that they could not before?
- UTIL.3 Is utility *demonstrated* via examples, or only *claimed*?
- UTIL.4 For $TARGET_VENUE specifically: does utility match the journal's editorial priorities?

### Phase 2: Self-audit (Claude answers)

For every question, write a structured response in `CONTRIBUTION_AUDIT.md`:

```markdown
### W1.1 What are the focal constructs?
- **Answer from manuscript**: [verbatim quote or paragraph reference]
- **Strength**: [strong / adequate / weak / missing]
- **Issue (if any)**: [what would a $TARGET_VENUE reviewer say?]
```

Be honest. If the manuscript does not answer a question, mark it **missing** — do not paper over gaps.

### Phase 3: Cross-model adversary (DIFFICULTY = hard or nightmare)

**Skip this phase if DIFFICULTY = medium.**

#### Hard mode — Codex MCP

Send the full audit-in-progress to Codex with this prompt:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "high"}
  prompt: |
    You are a senior reviewer at $TARGET_VENUE evaluating the theoretical
    contribution of the attached manuscript. The author has self-audited
    against the Whetten + Corley & Gioia framework. Your job is to
    challenge their self-grading.

    For every cell where the author claims "strong" or "adequate":
    - Identify the weakest part of the answer
    - State which $TARGET_VENUE reviewer would push back, and how
    - Suggest the MINIMUM revision (≤ 2 sentences in the manuscript) that
      would change your mind

    For every cell where the author admits "weak" or "missing":
    - Confirm the gap is real, or argue it is not actually fatal
    - Specify the minimum required addition

    Output format:
    - Per-cell rating: 0-10
    - Top 3 reasons this contribution would not survive desk review
    - Predicted reviewer rejection memo (300 words, in the actual voice
      of a $TARGET_VENUE senior reviewer at R&R stage 1)

    [paste contents of CONTRIBUTION_AUDIT.md so far]
    [paste extracted contribution paragraph from Phase 0]
```

Save the threadId for follow-up.

#### Nightmare mode — Codex exec

Let Codex read the manuscript directly (Claude does not pre-filter):

```bash
codex exec "$(cat <<'PROMPT'
You are a senior reviewer at $TARGET_VENUE. You have full read access to
the manuscript files in this repository. Independently locate and read:
1. The introduction
2. The contribution paragraph(s)
3. The hypothesis development section
4. The discussion of theoretical contributions

Then audit the contribution against Whetten (1989) and Corley & Gioia
(2011) using the same 6 cells. Output the same format as hard mode.

Crucially: cross-check whether the author's characterization of prior
literature is accurate. Misattribution of cited work is grounds for
desk reject.
PROMPT
)" --skip-git-repo-check
```

Save the full Codex output verbatim into `CONTRIBUTION_AUDIT.md` under `## Adversary Audit (Codex)`.

### Phase 4: Score & report

For each of the 6 cells, assign a 0-10 score using this rubric:

| Score | Meaning |
|---|---|
| 9-10 | Exemplary — would survive an AMR audit |
| 7-8 | Strong — minor polish only |
| 5-6 | Adequate — visible weaknesses, will draw reviewer comments but survivable |
| 3-4 | Weak — revise before submission |
| 0-2 | Missing / fatal — current draft will be desk-rejected on this cell alone |

If DIFFICULTY ≥ hard, use the adversary's per-cell ratings, not Claude's self-grade.

Then write the report:

```markdown
# CONTRIBUTION_AUDIT.md — $manuscript_id

Generated: $timestamp
Target venue: $TARGET_VENUE
Paper type: $PAPER_TYPE
Difficulty: $DIFFICULTY

## Extracted Contribution Claim
[verbatim paragraph(s) from Phase 0]

## Scorecard

| Cell | Score | Status |
|------|-------|--------|
| W (What)            | X/10 | ✅ / ⚠️ / 🔴 |
| H (How)             | X/10 | ✅ / ⚠️ / 🔴 |
| Y (Why)             | X/10 | ✅ / ⚠️ / 🔴 |
| WWW (Who/Where/When)| X/10 | ✅ / ⚠️ / 🔴 |
| ORIG (Originality)  | X/10 | ✅ / ⚠️ / 🔴 |
| UTIL (Utility)      | X/10 | ✅ / ⚠️ / 🔴 |
| **Overall**         | X/60 | |

✅ ≥ 7  ⚠️ 5-6  🔴 < 5

## Red-Flag Cells (must fix before submission)
[list every cell scoring < MIN_PASSING_SCORE with specific revision instructions]

## Predicted Reviewer Rejection Memo
[300-word memo simulating the senior reviewer voice at $TARGET_VENUE]

## Per-Cell Detail
[full Phase 2 + Phase 3 contents, cell by cell]

## Recommended Next Action
[one of:
  "Revise red cells then re-run with DIFFICULTY=hard"
  "Run /hypothesis-builder on H1-Hn to address Why/How cell gaps"
  "Run /theory-positioning — current contribution lacks a clear parent theory"
  "Ready to draft cover letter and submit to $TARGET_VENUE"
]
```

### Phase 5: Memory hook

Append one line to `memory/MEMORY.md` under `## 我的论文项目`:

```markdown
- [$manuscript_id contribution audit, $timestamp]: $TARGET_VENUE fit X/60 — red cells: [Why, WWW]. See contribution-stage/CONTRIBUTION_AUDIT.md
```

## Key Rules

- **Quote, don't paraphrase**: when extracting the contribution claim, paste the verbatim text. Auditing a paraphrase audits Claude, not the manuscript.
- **Cell independence**: a strong What does not rescue a weak Why. Score each cell independently.
- **Be brutal on Why**: thin mechanism is the modal R&R rejection reason in AMJ/ASQ. Default to skepticism.
- **Context theorization rule**: for studies in Chinese / emerging market contexts, "Chinese context" is not itself a contribution. The theoretical role of context (boundary, mechanism, or moderator) must be specified.
- **No score inflation under adversary**: if hard/nightmare mode lowers a cell's score, do not split the difference. The adversary is the better signal.
- **Falsifiability is non-negotiable**: any Why-cell claim that cannot, in principle, be refuted, scores ≤ 3.
- **Citation honesty**: when the audit references prior work, verify the citation supports the claim before including it. Use Crossref / DOI verification — never invent BibTeX.

## Bridging to other skills

- If cell Y (Why) scores < 5 → run `/hypothesis-builder` to rebuild argument chains
- If cell ORIG scores < 5 → run `/theory-positioning` to re-anchor in a parent theory
- If overall < 30/60 → run `/venue-fit` — current draft may be better matched to a different journal
- After fixing red cells → run `/auto-review-loop` to validate against a full reviewer simulation
