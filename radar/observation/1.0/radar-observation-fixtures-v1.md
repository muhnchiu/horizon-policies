# Horizon Radar V2 — Observation Fixture 1.0

**Status: FROZEN** · **Policy: Observation Identity Policy 1.0**

This independent synthetic fixture validates Observation identity, URL canonicalization, required inputs, and time semantics. Event Replay 1.0 remains a separate dataset and is unchanged. All sample URLs use reserved `example.com`, `example.org`, or `example.net` domains.

## Policy alignment

The authoritative algorithm is SHA-256 over UTF-8 `eventKey + "\n" + canonicalSourceUrl + "\n" + radar`; the ID is the first 16 lowercase hex characters. Identity fields are `eventKey`, `canonicalSourceUrl`, and `radar`. `observedAt`, `retrievedAt`, publication time, and source descriptors are metadata and do not affect identity. See [radar-observation-policy-v1.md](radar-observation-policy-v1.md).

URL canonicalization 1.0 lowercases scheme/hostname, removes fragments and default ports, removes trailing slash on non-root paths, preserves root `/`, and preserves path/query. No redirects, tracking removal, query sorting, semantic rewriting, or fuzzy matching are used.

## Contents

- **19 valid** rows, each explicitly supplying `eventKey`, `radar`, `sourceName`, `sourceUrl`, `sourceLevel`, and `observedAt`.
- **10 invalid** rows with explicit validation errors, covering missing required fields, invalid timestamp, invalid URL, unsupported scheme, and malformed event key.
- **9 identity assertions** with frozen expected IDs: identical tuple/refetch, fragment, host case, default port, trailing slash, query preservation, distinct source URL, cross-Radar, and metadata-only changes (`sourcePublishedAt`, `sourceName`, `sourceLevel`, `sourceAuthority`).

`observedAt` is never inferred from report date, `sourcePublishedAt`, or `retrievedAt`. For the refetch pair, expected `firstSeen` is `2026-08-01T09:00:00Z` and `lastSeen` is `2026-08-03T10:00:00Z`.
