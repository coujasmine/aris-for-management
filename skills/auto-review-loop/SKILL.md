---
name: auto-review-loop
description: Autonomous multi-round management research review loop (AMJ/ASQ/SMJ/JBV-level). Repeatedly reviews via Codex MCP simulating an FT50/UTD24 senior reviewer, implements revisions, and re-reviews until positive assessment or max rounds reached. Use when user says "auto review loop", "review until it passes", or wants autonomous iterative improvement on a management research manuscript.
argument-hint: [topic-or-scope]
allowed-tools: Bash(*), Read, Grep, Glob, Write, Edit, Agent, Skill, mcp__codex__codex, mcp__codex__codex-reply
---

# Auto Review Loop: Management Research (FT50 / UTD24)

Autonomously iterate: review → implement revisions → re-review, until the external reviewer (simulating an AMJ / ASQ / AMR / SMJ / JBV / SEJ / ETP / JOM / RP / OS senior reviewer at the target venue) gives a positive assessment or MAX_ROUNDS is reached.

**Adapted from ARIS** (originally NeurIPS/ICML-oriented) for management research. Evaluation criteria emphasize theoretical contribution (Whetten 1989 / Corley & Gioia 2011), hypothesis development, construct validity, identification strategy, and engagement with prior literature — not raw experimental performance.

## Context: $ARGUMENTS

## Constants

- MAX_ROUNDS = 4
- POSITIVE_THRESHOLD: score >= 6/10, or verdict contains "accept", "sufficient", "ready for submission"
- REVIEW_DOC: `review-stage/AUTO_REVIEW.md` (cumulative log) *(fall back to `./AUTO_REVIEW.md` for legacy projects)*
- REVIEWER_MODEL = `gpt-latest` — Model used via Codex MCP. Use whichever OpenAI model your Codex CLI currently routes to (auto-updates as OpenAI releases new versions). Do NOT hardcode an exact version number.
- **TARGET_VENUE = `AMJ`** — Target journal for reviewer simulation. Supported: `AMJ`, `ASQ`, `AMR`, `SMJ`, `JBV`, `SEJ`, `ETP`, `JOM`, `RP`, `OS`. Affects reviewer criteria weighting (theory-heavy: AMR/ASQ; phenomena-friendly: JBV/SEJ/ETP; method-heavy: SMJ).
- **PAPER_TYPE = `empirical-quant`** — One of: `empirical-quant` (regression/SEM/HLM), `empirical-qual` (case/grounded/ethnography), `mixed-methods`, `meta-analysis`, `theory-paper`, `review-essay`. Affects which revision tracks Phase C prioritizes.
- **REVIEWER_BACKEND = `codex`** — Default: Codex MCP (xhigh). Override with `— reviewer: oracle-pro` for GPT-5.4 Pro via Oracle MCP. See `shared-references/reviewer-routing.md`.
- **OUTPUT_DIR = `review-stage/`** — All review-stage outputs go here. Create the directory if it doesn't exist.
- **HUMAN_CHECKPOINT = false** — When `true`, pause after each round's review (Phase B) and present the score + weaknesses to the user. Wait for user input before proceeding to Phase C. The user can: approve the suggested fixes, provide custom modification instructions, skip specific fixes, or stop the loop early. When `false` (default), the loop runs fully autonomously.
- **COMPACT = false** — When `true`, (1) read `EXPERIMENT_LOG.md` and `findings.md` instead of parsing full logs on session recovery, (2) append key findings to `findings.md` after each round.
- **REVIEWER_DIFFICULTY = medium** — Controls how adversarial the reviewer is. Three levels:
  - `medium` (default): Current behavior — MCP-based review, Claude controls what context GPT sees.
  - `hard`: Adds **Reviewer Memory** (GPT tracks its own suspicions across rounds) + **Debate Protocol** (Claude can rebut, GPT rules).
  - `nightmare`: Everything in `hard` + **GPT reads the repo directly** via `codex exec` (Claude cannot filter what GPT sees) + **Adversarial Verification** (GPT independently checks if code matches claims).

