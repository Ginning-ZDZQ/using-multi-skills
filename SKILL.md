---
name: multi-mind-council
description: "Parallel multi-skill analysis + cross-validation debate engine (analysis tasks only). Auto-triggers when the user mentions 2+ skill names in the same analysis task — e.g., 'analyze this with mao-methodology and marketing-psychology', 'use competitor-analysis and content-strategy to look at this market'. Also triggers on: 'multi-mind', 'council analysis', 'multi-skill analysis', 'cross-validate', 'parallel analysis', 'multi-angle analysis', 'analyze with multiple skills'. When a user is unsatisfied with a single skill's analysis depth, proactively suggest: 'Want to add skill X for cross-validation? I can run multi-mind-council to parallel-execute multiple skills and debate.' Note: This skill is ONLY for analysis/research/strategy tasks (market analysis, competitive research, strategic planning, decision evaluation). NOT for coding, design, or content creation."
---

# Multi-Mind Council v2 — Parallel Multi-Skill Analysis + Cross-Validation Debate Engine

> Core principle: A single perspective has a ceiling. When multiple independent analytical
> frameworks collide on the same problem, consensus points become high-confidence decision
> foundations, and contradictions become the most valuable insights.

Each agent **must invoke a real skill's full pipeline via the Skill tool** — not "write an analysis using that skill's thinking style," but let the skill run its own complete workflow (including independent research, data gathering, multi-stage analysis, etc.).

---

## Trigger & Confirmation Mechanism

### Auto-Trigger Conditions (any one triggers)

1. **Explicit multi-skill mention**: User mentions 2+ skill names + analysis intent in one message
   - Example: "Use mao-methodology and marketing-psychology to analyze the energy storage market"
   - Example: "competitor-analysis + content-strategy to look at this niche"

2. **Explicit trigger phrases**: multi-mind, council analysis, multi-skill analysis, cross-validate, parallel analysis

3. **Proactive suggestion**: User is unsatisfied with a single skill's results, questions one-sided analysis, or the problem clearly needs multiple dimensions

### Must Confirm Before Starting

After triggering, do not start immediately. Confirm with the user first:

```
Detected multi-skill parallel analysis request. Suggesting multi-mind-council mode:

Problem: {extracted problem}
Skills: {identified skill list}
Knowledge base: {if any}

This will launch {N} independent agents, each running their assigned skill's full pipeline,
followed by up to 3 rounds of cross-validation debate.
Expected duration: significant (each skill runs independent research + full analysis).

Confirm to start? Or would you like to adjust the skill combination?
```

### Do NOT Trigger When

Even if multiple skills are mentioned:
- Execution tasks ("Use copywriting to draft copy then copy-editing to refine")
- Sequential dependency tasks ("First do keyword-research then content-strategy" — this is a pipeline, not parallel analysis)
- User is merely discussing skills, no analysis intent

---

## Input Format

User provides:
1. **Problem/topic** (required)
2. **Skill list** (required, minimum 2, maximum 6)
3. **Knowledge base/reference materials** (optional): file paths, URLs, background info
4. **Output preferences** (optional): report language, focus areas

**Example:**
```
multi-mind council: Analyze the balkonkraftwerk market entry opportunity
skills: mao-methodology, marketing-psychology, marketing-ideas
knowledge base: ~/reports/bkw-q1.md
```

If user doesn't specify a skill list, suggest 3-4 most relevant skills based on problem type, wait for confirmation before starting.

---

## PHASE 0: Setup & Validation

1. Extract from user input: problem description, skill list, knowledge base paths
2. Validate each skill name exists in the available skill list
3. skill < 2: Refuse to start ("Need at least 2 skills for cross-validation")
4. skill > 6: Warn about cost, continue after confirmation
5. Create project directory: `{cwd}/council-{YYYYMMDD-HHmmss}/`
6. Write `00-manifest.md` (problem, skills, knowledge base, timestamp)

