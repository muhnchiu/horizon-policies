# Score Policy 2.1.1

Status: **FROZEN**. This is the canonical frozen policy package. Normative base: frozen Score Policy 2.1.0.

## Approved delta

- relevanceLevel == "UNRELATED" AND impact <= 2 => hard filter
- Security Gate dependency: Security Gate Policy 1.0.0, exact version pin. Score consumes versioned gate output and does not recalculate Security Gate classification, filter, or cap.
- Security modifier ownership: SCORE_POLICY; modifier behavior and arithmetic remain inherited from the frozen base.
- Eligibility: unresolved or invalid Security Gate results cannot become SCORE_READY.
- Publication: UNDEFINED; ADOPT authorization: false.

All other policy fields inherit unchanged from Score Policy 2.1.0. The following embedded canonical JSON is generated from the same object as the JSON payload and is the complete normative representation.

## Complete normative JSON

```json
{
  "artifact": "radar-score-policy-v2.1.1",
  "scoreVersion": "2.1.1",
  "policyVersion": "2.1.1",
  "status": "FROZEN",
  "normative": true,
  "supersedes": "2.1.0",
  "baselineReferences": {
    "scorePolicy20": "2.0",
    "scorePolicy201": "2.0.1",
    "scorePolicy210": "2.1.0"
  },
  "scoreCalculation": {
    "weights": {
      "relevance": 0.3,
      "impact": 0.25,
      "actionability": 0.15,
      "confidence": 0.15,
      "novelty": 0.1,
      "momentum": 0.05
    },
    "relevanceMap": {
      "DIRECT": 9,
      "ADJACENT": 5,
      "EXPLORATORY": 3,
      "UNRELATED": 1
    },
    "formula": "baseScore=floor(weighted dimensions * 10); finalScore=clamp(baseScore + directBonus + radarModifier, 0, 100).",
    "numericRiskInput": false
  },
  "directBonus": {
    "condition": "relevance==DIRECT AND (impact>=7 OR actionability>=7)",
    "value": 5,
    "cap": 5
  },
  "signalThresholds": {
    "high": 75,
    "medium": 50,
    "low": 25
  },
  "radarModifier": {
    "version": "2.0.1",
    "policy": "Frozen deterministic Score Policy 2.0.1 resolver; bounds [-10,10].",
    "packageRef": "radar-modifier-resolver-v2.0.1 dependency",
    "owner": "SCORE_POLICY"
  },
  "securityGate": {
    "ownership": "Security Gate Policy",
    "scoreBehavior": "Use versioned gate evaluation output (applicability, resolution/class, hardFilter, effective signal/cap disposition, escalation result, baseline ref); Score does not recalculate class/filter/cap. Security hard-filter, signal-cap and escalation outputs remain outside weighted score/modifier; deterministic safety facts cannot be weakened by model judgment. Security modifier remains separately owned by SCORE_POLICY and resolved by radarModifier.",
    "dependency": {
      "policy": "Security Gate Policy",
      "version": "1.0.0",
      "exactPin": true,
      "candidateArtifact": "vendor/horizon-policies/radar/security-gate/1.0.0/radar-security-gate-policy-v1.json",
      "candidateSha256": "1050157be9eb2d5a7a9af2af3bb5777073ff255584a125889049f1da04487701"
    }
  },
  "filterPolicy": {
    "funding": "hard filter",
    "securityUnrelated": "hard filter under Security Gate",
    "unrelatedLowImpact": "relevanceLevel == \"UNRELATED\" AND impact <= 2 => hard filter",
    "duplicate": "hard filter only from committed Registry result"
  },
  "actionPolicy": {
    "path": "radar-action-decision-policy-v2.1.0.json",
    "actionVocabulary": [
      "ignore",
      "watch",
      "read",
      "test",
      "adopt"
    ]
  },
  "riskPolicy": {
    "path": "radar-risk-policy-v1.0.json",
    "generation": "HYBRID",
    "numericScoreMapping": "NONE"
  },
  "priorValidationPolicy": {
    "path": "radar-prior-validation-lifecycle-policy-v1.0.json",
    "scope": "ENTITY_LEVEL",
    "states": [
      "ACTIVE",
      "STALE",
      "REVOKED"
    ],
    "authority": "EXPLICIT_HUMAN_VALIDATION"
  },
  "inputGenerationPolicy": {
    "path": "radar-score-input-generation-policy-v2.1.0.json",
    "eligibility": "SCORE_READY or RECEIPT_ONLY; no partial scoring"
  },
  "compatibility": {
    "contract": "2.1.2",
    "eventPolicy": "1.0",
    "observationPolicy": "1.0",
    "registryPolicy": "1.0",
    "historicalReplay": "50/50 all fields; historical behavior delta NO"
  },
  "evaluationOrder": [
    "validate all score inputs and per-field provenance",
    "derive relevance score from relevance level",
    "compute weighted base score using frozen weights",
    "derive direct bonus",
    "resolve frozen radar modifier",
    "clamp final score",
    "apply hard filters from frozen rules and committed Registry result",
    "classify signal and apply Security Gate cap",
    "resolve action by ordered Action Decision Policy"
  ],
  "semanticChangeSummary": [
    "P07 first-discovery fallback resolves to READ; TEST requires explicit relevance and actionability gate.",
    "Risk is an action/safety input only, not a numeric score term.",
    "priorValidation is a human-authoritative entity lifecycle projection."
  ],
  "distributionStatus": "NOT_DISTRIBUTED",
  "freezeState": "FROZEN",
  "eligibilityPolicy": {
    "unresolvedSecurityGate": "RECEIPT_ONLY; required securityGate enum cannot be validly supplied and frozen input policy prohibits partial/default scoring; not SCORE_READY.",
    "invalidSecurityGate": "RECEIPT_ONLY with INVALID_VALUE diagnostics; not SCORE_READY."
  },
  "publicationBoundary": {
    "semantics": "UNDEFINED",
    "status": "OUT_OF_SCOPE",
    "policyRequired": true,
    "wouldPublish": "UNDEFINED",
    "adoptAuthorization": false
  }
}
```