> 💡 Override: `/auto-review-loop "topic" — compact: true, human checkpoint: true, difficulty: hard`

## State Persistence (Compact Recovery)

Long-running loops may hit the context window limit, triggering automatic compaction. To survive this, persist state to `review-stage/REVIEW_STATE.json` after each round:

```json
{
  "round": 2,
  "threadId": "019cd392-...",
  "status": "in_progress",
  "difficulty": "medium",
  "last_score": 5.0,
  "last_verdict": "not ready",
  "pending_experiments": ["screen_name_1"],
  "timestamp": "2026-03-13T21:00:00"
}
```

**Write this file at the end of every Phase E** (after documenting the round). Overwrite each time — only the latest state matters.

**On completion** (positive assessment or max rounds), set `"status": "completed"` so future invocations don't accidentally resume a finished loop.

## Output Protocols

> Follow these shared protocols for all output files:
> - **[Output Versioning Protocol](../shared-references/output-versioning.md)** — write timestamped file first, then copy to fixed name
> - **[Output Manifest Protocol](../shared-references/output-manifest.md)** — log every output to MANIFEST.md
> - **[Output Language Protocol](../shared-references/output-language.md)** — respect the project's language setting

## Workflow

### Initialization

1. **Check for `review-stage/REVIEW_STATE.json`** *(fall back to `./REVIEW_STATE.json` if not found — legacy path)*:
   - If neither path exists: **fresh start** (normal case, identical to behavior before this feature existed)
   - If it exists AND `status` is `"completed"`: **fresh start** (previous loop finished normally)
   - If it exists AND `status` is `"in_progress"` AND `timestamp` is older than 24 hours: **fresh start** (stale state from a killed/abandoned run — delete the file and start over)
   - If it exists AND `status` is `"in_progress"` AND `timestamp` is within 24 hours: **resume**
     - Read the state file to recover `round`, `threadId`, `last_score`, `pending_experiments`
     - Read `review-stage/AUTO_REVIEW.md` to restore full context of prior rounds *(fall back to `./AUTO_REVIEW.md`)*
     - If `pending_experiments` is non-empty, check if they have completed (e.g., check screen sessions)
     - Resume from the next round (round = saved round + 1)
     - Log: "Recovered from context compaction. Resuming at Round N."
2. Read project narrative documents, memory files, and any prior review documents. **When `COMPACT = true` and compact files exist**: read `findings.md` + `EXPERIMENT_LOG.md` instead of full `review-stage/AUTO_REVIEW.md` and raw logs — saves context window.
3. Read recent experiment results (check output directories, logs)
4. Identify current weaknesses and open TODOs from prior reviews
5. Initialize round counter = 1 (unless recovered from state file)
6. Create/update `review-stage/AUTO_REVIEW.md` with header and timestamp

### Loop (repeat up to MAX_ROUNDS)

#### Phase A: Review

**Route by REVIEWER_DIFFICULTY:**

##### Medium (default) — MCP Review