### No Pre-Digestion of Knowledge Base

If the user provides knowledge base files, **do NOT read and summarize them in the dispatcher**. Instead:
- Pass raw file paths to each agent
- Let each agent decide how to read, extract, and use these materials
- Different skills read from different angles (strategy skill focuses on market structure, psychology skill focuses on consumer behavior) — pre-digestion destroys this diversity

Rationale: Pre-digested knowledge base summaries make all agents start from the same point, undermining independence and discovery diversity. Each skill should see raw materials through its own lens.

---

## PHASE 1: Parallel Skill Execution

> Each agent independently and fully invokes their skill to analyze the problem, with no mutual interference, preventing confirmation bias.

### Execution Method

Use the Agent tool to **launch N agents simultaneously in one message**. Each agent gets a unique `name`: `council-{skill-name}`.

### Agent Prompt Template

The following is the exact agent prompt template. Mandatory requirements (marked with STOP) are hard rules derived from real-world failures and must not be omitted.

```
You are the agent responsible for "{skill-name}" analysis in the multi-mind council.

## Analysis Task
{user's problem description}

## Knowledge Base File Paths (if any)
The following files contain relevant background. You decide which to read and what to extract:
{knowledge base file path list, one absolute path per line}

## STOP — Core Requirement: Must Invoke "{skill-name}" via the Skill Tool

Your job is NOT to "write an analysis using {skill-name}'s thinking framework." Instead:

1. Use the Skill tool to invoke "{skill-name}", passing in the analysis task
2. Let the skill run its own complete workflow (including independent research, data gathering, multi-stage analysis, etc.)
3. If the skill has sub-stages (e.g., mao-methodology has Phase 0-5), ALL must execute

STOP — If you skip the Skill invocation and directly write a report using the skill's thinking style, this will be treated as invalid output.
The dispatcher will check whether your report contains the skill's signature output format (e.g., mao-methodology must have Pre-flight declaration + contradiction analysis + MVA table).

## Independent Research Requirement

STOP — Do not rely solely on existing conclusions from knowledge base files. You must:
- Obtain at least 3 new data points not in the knowledge base via WebSearch
- Supplement with real-time data via MCP tools (depending on availability)
- Clearly mark in the report which data came from the knowledge base and which from your independent research

Rationale: If all agents only digest the same knowledge base, the discussion phase won't produce genuinely new insights. Independent research ensures each agent brings a different data foundation.

## Output Requirements

Write the complete report to: {project directory}/01-{skill-name}-report.md

The report must contain:

### Part 1: Complete Analysis
Output the full analysis following the skill's own methodology. Do not compress or omit.
Must reflect the skill's signature workflow and output format.

### Part 2: Data Provenance Table
| # | Data Point | Source Type | Specific Source |
|---|-----------|-------------|----------------|
| 1 | ...       | Knowledge   | file path      |
| 2 | ...       | WebSearch   | query + URL    |
| 3 | ...       | MCP         | tool + query   |

STOP — At least 3 rows must come from sources outside the knowledge base, or the report is disqualified.

### Part 3: Structured Claims (end of report, mandatory)

Output core findings in YAML format (5-15 claims):

```yaml
claims:
  - id: "{SKILL-PREFIX}-01"
    statement: "One-sentence finding"
    confidence: HIGH | MEDIUM | LOW
    evidence_type: data | logic | pattern | absence
    evidence: "Supporting evidence summary"
    data_source: "Knowledge | WebSearch | MCP | Derived"
    blind_spots: "What this framework cannot see"
```(end yaml)

SKILL-PREFIX uses the skill name's uppercase abbreviation (e.g., mao-methodology -> MAO).
confidence: Has independent research data = HIGH, logical derivation = MEDIUM, pattern/absence inference = LOW.
data_source: Mark where the claim's core evidence came from.
blind_spots must be honestly declared.
```

### Quality Gates

After receiving a report, the dispatcher checks:

1. **Skill invocation verification**: Does the report contain the skill's signature output?
   - mao-methodology -> Must have Pre-flight declaration + contradiction analysis + MVA table
   - marketing-psychology -> Must have cognitive bias analysis + behavioral framework
   - Other skills -> Must have that skill's distinctive analysis structure
   - If missing -> SendMessage requesting the agent to re-execute using the Skill tool

2. **Data independence verification**: Does the provenance table have >= 3 non-knowledge-base sources?
   - If insufficient -> SendMessage requesting the agent to supplement independent research

3. **Claims format verification**: Are there 5-15 structured claims?
   - If missing or malformed -> Request correction

All agents must pass quality gates before entering Phase 2.

### Persistence
Each agent writes to `01-{skill-name}-report.md`. The dispatcher only extracts claims[] into context, does not load full reports.

---

## PHASE 2: Cross-Validation Discussion (3 Rounds)

> The purpose of discussion is not reconciliation, but discovering truth through collision.
> The value of debate is not who's right or wrong, but the new insights that emerge from collision that none of the parties foresaw.

### Round 1: Claim Clustering & Initial Response

**Dispatcher executes (does not launch new agents):**

1. Read all claims[] from the end of each `01-{skill-name}-report.md`
2. Cluster all claims:

| Type | Definition | Handling |
|------|-----------|---------|
| Corroborating | 2+ skills independently reached similar conclusions | Confidence boost |
| Contradictory | A says X, B says not-X | Enter debate |
| Complementary | Different facets, not contradictory | Integrate |
| Isolated | Only 1 skill raised this | Mark for verification |

3. **Anti-false-consensus check**: For conclusions "corroborated" by 3+ skills — strip all jargon, would an uninformed person also say this? If yes -> false consensus, downgrade.

4. **data_source cross-check**: If corroborating claims all point to the same knowledge base file -> this is not independent corroboration, downgrade to "common-source agreement." Only different data_source corroborations count as "multi-perspective independent corroboration."

5. Write to `02-round1-clustering.md`

6. For each contradictory pair, notify involved agents via SendMessage:

```
Your claim {id} ("{position}") contradicts {opposing skill}'s claim {id} ("{opposing position}").

STOP — Adversarial requirement: Your task is to defend your analytical framework's conclusions.
Do not concede easily — unless the other side provides hard data that your framework genuinely
cannot see (not better arguments, but new data).
"The other side makes a good point" is NOT sufficient reason to revise; "the other side produced
data I hadn't seen" IS.

Please:
1. Read {project directory}/01-{opposing skill}-report.md to understand their full argument
2. Self-critique: What is the weakest link in your evidence chain? What are your implicit assumptions?
3. Respond point-by-point to their arguments (with data, not opinions)
4. STOP — Must call WebSearch or MCP tools at least once to obtain new evidence supporting your position
   Mark clearly in response: [New Evidence] {source} {content}
5. Declare stance: maintain / partially revise (explain what changed + why) / withdraw (explain what data refuted you)
6. Write to {project directory}/02-round1-{skill-name}-response.md
```

Skills with no contradictions don't need to participate in Round 1.

**If no contradictions at all:** Skip Rounds 2-3, go directly to Phase 3. Note in report: "All skills converged. Consider adding a contrarian skill (e.g., mao-contrary-evidence)."

---

### Round 2: Evidence Confrontation

**Dispatcher executes:**

1. Read all Round 1 responses
2. Assess each contradiction's status:
   - **Resolved**: One side was refuted by new data and acknowledged
   - **Progressing**: Both sides have new evidence, positions shifting
   - **Deadlocked**: Both sides repeat original positions, no new evidence

3. **Check Round 1 adversarial quality**: If an agent made major concessions without receiving new data -> remind in Round 2 instructions: "You conceded quickly in Round 1. If your original position had reasonable basis, please re-evaluate whether you should maintain it. Concessions require new data the other side's framework couldn't see, not merely better arguments."

