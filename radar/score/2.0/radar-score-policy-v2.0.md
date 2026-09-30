# Radar Score Policy V2.0 Specification

## Policy Version: 2.0
## Score Version: 2.0
## Status: frozen
## Baseline Source: phase-4.2.1-deterministic
## Supersedes: phase-4.1-manual-replay

## 1. Score Formula

### 1.1 Dimension Weights

| Dimension | Weight | Input Range |
|---|---|---|
| Relevance | 0.30 | 0-10 (integer, derived from Relevance Level) |
| Impact | 0.25 | 0-10 (integer) |
| Actionability | 0.15 | 0-10 (integer) |
| Confidence | 0.15 | 0-10 (integer) |
| Novelty | 0.10 | 0-10 (integer) |
| Momentum | 0.05 | 0-10 (integer) |

### 1.2 Relevance Score Mapping

Relevance is not a free-form score. It is determined by the Relevance Level:

| Level | Score |
|---|---|
| DIRECT | 9 |
| ADJACENT | 5 |
| EXPLORATORY | 3 |
| UNRELATED | 1 |

### 1.3 Base Score

```
weightedSum = relevanceScore * 0.30 + impact * 0.25 + actionability * 0.15 + confidence * 0.15 + novelty * 0.10 + momentum * 0.05
baseScore = int(weightedSum * 10)
```

`relevanceScore` is the numeric value mapped from the Relevance Level (Section 1.2): DIRECT=9, ADJACENT=5, EXPLORATORY=3, UNRELATED=1.

The `* 10` scaling normalizes the weighted average from the 0-10 input range to a 0-100 base score scale.

`int()` truncates toward zero. Due to IEEE 754 floating-point arithmetic, `weightedSum * 10` may produce values like 67.99999999999999 instead of 68.0. `int()` correctly truncates these to 67. This is the intended behavior and is verified by the 50/50 replay fixtures.

### 1.4 Direct Bonus

```
directBonus = 5 if (relevanceLevel == "DIRECT" AND (impact >= 7 OR actionability >= 7)) else 0
```

Conditions:
- Relevance level must be DIRECT (score 9)
- AND at least one of: Impact >= 7 OR Actionability >= 7
- If both Impact and Actionability are below 7 (e.g. Impact=6, Actionability=5), directBonus = 0 even if relevance is DIRECT
- Bonus is capped at +5. Both conditions being met does not increase it beyond 5.

### 1.5 Radar Modifier

The radarModifier is an input field provided by the radar agent. The scoring engine applies it as-is, clamped to [-10, 10]. The trigger rules below describe how the radar agent produces the modifier value from the highlight title and entity name.

| Radar | Trigger Pattern | Modifier |
|---|---|---|
| AI | title contains: 免费, free, 降价, price | +5 |
| AI | title contains: agent, voice, capability | +3 |
| Dev | title contains: claude code, jev, mcp, compaction | +5 |
| Dev | title contains: agent, harness, review, abide | +3 |
| Skill | title/entity contains: skillopt | +3 |
| App | title contains: atuin, ghostty, tmux, shell | +3 |
| App | title contains: vpn, minio, radicle | -3 |
| Security | (default bias) | -2 |
| Security | title contains: ai, mcp, agent, gecko | +3 |

Only one trigger per radar applies (first match wins). Total modifier is clamped to [-10, 10].

### 1.6 Final Score

```
finalScore = max(0, min(100, baseScore + directBonus + radarModifier))
```

### 1.7 Evaluation Order

1. Compute relevance level from ACTIVE_STACK / INTEREST_STACK / radar patterns
2. Map relevance level to relevance score (DIRECT=9, ADJACENT=5, EXPLORATORY=3, UNRELATED=1)
3. Score remaining 5 dimensions (impact, actionability, confidence, novelty, momentum) using rubrics
4. Compute baseScore = int(weightedSum * 10) where weightedSum uses relevanceScore from step 2
5. Compute directBonus (0 or +5)
6. Compute radarModifier from trigger rules, clamped [-10, 10]
7. finalScore = clamp(baseScore + directBonus + radarModifier, 0, 100)
8. Apply hard filters (Section 6) -- if filtered, stop here
9. Determine signal: high (>=75), medium (>=50), low (>=25), else filtered
10. Apply security cap: if radar==sec AND securityGate==ECOSYSTEM_RELEVANT AND signal==high AND no escalation keys -> cap to medium
11. Determine action from Action Decision Matrix (Section 5)

## 2. Signal Thresholds

| Signal | Threshold |
|---|---|
| High | >= 75 |
| Medium | >= 50 |
| Low | >= 25 |
| Filtered | < 25 or hard filter |

## 3. Dimension Rubrics