Send comprehensive context to the external reviewer:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N/MAX_ROUNDS of autonomous review loop]

    [Full research context: claims, methods, results, known weaknesses]
    [Changes since last round, if any]

    Please act as a senior management research reviewer at $TARGET_VENUE level (FT50 / UTD24 tier).

    Evaluation rubric (apply with weights appropriate to $TARGET_VENUE and $PAPER_TYPE):

    1. **Theoretical contribution** (Whetten 1989 / Corley & Gioia 2011)
       - What new conceptual ground does this break? (what/how/why/who/when/where)
       - Is the contribution's scope, originality, and utility clearly articulated?
       - Does it advance, challenge, or extend an identifiable theoretical conversation?

    2. **Engagement with literature**
       - Are the right theoretical conversations identified?
       - Does the paper build on, distinguish from, AND extend prior work?
       - Glaring omissions (key papers from the past 5 years; competing theoretical lenses)?

    3. **Hypothesis development** (for empirical papers) / Proposition development (for theory papers)
       - Are arguments grounded in mechanisms, not just analogies or stylized facts?
       - Are boundary conditions and scope conditions specified?
       - Is the logic chain free of unwarranted leaps?

    4. **Methodology**
       - Sample representativeness, selection rationale, response rates
       - Construct validity (validated scales, α, CR, AVE, discriminant validity)
       - Identification strategy (endogeneity, reverse causality, omitted variables, selection)
       - Appropriate analysis given the question (OLS / FE / IV / DiD / PSM / SEM / HLM / mediation / moderation)
       - Common-method bias (Harman, marker variable, CFA-based) where self-report data is used

    5. **Results & robustness**
       - Are findings replicable across specifications, samples, alternative measures?
       - Are alternative explanations and rival hypotheses ruled out?
       - Effect sizes meaningful, not merely statistically significant?

    6. **Discussion & implications**
       - Theoretical implications clearly tied back to the contribution claims
       - Practical/policy implications grounded, not hand-waved
       - Limitations honestly acknowledged (not buried in a self-serving paragraph)

    Provide in your response:
    1. **Score** 1-10 specifically for $TARGET_VENUE (anchor: 6 = R&R territory, 8 = competitive submission, 10 = exemplar)
    2. **Critical weaknesses** ranked by severity
    3. For each weakness, the **MINIMUM revision** that would address it (additional analysis, theoretical reframing, robustness check, citation work, or reformulation)
    4. **Verdict**: READY / ALMOST / NOT READY for submission
    5. **Desk-reject risk** (high / medium / low) — would the editor send this out for review, or reject without external review?

    Be brutally honest. Channel an AMJ/ASQ senior reviewer who has personally desk-rejected or rejected ~80% of submissions this year. If the work is genuinely ready, say so clearly.
```

If this is round 2+, use `mcp__codex__codex-reply` with the saved threadId to maintain conversation context.

##### Hard — MCP Review + Reviewer Memory

Same as medium, but **prepend Reviewer Memory** to the prompt:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N/MAX_ROUNDS of autonomous review loop]

    ## Your Reviewer Memory (persistent across rounds)
    [Paste full contents of REVIEWER_MEMORY.md here]

    IMPORTANT: You have memory from prior rounds. Check whether your
    previous suspicions were genuinely addressed or merely sidestepped.
    The author (Claude) controls what context you see — be skeptical
    of convenient omissions.

    [Full research context, changes since last round...]

    Please act as a senior management research reviewer at $TARGET_VENUE level (FT50/UTD24).
    Apply the full rubric: theoretical contribution / literature engagement /
    hypothesis or proposition development / methodology & identification /
    results & robustness / discussion & implications.

    1. Score 1-10 specifically for $TARGET_VENUE
    2. Critical weaknesses, ranked by severity
    3. For each weakness, the MINIMUM revision required
    4. Verdict: READY / ALMOST / NOT READY
    5. Desk-reject risk: high / medium / low
    6. **Memory update**: New suspicions, unresolved concerns, or patterns
       to track in future rounds. Pay particular attention to:
       - Recurring "convenient framings" of the theoretical contribution
       - Whether new analyses actually address prior concerns or merely sidestep them
       - Citation cherry-picking (favorable studies cited, contrary evidence omitted)
       - Hypothesis-data alignment shifts between rounds (HARKing risk)

    Be brutally honest. Channel an AMJ senior reviewer at R&R stage 3,
    who has read this revision three times already and is now actively looking
    for things the author might be hiding behind academic prose.
```

##### Nightmare — Codex Exec (GPT reads repo directly)

**Do NOT use MCP.** Instead, let GPT access the repo autonomously via `codex exec`:

```bash
codex exec "$(cat <<'PROMPT'
You are an adversarial senior management research reviewer at $TARGET_VENUE level (FT50/UTD24).
This is Round N/MAX_ROUNDS of an autonomous review loop.

## Your Reviewer Memory (persistent across rounds)
[Paste full contents of REVIEWER_MEMORY.md]

## Instructions
You have FULL READ ACCESS to this repository. The author (Claude) does NOT
control what you see — explore freely. Your job is to find problems the
author might hide or downplay.

DO THE FOLLOWING:
1. Read the data files (CSV / .dta / .RData / Excel), analysis scripts (.do / .R / .py / .sps),
   and result tables / log files YOURSELF
2. Verify that reported coefficients, standard errors, sample sizes, and fit indices
   match the actual output (not what the manuscript claims)
3. Check whether construct measurement is documented — which validated scales,
   with what reliability (α / CR) and validity (AVE, discriminant) statistics
4. Look for: cherry-picked specifications, omitted standard controls, suspicious
   sample restrictions, dropped observations without justification, missing
   robustness checks, post-hoc hypothesizing (HARKing)
5. Read the manuscript draft (paper-stage/ or root .tex / .docx / .md)
   for the author's claims — then verify each against the data, code, and result files
6. Cross-check cited literature: do the cited works actually say what the author
   claims they say? Use the manuscript bibliography as your starting list

OUTPUT FORMAT:
- Score: X/10 for $TARGET_VENUE
- Verdict: ready / almost / not ready
- Desk-reject risk: high / medium / low
- Verified claims: [empirical and theoretical claims you independently confirmed]
- Unverified or false claims: [claims that don't match the data, code, or cited sources]
- Weaknesses (ranked): theory / literature / hypothesis / method / results / discussion
  — with MINIMUM revision for each
- Memory update: [new suspicions and patterns to track next round]

Be adversarial. Channel a reviewer who has personally been burned by p-hacked
or theory-light papers in the past and is now allergic to convenient framings.
Trust nothing the author tells you — verify everything yourself, including their
characterizations of prior literature.
PROMPT
)" --skip-git-repo-check 2>&1
```

**Key difference**: In nightmare mode, GPT independently reads code, result files, and logs. Claude cannot filter or curate what GPT sees. This is the closest analog to a real hostile reviewer who reads your actual paper + supplementary materials.

#### Phase B: Parse Assessment

**CRITICAL: Save the FULL raw response** from the external reviewer verbatim (store in a variable for Phase E). Do NOT discard or summarize — the raw text is the primary record.

Then extract structured fields:
- **Score** (numeric 1-10)
- **Verdict** ("ready" / "almost" / "not ready")
- **Action items** (ranked list of fixes)

**STOP CONDITION**: If score >= 6 AND verdict contains "ready" or "almost" → stop loop, document final state.

#### Phase B.5: Reviewer Memory Update (hard + nightmare only)

**Skip entirely if `REVIEWER_DIFFICULTY = medium`.**

After parsing the assessment, update `REVIEWER_MEMORY.md` in the project root:

```markdown
# Reviewer Memory

## Round 1 — Score: X/10
- **Suspicion**: [what the reviewer flagged]
- **Unresolved**: [concerns not yet addressed]
- **Patterns**: [recurring issues the reviewer noticed]

## Round 2 — Score: X/10
- **Previous suspicions addressed?**: [yes/no for each, with reviewer's judgment]
- **New suspicions**: [...]
- **Unresolved**: [carried forward + new]
```

**Rules**:
- Append each round, never delete prior rounds (audit trail)
- If the reviewer's response includes a "Memory update" section, copy it verbatim
- This file is passed back to GPT in the next round's Phase A — it is GPT's persistent brain

#### Phase B.6: Debate Protocol (hard + nightmare only)

**Skip entirely if `REVIEWER_DIFFICULTY = medium`.**

After parsing the review, Claude (the author) gets a chance to **rebut**:

**Step 1 — Claude's Rebuttal:**

For each weakness the reviewer identified, Claude writes a structured response:

```markdown
### Rebuttal to Weakness #1: [title]
- **Accept / Partially Accept / Reject**
- **Argument**: [why this criticism is invalid, already addressed, or based on a misunderstanding]
- **Evidence**: [point to specific code, results, or prior round fixes]
```

Rules for Claude's rebuttal:
- Must be honest — do NOT fabricate evidence or misrepresent results
- Can point out factual errors in the review (reviewer misread code, wrong metric, etc.)
- Can argue a weakness is out of scope or would require unreasonable effort
- Maximum 3 rebuttals per round (pick the most impactful to contest)

**Step 2 — GPT Rules on Rebuttal:**

Send Claude's rebuttal back to GPT for a ruling:

*Hard mode (MCP):*
```
mcp__codex__codex-reply:
  threadId: [saved]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    The author rebuts your review:

    [paste Claude's rebuttal]

    For each rebuttal, rule:
    - SUSTAINED (author's argument is valid, withdraw this weakness)
    - OVERRULED (your original criticism stands, explain why)
    - PARTIALLY SUSTAINED (revise the weakness to a narrower scope)

    Then update your score if any weaknesses were withdrawn.
```

*Nightmare mode (codex exec):*
```bash
codex exec "$(cat <<'PROMPT'
You are the same adversarial reviewer. The author rebuts your review:

[paste Claude's rebuttal]

VERIFY the author's evidence claims yourself — read the files they reference.
Do NOT take their word for it.

For each rebuttal, rule:
- SUSTAINED (verified and valid)
- OVERRULED (evidence doesn't check out or argument is weak)
- PARTIALLY SUSTAINED (partially valid, narrow the weakness)

Update your score. Update your memory.
PROMPT
)" --skip-git-repo-check 2>&1
```

**Step 3 — Update score and action items** based on the ruling:
- SUSTAINED weaknesses: remove from action items
- OVERRULED: keep as-is
- PARTIALLY SUSTAINED: revise scope

Append the full debate transcript to `review-stage/AUTO_REVIEW.md` under the round's entry.

#### Human Checkpoint (if enabled)

**Skip this step entirely if `HUMAN_CHECKPOINT = false`.**

When `HUMAN_CHECKPOINT = true`, present the review results and wait for user input:

```
📋 Round N/MAX_ROUNDS review complete.

Score: X/10 — [verdict]
Top weaknesses:
1. [weakness 1]
2. [weakness 2]
3. [weakness 3]

Suggested fixes:
1. [fix 1]
2. [fix 2]
3. [fix 3]

Options:
- Reply "go" or "continue" → implement all suggested fixes
- Reply with custom instructions → implement your modifications instead
- Reply "skip 2" → skip fix #2, implement the rest
- Reply "stop" → end the loop, document current state
```

Wait for the user's response. Parse their input:
- **Approval** ("go", "continue", "ok", "proceed"): proceed to Phase C with all suggested fixes
- **Custom instructions** (any other text): treat as additional/replacement guidance for Phase C. Merge with reviewer suggestions where appropriate
- **Skip specific fixes** ("skip 1,3"): remove those fixes from the action list
- **Stop** ("stop", "enough", "done"): terminate the loop, jump to Termination

#### Feishu Notification (if configured)

After parsing the score, check if `~/.claude/feishu.json` exists and mode is not `"off"`:
- Send a `review_scored` notification: "Round N: X/10 — [verdict]" with top 3 weaknesses
- If **interactive** mode and verdict is "almost": send as checkpoint, wait for user reply on whether to continue or stop
- If config absent or mode off: skip entirely (no-op)

#### Phase C: Implement Revisions (if not stopping)

For each action item (highest priority first):

1. **Theoretical work**: Sharpen contribution claims, add boundary conditions,
   integrate missing key citations, reframe positioning against prior literature,
   strengthen the "why" mechanism behind each hypothesis
2. **Hypothesis / proposition development**: Tighten logic chains, add mechanism
   arguments, specify scope conditions, address counterarguments preemptively,
   eliminate analogical reasoning where causal mechanism is required