4. Write to `03-round2-status.md`

5. For unresolved contradictions, SendMessage to both agents:

```
The opposing Round 1 response is at {project directory}/02-round1-{opposing skill}-response.md.

Please:
1. Read their response, especially the data tagged with [New Evidence]
2. STOP — Must call WebSearch or MCP tools at least once for new evidence
   Focus searches on the opponent's weakness areas — the vulnerabilities they admitted in self-critique
3. Declare stance change: maintain / revise / withdraw
4. If maintaining, you must answer: What specific data (not arguments) would change your mind?
5. Write to {project directory}/03-round2-{skill-name}-response.md
```

**Convergence detection:** If Round 2 responses are substantively identical to Round 1 (no new evidence, no stance changes) -> skip Round 3, mark as "unresolvable (stable positions)."

---

### Round 3: Final Positions

**Dispatcher executes:**

1. Read Round 2 responses
2. For still-unresolved contradictions, SendMessage to relevant agents:

```
Final round. State your final position on {contradiction topic}:
1. Final stance: maintain / revise (explain what changed)
2. Conditions for change: What specific data or experimental results would change your mind?
3. Validation proposal: What concrete test can the user run to settle this? (Specifically: what to test, how to test, how long for results, what outcome supports you / refutes you)
Write to {project directory}/04-round3-{skill-name}-final.md
```

3. Write `04-round3-resolution-map.md` (resolved + unresolved list)

### Core Rules
- **No voting.** No "both sides have a point." Contradictions are either data-arbitrated or acknowledged as unresolvable.
- **Forced end after 3 rounds.** Unresolved disagreements preserve both sides' full arguments for user judgment.
- Newly obtained data during debate must be checked for impact on other claims.
- **Concessions require data justification.** "The other side argued more persuasively" is insufficient; must be "the other side provided data X that my framework genuinely didn't cover."

---

## PHASE 3: Synthesis Report

> Synthesis is not compilation — the dispatcher must produce **emergent insights** beyond what the three skills found:
> "The three skills saw A, B, and C respectively, but combined they point to D — a conclusion no single skill could independently discover."

The dispatcher reads all files in the project directory, assembles the final report, writes to `05-final-report.md`.

### Emergent Insight Mining Before Template (STOP — Must Execute)

Before applying the template, the dispatcher performs the following analysis (this is the key to making the synthesis report exceed "the sum of three reports"):

1. **Cross-blind-spot scan**: Are Agent A's blind_spots covered by Agent B's claims? What blind spots remain uncovered by ALL agents?

2. **Causal chain reassembly**: Different skills may have found different links in the same causal chain. Attempt to connect them:
   - Example: Psychology finds "compatibility anxiety is #1 decision killer" + strategy finds "content gap in nachrüsten" + tactics finds "calculator is highest-ROI tool" -> Emergent insight: "compatibility anxiety x content gap x tool absence = triple vacuum; filling any one creates value, but filling all three simultaneously produces a multiplier effect"

3. **Second-order effects of contradiction resolution**: Does the resolution of a debate itself create new problems?
   - Example: D1 resolved as "authorized reseller" -> but this introduces MOQ capital threshold risk — a second-order problem created by the first-order resolution

4. **Confidence-weighted action ranking**: Rank all recommended actions by "confidence x impact," not by discussion order

### Report Template

