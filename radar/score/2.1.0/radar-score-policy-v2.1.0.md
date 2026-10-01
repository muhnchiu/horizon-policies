# Horizon Radar Score Policy 2.1.0

Status: FROZEN. Local canonical baseline; not distributed.

Score arithmetic, relevance map, Direct Bonus, signal thresholds and 2.0.1 modifier behavior remain unchanged. Score Policy 2.1.0 adds a versioned Action Decision Policy with P07 READ fallback, an action-only HYBRID risk policy, entity-level human priorValidation lifecycle projection, and a field-level provenance-carrying input contract.

First-discovery TEST requires relevanceScore >= 8 and actionability >= 7; first-discovery status alone never produces TEST. Risk is not added to numeric scoring. priorValidation=true only comes from an ACTIVE explicit human validation record.

Duplicate and EventState are consumed only from committed Registry results. Score Policy does not own Event/Observation identity or Registry state. Required unresolved inputs make an ineligible item RECEIPT_ONLY; no partial scoring or V1 reverse mapping is permitted.

Historical Score Policy 2.0/2.0.1 artifacts and the 50-row historical replay remain byte-identical. Contract 2.1.2, Event Policy 1.0, Observation Policy 1.0 and Registry Policy 1.0 are pinned dependencies.