3. **Additional empirical analyses** (for empirical papers):
   - Robustness checks (alternative measures, alternative samples, alternative estimators)
   - Endogeneity treatments (IV, PSM, DiD, Heckman, GMM, regression discontinuity)
   - Post-hoc tests (subsample analysis, mediation/moderation, sensitivity bounds)
   - Common-method bias remedies for self-report studies
4. **Construct measurement work**: Document scale sources and items, compute
   reliability (Cronbach α, CR), validity (CFA loadings, AVE, discriminant
   via Fornell-Larcker or HTMT); add bias diagnostics
5. **Manuscript edits**: Update introduction (hook + gap + contribution),
   theory section, methods (including all of the above), results tables,
   discussion (theoretical AND practical implications), limitations
6. **Citation / literature work**: Add missing key papers (top 5 years for the
   target venue + foundational works); verify every citation actually supports
   the claim made; standardize to APA 7 (or venue style)
7. **Documentation**: Update project notes (memory/) and review document

Prioritization rules:
- Theoretical reframing first when feasible (highest ROI, lowest cost)
- For empirical revisions: prefer additional analyses on existing data over new data collection
- Skip revisions requiring 6+ months of new data collection — flag for human scoping
- Always implement citation / literature gaps (cheap, high reviewer satisfaction)
- For qualitative papers: prefer recoding / reanalysis with new theoretical lens over collecting additional cases
- For theory papers: prefer adding a counterargument-then-resolution section over removing scope

#### Phase D: Verify Revised Results

After analyses are re-run (or theoretical sections are rewritten):
- Re-extract coefficients, standard errors, sample sizes, fit indices from the latest analysis output. Do NOT carry numbers over from a prior round — every reported number must match the latest run.
- Update all results tables, figures, and in-text reported statistics in the manuscript
- **Robustness reconciliation** — confirm main effects survive across the new robustness specifications. If a key result no longer holds, **flag this prominently** in the next review round. Do NOT hide a now-fragile result.
- For new measurement work: verify reliability and validity stats meet conventional cutoffs (Cronbach α > .70, CR > .70, AVE > .50, discriminant validity per Fornell-Larcker or HTMT < .85)
- Save analysis logs, do-files, R scripts, output to `analysis-stage/` so the reviewer can re-verify in `nightmare` mode

#### Phase E: Document Round

Append to `review-stage/AUTO_REVIEW.md`:

```markdown
## Round N (timestamp)

### Assessment (Summary)
- Score: X/10
- Verdict: [ready/almost/not ready]
- Key criticisms: [bullet list]

### Reviewer Raw Response

<details>
<summary>Click to expand full reviewer response</summary>

[Paste the COMPLETE raw response from the external reviewer here — verbatim, unedited.
This is the authoritative record. Do NOT truncate or paraphrase.]

</details>

### Debate Transcript (hard + nightmare only)

<details>
<summary>Click to expand debate</summary>

**Claude's Rebuttal:**
[paste rebuttal]

**GPT's Ruling:**
[paste ruling — SUSTAINED / OVERRULED / PARTIALLY SUSTAINED for each]

**Score adjustment**: X/10 → Y/10

</details>

### Actions Taken
- [what was implemented/changed]

### Results
- [analysis outcomes, robustness check results, new measurement statistics, or theoretical reframing produced, if any]

### Status
- [continuing to round N+1 / stopping]
- Difficulty: [medium/hard/nightmare]
```

**Write `review-stage/REVIEW_STATE.json`** with current round, threadId, score, verdict, and any pending analyses (long-running SEM, simulations, meta-analyses, etc.).

**Append to `findings.md`** (when `COMPACT = true`): one-line entry per key finding this round:

```markdown
- [Round N] [positive/negative/unexpected]: [one-sentence finding] (metric: X.XX → Y.YY)
```

Increment round counter → back to Phase A.

### Termination

When loop ends (positive assessment or max rounds):