### Relevance
- 0: No relationship to tracked technology
- 2: Tangentially related to broad field
- 5: Conceptually related but not directly used
- 7: Related to active use areas but not core stack
- 10: Directly impacts active stack component

Note: In practice, relevance is always one of {1, 3, 5, 9} derived from the Relevance Level mapping.

### Impact
- 0: No effect
- 2: Minor informational value
- 5: Moderate change to non-core area
- 7: Significant workflow/cost/capability change
- 10: Generational/breaking change

### Actionability
- 0: Nothing can be done
- 2: Theoretical interest only
- 5: Could investigate if time permits
- 7: Clear action available
- 10: Immediate action required

### Confidence
- 0: Rumor
- 2: Single community source
- 5: Multiple ecosystem sources
- 7: Official source confirmed
- 10: Multiple official sources cross-verified

### Novelty
- 0: Complete repetition
- 2: Minor update (version bump)
- 5: Material change to known item
- 7: New project/tool/model
- 10: Entirely new paradigm

### Momentum
- 0: No activity
- 2: <100 engagement
- 5: Hundreds, active discussion
- 7: Thousands, trending
- 10: Viral adoption (>10k)

## 4. Security Gate

| Gate Level | Score Behavior | Signal Cap |
|---|---|---|
| DIRECTLY_EXPOSED | full scoring, no cap | none |
| DEPENDENCY_RELEVANT | full scoring, no cap | none |
| ECOSYSTEM_RELEVANT | scored | Medium (unless escalation) |
| UNRELATED | filtered | filtered |

### Escalation Keys
- activeExploitation
- supplyChainImpact
- reachableDependency
- officialEmergencyAdvisory

CVSS score does NOT bypass the Security Gate. CVSS only influences the Impact dimension.

## 5. Action Decision Matrix

### Rule 1: Filtered
- Condition: signal == "filtered"
- Action: ignore

### Rule 2: First Discovery + High
- Condition: signal == "high" AND eventType in [model-release, project-discovery, skill-release, research-release, price-change, capability-change]
- Logic: actionability>=7 AND relevance>=8 -> test; impact>=6 -> read; else -> test

### Rule 3: Version Release + High
- Condition: signal == "high" AND eventType == "version-release"
- Action: read

### Rule 4: High + Prior Validation
- Condition: signal == "high" AND priorValidation == true AND actionability >= 7 AND confidence >= 6 AND relevance >= 8
- Action: adopt

### Rule 5: High Fallback
- Condition: signal == "high" (not matched above)
- Logic: actionability>=5 -> test; else -> read

### Rule 6: Medium
- Condition: signal == "medium"
- Logic: actionability>=6 AND relevance>=7 -> test; impact>=5 -> read; else -> watch

### Rule 7: Low
- Condition: signal == "low"
- Logic: actionability>=4 -> read; else -> watch

### ADOPT Guard
priorValidation must be true. First discovery events NEVER result in ADOPT.

## 6. Filter Rules

| ID | Condition | Action |
|---|---|---|
| FUNDING | eventType == "funding" | hard_filter |
| SEC_UNRELATED | radar == "sec" AND securityGate == "UNRELATED" | hard_filter |
| UNRELATED_LOW_IMPACT | relevanceLevel == "UNRELATED" AND impact <= 2 | hard_filter |
| DUPLICATE | duplicate == true | hard_filter |

Note: `duplicate` is an external input field. Phase 4 Engine only consumes this flag. Phase 5 will detect cross-day duplicates.

## 7. Publish False Policy

- publish:false + semantic ERROR -> report ERROR, build does NOT fail
- publish:true + semantic ERROR -> FAIL CLOSED

## 8. Popularity Guardrail

Stars/Downloads/Media heat only influence Momentum (weight 0.05, max 10). Cannot directly produce High signal.

## 9. Replay Verification

| Metric | Value |
|---|---|
| Total | 50 |
| High | 8 |
| Medium | 8 |
| Low | 13 |
| Filtered | 21 |
| Net | 16 (high + medium, publishable signals) |

**Match: MATCH**
**Determinism: PASS**
**Fixtures: 50/50 deterministic**
**Entity-Specific Overrides: 0**

## 10. Frozen High Signals (8)

| Score | Base | Bonus | Modifier | Title | Radar | Action |
|---|---|---|---|---|---|---|
| 90 | 80 | +5 | +5 | fast-jev-compaction | Dev | test |
| 81 | 76 | +5 | +0 | GPT-6 Sol/Luna | AI | test |
| 81 | 76 | +5 | +0 | Opus 5.5 | AI | test |
| 81 | 71 | +5 | +5 | GLM-5.2 Free | AI | test |
| 79 | 71 | +5 | +3 | abide | Dev | test |
| 79 | 69 | +5 | +5 | jev-review MCP | Dev | read |
| 77 | 67 | +5 | +5 | CometixCode | Dev | read |
| 75 | 67 | +5 | +3 | Atuin | App | test |

