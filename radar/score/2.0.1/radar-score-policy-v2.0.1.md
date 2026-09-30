# Horizon Radar V2 — Score Policy 2.0.1

**Status: FROZEN**

This independent policy supersedes Score Policy 2.0 for the modifier resolver while retaining Score Policy 2.0 as an immutable historical baseline. The base score, direct bonus, signal, action, filters, and gates remain governed by their existing frozen rules.

## Frozen resolver contract

- Inputs: radar, title, entity, and relevanceLevel. No Security Gate or score dimensions enter modifier resolution.
- Normalize title/entity with Unicode NFKC and en-US lowercase; all policy terms are evaluated deterministically without an LLM.
- Ordinary triggers use ordered literal substring matching across title and entity. The first matching rule for that Radar applies.
- Security terms `ai`, `mcp`, `agent`, and `gecko` use whole-token matching. ASCII letters and digits are token characters; punctuation and whitespace delimit tokens. Thus `AI-assisted` matches, while `AIO`, `paid`, and `maintain` do not match `ai`.
- Security composition is default bias -2 plus at most one topical +3 trigger, then clamping to [-10, 10]. The modifier is independent of Security Gate, direct bonus, relevance, impact, and actionability.
- An App negative trigger is suppressed to zero when relevanceLevel is UNRELATED. The rule is generic and has no fixture/entity/title exceptions.
- Output: radarModifier, matchedRules[], modifierComponents[], and modifierReason. modifierReason is debug-only and has no scoring effect.

## Expected distribution and delta

| Total | High | Medium | Low | Filtered | Net |
|---:|---:|---:|---:|---:|---:|
| 50 | 8 | 8 | 13 | 21 | 16 |

F34 is the only score delta: radarModifier 1→-2 and finalScore 18→15. Its filtered/ignore outcome and Security Gate reason stay unchanged. F13/F15/F17 resolve to zero through the generic App unrelated rule.

## Canonical policy data

The JSON block below is the canonical machine-readable policy embedded verbatim for cross-artifact consistency.

```json
{
  "scoreVersion": "2.0.1",
  "policyVersion": "2.0.1",
  "status": "FROZEN",
  "supersedes": "2.0",
  "baselineSource": "Score Policy 2.0 frozen artifacts plus Phase 5A.6.2 Option C reconciliation; historical candidate replay remains unchanged.",
  "sourcePolicyJsonSha256": "52f32078c63b07ddeb7f3b0d76c9e2a1e1cab6e5cfc67b08133f11153a7ad0e5",
  "sourceReplaySha256": "cd383741a8532b83c1ea761aba0e98f97edf20c5c31d7c7f077e22bad25f8c9b",
  "modifierResolver": {
    "radarModifierBounds": {
      "min": -10,
      "max": 10
    },
    "triggerOrder": "First matching rule by declared rule order; one trigger rule per Radar, except Security default bias plus at most one topical trigger.",
    "matching": {
      "fields": "NFKC-normalized title and entity, concatenated with a newline.",
      "case": "Unicode NFKC then en-US lowercase.",
      "ordinaryTerms": "Literal substring matching; phrase and token boundaries are not inferred.",
      "securityTerms": "Whole-token matching for every Security term; token characters are ASCII letters and digits, and every other character is a delimiter.",
      "aiAssisted": "The hyphen is a delimiter, so AI-assisted contains the whole token ai and matches.",
      "noLlm": true
    },
    "rules": {
      "ai": [
        {
          "id": "AI_PRICE_OR_COST",
          "terms": [
            "免费",
            "free",
            "降价",
            "price"
          ],
          "value": 5
        },
        {
          "id": "AI_CAPABILITY",
          "terms": [
            "agent",
            "voice",
            "capability"
          ],
          "value": 3
        }
      ],
      "dev": [
        {
          "id": "DEV_CONTEXT_TOOL",
          "terms": [
            "claude code",
            "jev",
            "mcp",
            "compaction"
          ],
          "value": 5
        },
        {
          "id": "DEV_WORKFLOW_TOOL",
          "terms": [
            "agent",
            "harness",
            "review",
            "abide"
          ],
          "value": 3
        }
      ],
      "skill": [
        {
          "id": "SKILL_TRAINING_RESEARCH",
          "terms": [
            "skillopt"
          ],
          "value": 3
        }
      ],
      "app": [
        {
          "id": "APP_STACK_MATCH",
          "terms": [
            "atuin",
            "ghostty",
            "tmux",
            "shell"
          ],
          "value": 3
        },
        {
          "id": "APP_NEGATIVE_RELEVANT_ONLY",
          "terms": [
            "vpn",
            "minio",
            "radicle"
          ],
          "value": -3
        }
      ],
      "sec": [
        {
          "id": "SECURITY_TOPICAL",
          "terms": [
            "ai",
            "mcp",
            "agent",
            "gecko"
          ],
          "value": 3
        }
      ]
    },
    "appUnrelatedRule": {
      "appliesWhen": "radar=app AND relevanceLevel=UNRELATED AND a negative App trigger matches",
      "effect": "Replace that negative trigger component with 0; preserve the match in debug provenance."
    },
    "securityComposition": {
      "defaultBias": -2,
      "topicalTrigger": "+3 for the first whole-token match among ai, mcp, agent, gecko.",
      "formula": "default bias + one topical trigger (if any), then clamp to [-10, 10].",
      "independentOf": [
        "Security Gate",
        "directBonus",
        "relevance",
        "impact",
        "actionability"
      ]
    },
    "output": {
      "required": [
        "radarModifier",
        "matchedRules",
        "modifierComponents",
        "modifierReason"
      ],
      "modifierReason": "Debug only; must not be consumed by scoring decisions.",
      "clamp": "Apply once to the sum of modifier components; clamp inclusively to [-10, 10]."
    },
    "evaluationOrder": [
      "Validate Radar and read explicit title/entity/relevanceLevel inputs.",
      "Normalize title/entity using NFKC and en-US lowercase.",
      "Match ordered Radar-specific rules, stopping after the first matching rule.",
      "For Security, add the default bias and then the first whole-token topical trigger, if any.",
      "For App negative trigger with UNRELATED relevance, emit a zero component.",
      "Sum components, clamp modifier, and produce provenance/debug output.",
      "Pass radarModifier independently to Score Policy; do not feed provenance to scoring."
    ]
  },
  "expectedDistribution": {
    "total": 50,
    "high": 8,
    "medium": 8,
    "low": 13,
    "filtered": 21,
    "net": 16
  },
  "expectedDeltaFrom2_0": {
    "fixtureId": 34,
    "fields": [
      "radarModifier",
      "finalScore"
    ],
    "modifier": {
      "from": 1,
      "to": -2
    },
    "finalScore": {
      "from": 18,
      "to": 15
    },
    "signal": "filtered (unchanged)",
    "action": "ignore (unchanged)",
    "filtered": true,
    "filterReason": "Security Gate UNRELATED (unchanged)"
  }
}
```
