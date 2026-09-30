# Horizon Radar V2 — Observation Identity Policy 1.0

**Status: FROZEN**

## Authority and scope

This policy is authoritative for Observation identity, canonical source URLs, Observation deduplication, and Observation time semantics. It formally supersedes only the `observationId` definition in Frozen Event Policy 1.0. Event Policy 1.0 remains authoritative for `eventKey`, `canonicalEventType`, fingerprints, `NEW` / `DUPLICATE` / `UPDATE`, and material change. Event Replay 1.0 remains unchanged.

An Observation is a Radar evidence relationship between one Event and one canonical Source. It is not a fetch execution, crawler run, or daily report occurrence.

## Identity

Identity fields are `eventKey`, `canonicalSourceUrl`, and `radar`. Encode exactly as UTF-8 bytes of:

```text
eventKey + "\n" + canonicalSourceUrl + "\n" + radar
```

`observationId` is the first 16 lowercase hexadecimal characters of SHA-256 over those bytes. The identity excludes `entity`, `date`, `observedAt`, `retrievedAt`, `sourcePublishedAt`, `sourceName`, `sourceLevel`, `sourceAuthority`, and `eventOccurredAt`.

## URL canonicalization 1.0

- Lowercase scheme and hostname. Accept HTTP and HTTPS.
- Remove default ports (`http:80`, `https:443`).
- Remove fragments.
- Remove trailing slash from non-root paths; preserve root `/`.
- Preserve path and query as provided. Do not sort/filter query parameters.
- Do not resolve redirects over a network, strip tracking parameters, rewrite semantic URLs, or fuzzy-match.

## Metadata and time

`sourceName`, `sourceLevel`, `sourceAuthority`, `observedAt`, `sourcePublishedAt`, and `retrievedAt` are metadata, not identity fields. `observedAt` is required and explicitly supplied as an ISO 8601 date-time for when Radar first records the Observation or the timestamp on its current record. `sourcePublishedAt` is optional and records source publication time. `retrievedAt` is local-state-only fetch/read time. `eventOccurredAt` belongs to the Event. No time field enters the ID.

Re-fetching the same Event/source/Radar with changed `observedAt` or `retrievedAt` retains the same ID. A registry may update `lastObservedAt`, `retrievedAt`, and metadata without creating another identity. Multiple observation timestamps can derive `firstSeen = min(observedAt)` and `lastSeen = max(observedAt)`.

## Deduplication

- Same Event, canonical URL, and Radar: same Observation.
- Different canonical URL for one Event: different Observation.
- Different Radar for one Event and URL: separate Observations; Event remains the same.
- Changes to source name, level, authority, or timestamps do not change the ID.

## Contract compatibility

Contract 2.1.1 is unchanged and carries `eventKey`, Radar, `observedAt`, and source URL/name/level/publication/authority metadata. `observationId`, `canonicalSourceUrl`, and `retrievedAt` remain internal fields. Contract 2.1.1 defines `sourceLevel` as `official`, `research`, `ecosystem`, `media`, or `community`; this policy does not redesign that enum. `sourceAuthority` remains optional.

## Fixture baseline

Observation Fixture 1.0 is frozen with **29 cases: 19 valid and 10 invalid**. Expected canonical URLs and IDs are stored in `radar-observation-fixtures-v1.json`; the fixture manifest stores its hashes and this policy's artifact hashes.
