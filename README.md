# Using Multi-Skills

**Run multiple skills in parallel on the same problem, then let agents debate and cross-validate findings through structured discussion rounds.**

This skill invokes **real skill pipelines** with full tool access (WebSearch, MCP servers, etc.), forces independent research, and produces emergent insights through structured adversarial debate.

**Designed for analysis tasks only** — market research, strategic planning, competitive analysis, opportunity assessment.

---

## Table of Contents

- [How It Works](#how-it-works)
- [Architecture](#architecture)
- [Installation](#installation)
- [Usage](#usage)
- [Project Output Structure](#project-output-structure)
- [Key Mechanisms](#key-mechanisms)
- [Configuration & Requirements](#configuration--requirements)
- [Final Report Structure](#final-report-structure)
- [FAQ](#faq)
- [License](#license)

---

## How It Works

### The 4-Phase Flow

```
User: "Analyze {problem} with skills: {A}, {B}, {C}"
         |
 [Phase 0] Setup
         |  Validate skills, create project directory, write manifest
         |  Knowledge base files passed as raw paths (NOT pre-digested)
         |
 [Phase 1] Parallel Skill Execution
         |  N agents launched simultaneously
         |  Each MUST invoke their skill via the Skill tool (verified by dispatcher)
         |  Each MUST do independent research (>= 3 new data points beyond knowledge base)
         |  Each writes: full report + data provenance table + structured claims
         |
 [Phase 2] Cross-Validation Discussion (up to 3 rounds)
         |  Round 1: Dispatcher clusters claims -> agents respond to contradictions
         |  Round 2: Agents bring NEW evidence (mandatory tool calls) -> stance changes
         |  Round 3: Final positions with concrete validation proposals
         |  (Rounds skipped if convergence detected early)
         |
 [Phase 3] Synthesis
         |  Emergent insight mining (not just compilation)
         |  Cross-blind-spot scan, causal chain reassembly, second-order effects
         |  Final report with confidence x impact action priorities
         |
 Output: Project directory with all artifacts + final report
```

### Phase-by-Phase Detail

#### Phase 0: Setup & Validation

- Parses problem statement, skill list, and optional knowledge base paths
- Validates skill names exist in your system
- Creates a timestamped project directory: `council-{YYYYMMDD-HHmmss}/`
- **Knowledge base files are NOT pre-digested** — each agent receives raw file paths and reads them independently through its own analytical lens. This preserves diversity of interpretation.

#### Phase 1: Parallel Skill Execution

All agents launch simultaneously. Each agent must:

1. **Invoke the assigned skill via the Skill tool** — not just adopt the skill's thinking style. The dispatcher checks for the skill's signature output format. Reports missing signature formats are rejected.

2. **Conduct independent research** — minimum 3 data points from WebSearch or MCP tools that aren't in the knowledge base. A Data Provenance Table is required in every report, showing each data point's source type (knowledge base, WebSearch, or MCP).

3. **Output structured claims** (5-15) at the end of every report in YAML format, each with: id, statement, confidence level, evidence type, data source, and declared blind spots.

Reports that fail quality checks are **rejected and agents must re-execute**.

#### Phase 2: Cross-Validation Discussion

**Round 1 — Claim Clustering & Initial Response**

The dispatcher clusters all claims into 4 types:

| Type | Definition | Handling |
|------|-----------|---------|
| **Corroborating** | 2+ skills independently reached similar conclusions | Confidence boost |
| **Contradictory** | Skill A says X, Skill B says not-X | Debate triggered |
| **Complementary** | Different facets, not conflicting | Integrated |
| **Isolated** | Only 1 skill found this | Marked for verification |

Two additional checks:
- **Anti-false-consensus**: If 3+ skills "agree", strip all jargon — would an uninformed person also say this? If yes, it's platitude wisdom, not genuine corroboration. Downgraded.
- **Data source cross-check**: If "corroborating" claims trace to the same knowledge base file, that's same-source agreement, not independent corroboration. Downgraded.

For each contradiction, agents must:
- Read the opposing report
- Self-critique their weakest evidence link
- Respond point-by-point with data (not opinions)
- **Search for new evidence** (mandatory — at least 1 WebSearch or MCP call per round, tagged as `[New Evidence]`)
- Declare stance: maintain / partially revise / withdraw

Agents are instructed to **defend their framework's conclusions** and not capitulate easily. Stance changes require new data the agent's framework genuinely couldn't see, not just compelling arguments.

**Round 2 — Evidence Confrontation**

- Dispatcher assesses each contradiction: resolved / progressing / deadlocked
- If an agent conceded in Round 1 **without receiving new data**, the dispatcher flags this and asks the agent to re-evaluate
- Agents must again bring at least 1 new evidence source
- **Convergence detection**: If positions are identical to Round 1 (no new evidence, no stance changes), Round 3 is skipped and the contradiction is marked "unresolvable (stable positions)"

**Round 3 — Final Positions**

For still-unresolved contradictions, agents provide:
- Final stance (maintain or revise)
- What specific evidence would change their mind
- A concrete validation test the user can run (what to test, how, timeline, what result supports which side)

**Core rules:**
- No voting. No "both sides have a point." Contradictions are either data-settled or declared unresolvable.
- 3 rounds maximum. Unresolved disagreements preserved with both sides' full arguments.
- Stance changes require new data, not just better arguments.

#### Phase 3: Synthesis

Before assembling the report, the dispatcher performs **emergent insight mining**:

1. **Cross-blind-spot scan** — Does Agent A's blind spot get covered by Agent B's claims? What remains uncovered by ALL agents?
2. **Causal chain reassembly** — Different skills may have found different links in the same causal chain. Connect them to reveal insights invisible to any single skill.
3. **Second-order effects** — Does resolving Contradiction D1 create a new problem?
4. **Confidence x Impact prioritization** — Rank all recommended actions by both certainty and potential impact.

The final report highlights what **no single skill could have found alone**.

---

## Architecture

```
                    +------------------+
                    |   User Request   |
                    | Problem + Skills |
                    +--------+---------+
                             |
                    +--------v---------+
                    |   Phase 0: Setup |
                    |  Validate + Dir  |
                    +--------+---------+
                             |
              +--------------+--------------+
              |              |              |
     +--------v----+  +-----v-------+  +---v----------+
     | Agent A      |  | Agent B     |  | Agent C      |
     | Skill tool   |  | Skill tool  |  | Skill tool   |
     | invocation   |  | invocation  |  | invocation   |
     | + WebSearch  |  | + WebSearch |  | + WebSearch  |
     | + MCP tools  |  | + MCP tools |  | + MCP tools  |
     +--------+-----+  +-----+-------+  +---+----------+
              |              |              |
              +--------------+--------------+
                             |
                    +--------v---------+
                    | Dispatcher:      |
                    | Claim Clustering |
                    | (4-type system)  |
                    +--------+---------+
                             |
              +--------------+--------------+
              |              |              |
     +--------v----+  +-----v-------+  +---v----------+
     | Debate R1-3  |  | Debate R1-3 |  | Debate R1-3  |
     | New evidence |  | New evidence|  | New evidence |
     | required     |  | required    |  | required     |
     +--------+-----+  +-----+-------+  +---+----------+
              |              |              |
              +--------------+--------------+
                             |
                    +--------v---------+
                    | Synthesis:       |
                    | Emergent Insight |
                    | Mining + Report  |
                    +------------------+
```

---

## Installation

### Method 1: Direct Copy (Recommended)

```bash
git clone https://github.com/Ginning-ZDZQ/using-muti-skills.git
mkdir -p ~/.claude/skills/multi-mind-council
cp using-muti-skills/SKILL.md ~/.claude/skills/multi-mind-council/SKILL.md
```

### Method 2: Git Submodule

```bash
cd ~/.claude/skills
git submodule add https://github.com/Ginning-ZDZQ/using-muti-skills.git multi-mind-council
```

### Method 3: Manual Download

1. Download `SKILL.md` from this repository
2. Place it at `~/.claude/skills/multi-mind-council/SKILL.md`
3. Restart Claude Code

### Verify Installation

Type in Claude Code:
```
multi-mind council
```
Claude should recognize the skill and ask you for a problem + skill list.

---

## Usage

### Basic Usage

```
Analyze {your problem}
skills: {skill-A}, {skill-B}, {skill-C}
```

### With Knowledge Base

```
multi-mind council: Analyze {your problem}
skills: {skill-A}, {skill-B}, {skill-C}
Knowledge base: ~/reports/data.md
```

### Auto-Trigger

The skill auto-triggers when you mention 2+ skill names in an analysis context:
```
Use {skill-A} and {skill-B} to analyze {your problem}
```
Claude will confirm before starting:
```
Detected multi-skill analysis request:
  Problem: {your problem}
  Skills: {skill-A}, {skill-B}
  Estimated time: significant (each skill runs full pipeline)
  Confirm?
```

### What NOT to Use It For

This skill is designed for **analysis and research tasks only**:
- Market analysis, competitive research, strategic planning
- Decision evaluation, opportunity assessment
- Cross-validation of analytical findings

**Don't use it for:**
- Writing code (use dispatching-parallel-agents instead)
- Content creation (use individual skills sequentially)
- Tasks where skills have sequential dependencies

---

## Project Output Structure

Each run creates a timestamped directory with all artifacts:

```
council-{YYYYMMDD-HHmmss}/
|
+-- 00-manifest.md                               # Problem, skills, config
|
+-- 01-{skill-A}-report.md                       # Full report from Skill A
+-- 01-{skill-B}-report.md                       # Full report from Skill B
+-- 01-{skill-C}-report.md                       # Full report from Skill C
|
+-- 02-round1-clustering.md                      # Claim clustering results
+-- 02-round1-{skill-A}-response.md              # Skill A debate response
+-- 02-round1-{skill-B}-response.md              # Skill B debate response
|
+-- 03-round2-status.md                          # Round 2 status
+-- 03-round2-{skill-A}-response.md              # Round 2 responses
+-- 03-round2-{skill-B}-response.md
|
+-- 04-round3-resolution-map.md                  # What resolved, what didn't
|
+-- 05-final-report.md                           # Comprehensive synthesis
```

Every artifact is preserved — you can trace how conclusions evolved through debate.

---

## Key Mechanisms

### 1. Forced Skill Invocation

Agents must use the Skill tool to invoke their assigned skill's full pipeline — not just adopt the thinking style. The dispatcher verifies by checking for signature output formats. Reports without proper skill execution are rejected and agents must re-execute.

### 2. Independent Research Requirement

If all agents consume the same pre-digested summary, "cross-validation" is just three agents agreeing on the same input. Each agent receives raw file paths, reads independently, and must contribute 3+ new data points from outside the knowledge base. A Data Provenance Table is required in every report.

### 3. Adversarial Tension Calibration

Agents naturally want to agree. This skill prevents premature capitulation:
- Stance changes require new data, not just compelling arguments
- Rapid concession without `[New Evidence]` tag triggers a dispatcher warning in the next round
- Each debate round mandates at least 1 WebSearch/MCP tool call for new evidence

### 4. Anti-False-Consensus Detection

When 3+ skills "agree", strip all jargon and ask: would an uninformed person say this too? If yes, it's common wisdom, not genuine multi-framework corroboration. Additionally, claims tracing to the same data source are flagged as same-source agreement, not independent corroboration.

### 5. Emergent Insight Mining

The final report must go beyond compilation. The dispatcher performs:
- **Cross-blind-spot scanning**: Does Skill B cover Skill A's declared blind spot?
- **Causal chain reassembly**: Connect fragments found by different skills into a complete causal picture
- **Second-order effect analysis**: Does resolving Debate D1 create a new problem?
- **Confidence x Impact prioritization**: Not just listing findings, but ranking actions

---

## Configuration & Requirements

### Prerequisites

- **Claude Code** with skill support
- **At least 2 analysis-oriented skills** installed

### Recommended Skill Types

Any analysis-oriented skills work well. Good combinations include skills that offer **different analytical lenses** on the same problem — for example, pairing a strategic analysis skill with a behavioral/psychological skill and a tactical/creative skill gives you three distinct perspectives that are most likely to produce meaningful contradictions and emergent insights.

### Optional

- **WebSearch** tool enabled — agents need this for independent research
- **MCP servers** configured for domain-specific data (if available)

### Resource Considerations

| Factor | Estimate |
|--------|----------|
| Agents | 2-6 parallel, each running full skill pipeline |
| Discussion rounds | Up to 3 with mandatory tool calls |
| Token usage | 300K-500K+ for a 3-skill run |
| Time | 10-30 minutes |

For lighter needs, use individual skills directly.

### No Additional Config Files Needed

The skill runs entirely from `SKILL.md`. Project directories are created in your current working directory.

---

## Final Report Structure

| Section | Content |
|---------|---------|
| **Executive Summary** | 3-5 sentences: main finding, core tension, direction |
| **Emergent Insights** | What no single skill could find alone |
| **Consensus Findings** | Multi-skill corroboration with data independence check |
| **Per-Skill Highlights** | Unique discoveries + declared blind spots |
| **Resolved Debates** | Evidence trail + second-order effects |
| **Unresolved Disagreements** | Both sides preserved + validation proposals |
| **Complementary Insights** | Different facets assembled |
| **System Blind Spots** | What ALL skills missed |
| **Recommended Actions** | Ranked by confidence x impact |
| **Debate Statistics** | Claims count, stance changes, new evidence introduced |

---

## FAQ

**Q: Can I use this with any skill?**
A: Best with analysis/research skills. Execution skills (copywriting, code generation) aren't suitable — they produce outputs, not analytical claims.

**Q: What if I only have 2 skills?**
A: 2 is the minimum. 3-4 is the sweet spot for depth vs. cost.

**Q: What if agents skip the Skill tool call?**
A: The dispatcher checks for signature output formats. Missing = rejected, agent must re-execute.

**Q: What if all skills agree?**
A: Anti-false-consensus check runs. If it passes, debate rounds are skipped and the report notes the convergence.

**Q: Can agents actually search the web during debates?**
A: Yes, and they must. Each round requires at least 1 WebSearch or MCP call for new evidence.

---

## Contributing

Issues and PRs welcome:
- **Battle-test reports** — Run the skill and share what worked
- **Skill combination guides** — Which combos work well for which domains
- **Translations** — README and SKILL.md translations welcome

---

## License

MIT — see [LICENSE](LICENSE)