1. Update `review-stage/REVIEW_STATE.json` with `"status": "completed"`
2. Write final summary to `review-stage/AUTO_REVIEW.md`
3. Update project notes with conclusions
4. **Write method/pipeline description** to `review-stage/AUTO_REVIEW.md` under a `## Method Description` section — a concise 1-2 paragraph description of the final method, its architecture, and data flow. This serves as input for `/paper-illustration` in Workflow 3 (so it can generate architecture diagrams automatically).
5. **Generate claims from results** — invoke `/result-to-claim` to convert experiment results from `review-stage/AUTO_REVIEW.md` into structured paper claims. Output: `CLAIMS_FROM_RESULTS.md`. This bridges Workflow 2 → Workflow 3 so `/paper-plan` can directly use validated claims instead of extracting them from scratch. If `/result-to-claim` is not available, skip silently.
6. If stopped at max rounds without positive assessment:
   - List remaining blockers
   - Estimate effort needed for each
   - Suggest whether to continue manually or pivot
5. **Feishu notification** (if configured): Send `pipeline_done` with final score progression table

## Key Rules

- **Large file handling**: If the Write tool fails due to file size, immediately retry using Bash (`cat << 'EOF' > file`) to write in chunks. Do NOT ask the user for permission — just do it silently.

- ALWAYS use `config: {"model_reasoning_effort": "xhigh"}` for maximum reasoning depth
- Save threadId from first call, use `mcp__codex__codex-reply` for subsequent rounds
- **Anti-hallucination citations**: When adding references during revisions, NEVER fabricate BibTeX, DOIs, or page numbers. Management citation chain: (1) if DOI is known, `curl -sLH "Accept: application/x-bibtex" "https://doi.org/{doi}"`; (2) otherwise CrossRef: `curl -s "https://api.crossref.org/works?query.bibliographic=TITLE+AUTHOR&rows=1"`; (3) fall back to Google Scholar / Web of Science / Scopus lookup; (4) if all fail, mark with `[VERIFY-CITATION]` in the draft and add to a TODO list. Never invent BibTeX from memory. **Also verify the cited work actually supports the claim you attribute to it** — misattribution is a common rejection reason.
- Be honest — include null results, failed robustness checks, and unsupported hypotheses
- Do NOT hide weaknesses to game a positive score
- Implement revisions BEFORE re-reviewing (don't just promise to fix in a future round)
- **Exhaust before surrendering** — before marking any reviewer concern as "cannot address": (1) try at least 2 different solution paths, (2) for empirical issues, try alternative specifications, alternative measures, or subsample analyses, (3) for theoretical issues, provide a weaker version of the claim, an alternative theoretical lens, or scope the contribution more narrowly, (4) only then concede and bound the damage. Never surrender on the first attempt.
- Long-running analyses (large meta-analyses, complex SEM, simulations): launch the script in R / STATA / Mplus batch mode and continue with theory / citation / writing work while it runs
- Document EVERYTHING — the review log should be self-contained
- Update project notes after each round, not just at the end

## Prompt Template for Round 2+

```
mcp__codex__codex-reply:
  threadId: [saved from round 1]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N update]

    Since your last review, we have:
    1. [Revision 1 — theory / hypothesis / methods / analysis / discussion]:
       [outcome: what changed, what's now in the manuscript / data / code]
    2. [Revision 2]: [outcome]
    3. [Revision 3]: [outcome]

    Updated results table / contribution paragraph / hypothesis section:
    [paste the updated table or text]

    Please re-score and re-assess for $TARGET_VENUE. Are the remaining concerns addressed?
    Same format: Score, Verdict, Desk-reject risk, Remaining Weaknesses, Minimum Revisions.
```

## Review Tracing

After each `mcp__codex__codex` or `mcp__codex__codex-reply` reviewer call, save the trace following `shared-references/review-tracing.md`. Use `tools/save_trace.sh` or write files directly to `.aris/traces/<skill>/<date>_run<NN>/`. Respect the `--- trace:` parameter (default: `full`).