## 11. Formula Verification Examples

### fast-jev-compaction (score=90)
- R=9(DIRECT) I=8 A=9 N=7 C=6 M=8
- weightedSum = 9*0.30 + 8*0.25 + 9*0.15 + 6*0.15 + 7*0.10 + 8*0.05 = 8.05
- baseScore = int(8.05 * 10) = int(80.5) = 80
- directBonus = 5 (DIRECT + impact>=7)
- radarModifier = +5 (dev: "jev"+"compaction" matches)
- finalScore = 80 + 5 + 5 = 90

### GPT-6 Sol/Luna (score=81)
- R=9(DIRECT) I=7 A=7 N=6 C=8 M=7
- weightedSum = 9*0.30 + 7*0.25 + 7*0.15 + 8*0.15 + 6*0.10 + 7*0.05 = 7.65
- baseScore = int(7.65 * 10) = int(76.5) = 76
- directBonus = 5 (DIRECT + impact>=7)
- radarModifier = 0
- finalScore = 76 + 5 + 0 = 81

### abide (score=79)
- R=9(DIRECT) I=7 A=7 N=6 C=5 M=5
- weightedSum = 9*0.30 + 7*0.25 + 7*0.15 + 5*0.15 + 6*0.10 + 5*0.05 = 7.10
- baseScore = int(7.10 * 10) = int(71.0) = 71
- directBonus = 5 (DIRECT + impact>=7)
- radarModifier = +3 (dev: "abide" matches trigger pattern)
- finalScore = 71 + 5 + 3 = 79

### CometixCode (score=77)
- R=9(DIRECT) I=7 A=5 N=7 C=4 M=6
- weightedSum = 9*0.30 + 7*0.25 + 5*0.15 + 4*0.15 + 7*0.10 + 6*0.05 = 6.80
- baseScore = int(6.80 * 10) = int(67.999...) = 67 (floating-point: 6.80 * 10 = 67.99999999999999)
- directBonus = 5 (DIRECT + impact>=7, even though actionability=5 < 7)
- radarModifier = +5 (dev: "claude code" matches)
- finalScore = 67 + 5 + 5 = 77

### Atuin (score=75)
- R=9(DIRECT) I=6 A=7 N=5 C=5 M=5
- weightedSum = 9*0.30 + 6*0.25 + 7*0.15 + 5*0.15 + 5*0.10 + 5*0.05 = 6.75
- baseScore = int(6.75 * 10) = int(67.5) = 67
- directBonus = 5 (DIRECT + actionability>=7, even though impact=6 < 7)
- radarModifier = +3 (app: "atuin"+"shell" matches)
- finalScore = 67 + 5 + 3 = 75

### ChatGPT Voice Agent (score=72, Medium)
- R=9(DIRECT) I=6 A=5 N=6 C=7 M=6
- weightedSum = 9*0.30 + 6*0.25 + 5*0.15 + 7*0.15 + 6*0.10 + 6*0.05 = 6.90
- baseScore = int(6.90 * 10) = int(69.0) = 69
- directBonus = 0 (DIRECT but impact=6 < 7 AND actionability=5 < 7, so bonus does NOT apply)
- radarModifier = +3 (ai: "agent"+"voice"+"capability" matches)
- finalScore = 69 + 0 + 3 = 72
- signal = medium (< 75)

### Claude Code v2.1.281 (score=72, Medium)
- R=9(DIRECT) I=6 A=5 N=3 C=8 M=7
- weightedSum = 9*0.30 + 6*0.25 + 5*0.15 + 8*0.15 + 3*0.10 + 7*0.05 = 6.80
- baseScore = int(6.80 * 10) = int(67.999...) = 67 (floating-point)
- directBonus = 0 (DIRECT but impact=6 < 7 AND actionability=5 < 7, so bonus does NOT apply)
- radarModifier = +5 (dev: "claude code" matches)
- finalScore = 67 + 0 + 5 = 72
- signal = medium (< 75)

## 12. Atuin Resolution

- Primary event (id=11): score=75, High/TEST
- Duplicate event (id=19): duplicate=true, Filtered
- duplicate is a Replay input. Phase 4 Engine only consumes it. Phase 5 detects duplicates.

## 13. Security Regression

- WordPress CVE: Filtered (Security Gate UNRELATED)
- F5 appliance: Filtered (Security Gate UNRELATED)
- Check Point: Filtered (Security Gate UNRELATED)
- Jackson CVE: Retained (DIRECTLY_EXPOSED, Medium)
- Linux Kernel CISA: Retained (DIRECTLY_EXPOSED, Medium)