```markdown
# Multi-Mind Council Synthesis Report

**Problem**: {problem}
**Skills**: {list}
**Date**: {date}
**Project Directory**: {path}

---

## Executive Summary
{3-5 sentences: main finding, core tension, recommended direction. Not a summary of three reports, but an integrated new understanding.}

---

## 0. Emergent Insights (New Knowledge from Multi-Perspective Collision)

> The following findings belong to no single skill — they emerged from cross-analysis.

| # | Emergent Insight | Source Claim Combinations | Confidence | Action Implication |
|---|-----------------|-------------------------|------------|-------------------|

---

## 1. Consensus Findings (Multi-Skill Independent Corroboration)

> The following were independently reached by 2+ skills from **different data sources**. Marked whether they passed anti-false-consensus check.

| # | Finding | Corroboration Sources | Data Independence | Confidence |
|---|---------|----------------------|-------------------|------------|

---

## 2. Per-Skill Unique Findings

### {skill-1}
- **Unique Insight**: {What only this skill found, invisible to other frameworks}
- **Core Claims**: {top 3}
- **Blind Spots**: {What it knows it cannot see}
- **Data Contribution**: {New data points from this skill's independent research}

### {skill-2}
...

---

## 3. Resolved Debates
| # | Topic | Original Disagreement | Conclusion | Pivotal Data | Round | Second-Order Effects |
|---|-------|----------------------|------------|-------------|-------|---------------------|

---

## 4. Unresolved Disagreements (User Judgment Required)

### Disagreement 1: {topic}
**Side A** ({skill}): {final position + key evidence}
**Side B** ({skill}): {final position + key evidence}
**Validation Proposal**: {what to test, how, timeline, what result determines who's right}
**Impact**: {what it means if Side A is right / Side B is right}

---

## 5. Complementary Findings
{Different facets seen by different skills, assembled into a complete picture. Focus on causal chain reassembly.}

---

## 6. Systemic Blind Spots
| # | Blind Spot | Why All Skills Missed It | What's Needed to Cover It | Priority |
|---|-----------|-------------------------|--------------------------|----------|

---

## 7. Recommended Next Steps (Ranked by Confidence x Impact)
| Priority | Action | Basis | Confidence | Impact | Timeframe |
|----------|--------|-------|------------|--------|-----------|

---

## 8. Debate Process Statistics
| Metric | Value |
|--------|-------|
| Total claims | |
| Corroborating | |
| Contradictory | |
| Complementary | |
| Isolated | |
| Debate rounds | |
| Resolved contradictions | |
| Unresolved contradictions | |
| Stance revisions | |
| New evidence introduced | |

---

## Appendix: Full Report Index
| File | Description |
|------|------------|
```

### Quality Self-Check

Before writing the report, check:
- **Anti-decoration**: Do conclusions still hold after stripping all jargon?
- **Anti-fence-sitting**: Was any dispute glossed over with "both sides have a point"?
- **Anti-omission**: Was every blind_spots checked? Were isolated findings forgotten?
- **Actionability**: Is every recommendation specific enough to execute (who, what, with what tool, how long)?
- **Emergence**: Does Section 0 have genuinely new insights beyond the sum of three reports? If not, redo cross-blind-spot scan and causal chain reassembly.

### Final Output

In the conversation, only output:
```
Report generated: {project directory}/05-final-report.md
(Contains N independent skill analyses + M cross-validation rounds + synthesis + K emergent insights)
[Executive Summary 3-5 sentences]
```

---

## Edge Cases

| Situation | Handling |
|-----------|---------|
| skill < 2 | Refuse, explain need at least 2 |
| skill > 6 | Warn about cost, continue after confirmation |
| No contradictions at all | Skip discussion, suggest adding contrarian skill |
| Circular contradiction (A<->B<->C<->A) | Three-way debate, respond to both opponents |
| Agent fails mid-discussion | Preserve last output, mark round of exit |
| Skill doesn't fit the input | Warn and exclude, or ask user for additional info |
| User didn't specify skills | Suggest 3-4 based on problem type, wait for confirmation |
| Agent didn't invoke Skill tool | Quality gate blocks, require re-execution |
| Agent has no independent research data | Quality gate blocks, require WebSearch/MCP supplement |
| Agent concedes without justification in debate | Round 2 reminder to re-evaluate, require data-driven stance changes |

---

## Context Management

Dispatcher context only retains: manifest + claims[] summaries + phase status.
Full reports are read from files, never loaded into context.
Every sub-phase output is immediately written to file upon completion.
