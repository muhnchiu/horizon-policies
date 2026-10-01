# Security Gate Policy 1.0.0

Status: FROZEN.

Owns deterministic SEC applicability, tracked-environment classification, escalation, hard-filter/cap dispositions, provenance validation, and reason codes. It does not own modifier, score, generic action, Event/Observation identity, Registry, publication, or human validation.

## Classification

Use the three evidence predicates directExposure, dependencyRelationship, ecosystemRelationship over a versioned tracked-environment baseline and canonical candidate technology/component identities. Use exact IDs or explicit versioned aliases only. Direct deployment and dependency require structured evidence; ecosystem relevance requires explicit tracked membership. FALSE requires a completed, provenance-bearing query over complete inventory coverage. UNKNOWN and MISSING never become UNRELATED. NOT_APPLICABLE requires a versioned applicability basis and stays distinct from FALSE.

Apply specificity order DIRECTLY_EXPOSED > DEPENDENCY_RELEVANT > ECOSYSTEM_RELEVANT > UNRELATED. A higher-specificity UNKNOWN/MISSING blocks a lower-specificity class. UNRELATED requires complete, versioned scope coverage and no positive relationship. Malformed identity/provenance/value is INVALID_INPUT.

## Gate disposition

- SEC + resolved UNRELATED => HARD_FILTER (SECURITY_UNRELATED_HARD_FILTER), outgoing signal filtered.
- SEC + ECOSYSTEM_RELEVANT + incoming high: ESCALATE preserves high; NO_ESCALATION and UNRESOLVED cap to medium. UNRESOLVED does not hard-filter.
- DIRECTLY_EXPOSED and DEPENDENCY_RELEVANT do not cap or hard-filter.
- Explicit NOT_APPLICABLE preserves signal with no gate filter/cap; missing applicability is UNRESOLVED. Non-SEC radar is gate NOT_APPLICABLE.
- Unresolved classification => RECEIPT_ONLY; do not invoke score.

Escalation aggregation and precedence reference the approved Phase 5A.6.8e.3a artifact unchanged. Numeric security modifier remains governed by Score Policy 2.0.1 and is separate from Security Gate.

## Reason codes

| Code | Exact condition |
|---|---|
| SECURITY_CLASS_DIRECT_EXPOSURE | resolved direct class |
| SECURITY_CLASS_DEPENDENCY_RELEVANT | resolved dependency class |
| SECURITY_CLASS_ECOSYSTEM_RELEVANT | resolved ecosystem class |
| SECURITY_CLASS_UNRELATED_CONFIRMED | resolved negative scope evaluation |
| SECURITY_CLASS_INPUT_UNKNOWN | required evidence is unknown |
| SECURITY_CLASS_INPUT_MISSING | required evaluation absent |
| SECURITY_CLASS_INPUT_NOT_APPLICABLE | predicate explicitly N/A with basis |
| SECURITY_CLASS_INVALID_INPUT | malformed structure/value |
| SECURITY_CLASS_TRACKED_BASELINE_MISSING | baseline/version/hash absent |
| SECURITY_CLASS_PROVENANCE_INVALID | required provenance invalid |
| SECURITY_UNRELATED_HARD_FILTER | SEC and resolved UNRELATED |
| SECURITY_SIGNAL_CAP_APPLIED | ecosystem/high and result is NO_ESCALATION or UNRESOLVED |
| SECURITY_SIGNAL_CAP_EXEMPT_ESCALATION | ecosystem/high and result is ESCALATE |
| SECURITY_GATE_NOT_APPLICABLE | non-SEC or explicit valid N/A applicability |
| SECURITY_GATE_INVALID_INPUT | malformed Security Gate input |

See machine-readable candidate for exact fields and deterministic result shape.
