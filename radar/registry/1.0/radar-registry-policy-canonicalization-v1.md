# Horizon Radar V2 Phase 5A.6.7a — Registry Policy Canonicalization & Baseline Compatibility Audit

**Result: PASS**  
**Canonicalization: FROZEN**  
**Distribution readiness: YES**  
**Next Gate: READY_FOR_PHASE_5A.6.7_POLICY_DISTRIBUTION**

## Source of Truth and Method

Frozen Registry Design (`radar-registry-design-v1.md`) is the normative semantic source. Frozen Registry Schema (`radar-registry-schema-v1.json`) supplies record shape and constraints. The historical Registry Manifest is retained as immutable historical evidence. The implementation and conformance artifacts were used only to verify conformance; no policy semantics were inferred from implementation. The deterministic projection emits each normative rule as `ruleId`, `sourceSection`, and `canonicalValue`.

Canonicalization timestamp: 2026-09-30T12:10:39Z. Canonicalization version: 1.0. Contract pin: `5ac615eda0de0b0fa1d2cc398309fc2657bacc49` (Contract 2.1.2, schemaVersion 2).

## Frozen Integrity and Conformance

- Frozen Registry Design: `f054be5caf1bd79c432e8ef0b4444ccc3e62cbf18f10671907e9e0143366acf5` — PASS
- Frozen Registry Schema: `828b3a378c1f9801cfd3837a5d8a3bbd399df5ce6772949d2d7cb157f5dc091a` — PASS
- Historical Registry Manifest: `54ab0cde71865005e6f60c0d1204a1f5d68be47f9b3fa5785236bd3f3f5b2250` — PASS, unchanged
- Latest design-to-implementation matrix: `46fb3c2cd20a7b5603960cbfc26997b77e239afbb9252f54f4d2c72fcf262d67` — **36/36 CONFORMANT**
- 5A.6.7b Lock Conformance: PASS; global lock, PID/start identity, reuse and fail-safe cases retained
- 5A.6.7c Corruption Quarantine Conformance: PASS
- 5A.6.7d Invalid Row Shape Quarantine Conformance: PASS; malformed row shapes produce structured diagnostics and paired raw-byte quarantine; no repair
- Earlier quarantine conflict: RESOLVED by 5A.6.7d; no remaining matrix deviation

The Design, Schema, historical manifest, Event Policy 1.0 / Replay 1.0, Observation Policy 1.0 / Fixture 1.0, Score Policy 2.0 / 2.0.1, and Contract 2.1.2 package pin match their frozen integrity baselines.

## Canonicalization and Traceability

`radar-registry-policy-v1.json` is now `policyVersion: 1.0`, `status: FROZEN`. Its rule sets cover registry model, Event and Observation records, occurrences, global/process/stale lock policy, transaction and atomic commit, deterministic recovery, corruption and quarantine, validation, paired snapshots, candidate receipts, Event resolution, Observation deduplication, production mutation boundary, and compatibility. Every normative rule carries its Frozen Design section and canonical value.

Policy ambiguity: **NONE**. Policy drift: **NONE**. The policy does not redefine Event Identity, Observation Identity, Contract semantics, or scoring semantics. The registry projection honors only frozen Event fact fields and Observation facts.

## Compatibility Audit

- Registry Policy 1.0 × Score Policy 2.0: **COMPATIBLE**
- Registry Policy 1.0 × Score Policy 2.0.1: **COMPATIBLE**
- Score compatibility scope: 50 candidate synthetic persistence projections in temporary Registry roots; this is not represented as a production dataset join. Identity, event state, materialChange, duplicate, observationId, occurrences, write/transaction/snapshot/receipt behavior are equal. Score outputs were not passed into Registry APIs.
- F34: Score 2.0 modifier `+1`, score `18`; Score 2.0.1 modifier `-2`, score `15`. Only the scoring projection changes. Registry identity, state, Observation identity, occurrences, and mutation projection are identical.
- Registry Policy 1.0 × Event Policy 1.0: **PASS**; 50 candidate replay exact-matches 39 unique Events, 39 NEW / 11 DUPLICATE / 0 UPDATE.
- Registry Policy 1.0 × Observation Policy 1.0: **PASS**; 29/29 fixture cases, 19 valid / 10 invalid, identity assertions 9/9, invalid assertions 10/10; 13 unique Observations and 9 Events remain unchanged on second run.

## Contract and Mutation Boundaries

Contract 2.1.2 public fields remain limited to the versioned Contract shape. Derived Registry aggregates are identified as derived; identity fingerprints/history, observation identity implementation values, retrieval/writer timestamps, lock/journal/transaction/snapshot/quarantine details remain internal; run receipts and match/quarantine diagnostics are debug-only. Registry recovery and transaction metadata are not published. Contract verification and normalizer/validator tests pass at the pinned package.

`commitObservation()` is the verified production-authoritative mutation boundary. `upsertEvent()` and `upsertObservation()` remain legacy/non-production compatibility APIs. This records the existing verified interface boundary and introduces no API semantics.

## Regression Evidence

- Contract verification: PASS; Contract/normalizer/identity suite: **56/56 PASS**
- Event Replay 1.0: **50/50 exact** (39 Event records; 39 NEW / 11 DUPLICATE / 0 UPDATE)
- Observation Fixture 1.0: **29/29** (19 valid / 10 invalid; identity 9/9; invalid 10/10)
- Phase 3 validator: **11/11 PASS**
- Phase 4 historical Score Policy 2.0 replay: **50/50 exact**; Phase 4 tests **7/7 PASS**
- Score Policy 2.0 verifier: PASS; Score Policy 2.0.1 verifier and modifier replay: **50/50 PASS**; modifier tests **4/4 PASS**
- Registry full suite: **96/96 PASS**
- Focused quarantine suite: **44/44 PASS**
- Evidence pipeline: **7/7 PASS**
- `npm run build`: **PASS**, 46 pages
- `git diff --check` and `git diff --cached --check`: **PASS**

Registry tests use temporary directories. Production Radar, Production Registry mutation, Publisher, deployment, migration, commit, and push were not run. Existing workspace changes were preserved, including `src/styles/global.css`.

## Distribution Gate

`DISTRIBUTION_READY = YES`. No policy package was published and no distribution commit was created. The historical manifest and all frozen inputs remain unchanged.
