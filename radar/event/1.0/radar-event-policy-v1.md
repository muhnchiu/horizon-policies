# Radar Event Identity Policy V1.0 (Frozen)

## Phase: 5A.1 — Event Identity Refinement & Freeze
## Status: FROZEN
## Policy Version: 1.0
## Baseline: Phase 4.2.1 deterministic fixtures (50 candidates)

## 1. Semantic Model

### Entity
A continuously tracked object. Examples: GPT-6, Claude Code, ZCode, CVE-2026-94127.

### Event
A specific change that happened to an entity. Examples: release, pricing change, version update, CISA KEV addition.

### Observation
A discovery or report of an Event by a Radar/Source.

### Rules
- Multiple observations may map to one event
- Multiple events may belong to one entity
- Different canonicalEventType for same entity = different event (no cross-type UPDATE)

## 2. Event Key Format

```
<entity>:<canonicalEventType>:<eventIdentifier>
```

Rules:
- All lowercase
- Deterministic (same inputs always produce same key)
- Human debuggable (readable, not a hash)
- Machine comparable (string equality)
- No random UUID
- canonicalEventType is part of identity — different types never share eventKey

Examples:

| Event Key | Meaning |
|---|---|
| `openai-gpt6:model-release:initial` | GPT-6 Sol/Luna initial release |
| `claude-code:version-update:2.1.281` | Claude Code v2.1.281 version bump |
| `gigatech-pdv5701:security-cve:cve-2026-94493` | CVE-2026-94493 for Gigatech PDV5701 |
| `zcode:release:initial` | ZCode project first release |
| `f5-bigip:security-cve:f5-bigip` | F5 BIG-IP CVE (no CVE-ID extractable) |
| `f5-bigip:security-cisa-kev:kev` | F5 BIG-IP CISA KEV addition (separate event) |
| `linux-kernel-cisa:security-cisa-kev:kev` | Linux Kernel CISA KEV addition |

## 3. Canonical Event Type Registry (13 types)

| Type | Description |
|---|---|
| `release` | Generic software/tool release (absorbs project-discovery, product-discovery) |
| `model-release` | AI model publication or weight release |
| `skill-release` | Skill, plugin, or extension published |
| `research-release` | Research paper, study, or project published |
| `version-update` | Version bump of an existing tool or library |
| `pricing-change` | Pricing, availability, or licensing terms change |
| `capability-change` | New feature or capability added to existing product |
| `security-cve` | CVE vulnerability disclosure |
| `security-cisa-kev` | CISA KEV addition |
| `security-advisory` | Non-CVE security advisory or vulnerability disclosure |
| `funding` | Funding, acquisition, or valuation event |
| `incident` | Incident, controversy, or outage |
| `documentation` | Official documentation or guide published |

### Removed Types
- `tool-discovery`: Discovery is observation metadata, not an event type. Merged into `release`.
- `project-discovery`: Same as above.
- `product-discovery`: Same as above.

## 4. Release Taxonomy Precedence

When classifying an event, apply in order (first match wins):

1. `version-update`: Version string extractable AND entity already tracked
2. `model-release`: AI model entity
3. `skill-release`: Skill/plugin/extension entity
4. `research-release`: Research paper/study entity
5. `release`: Everything else (including first discovery of new tools)

Precedence is determined by entity type and available version info, not title wording.

## 5. Event Identifier Precedence

| Priority | Field | Description |
|---|---|---|
| 1 | `cveId` | CVE ID extracted from title (e.g. CVE-2026-94493) |
| 2 | `version` | Version string extracted from title (e.g. 2.1.281) |
| 3 | `entity` | Canonical entity slug from Phase 4 |
| 4 | `canonicalEventType` | Event type from registry |
| 5 | `eventIdentifier` | Type-specific identifier (initial, version, cve-id, etc.) |

Raw title is NEVER used as final identifier.

## 6. Canonicalization

| Type | Rule |
|---|---|
| Entity | Normalize to lowercase kebab-case slug. Variants of same entity map to same slug. |
| Version | Extract numeric version string (major.minor.patch) from title. Ignore 'v' prefix. |
| Repository | Normalize to owner/repo format (e.g. microsoft/azure-skills). |
| CVE | Normalize to CVE-YYYY-NNNN format, lowercase. |
| URL | Strip trailing slash, strip query params (except canonical article IDs). |
| Model | Normalize to entity:modelVariant format. |

Examples:
- ZCode / ZCode - Z.ai / Z.ai ZCode -> `zcode`
- CVE-2026-94493 / cve-2026-94493 -> `cve-2026-94493`

## 7. Event States

### NEW
- eventKey has never been seen in the registry
- Outputs: `duplicate=false, materialChange=false`
- Registry action: Create new event entry

### DUPLICATE
- eventKey exists AND no material change detected
- Outputs: `duplicate=true, materialChange=false`
- Registry action: Update lastSeen, increment occurrences, append source

### UPDATE
- eventKey exists AND material change detected within same canonicalEventType
- Outputs: `duplicate=false, materialChange=true`
- Registry action: Update lastSeen, increment occurrences, update fingerprint, append source
- **Boundary rule**: If canonicalEventType changed, it is a NEW event, not UPDATE. Cross-type progression (e.g. security-cve -> security-cisa-kev) = two separate events.

## 8. Minimal Material Change (Phase 5A scope)

### IS a material change

| Change | Within Event Type |
|---|---|
| New version | version-update |
| Pricing terms materially changed | pricing-change |
| API availability changed | any |
| Exploitation status changed | security-* |
| New official release | release, model-release |
| License changed | any |
| Major capability change | any |

### IS NOT a material change

| Non-Change | Why |
|---|---|
| GitHub Stars increased | Popularity drift |
| Downloads increased | Popularity drift |
| Media coverage increased | Additional reporting |
| Title changed | Cosmetic |
| Description changed | Cosmetic |
| Community heat increased | Engagement drift |
| Same claim from another source | Accumulate into sources[] |

### Cross-Type Rule
canonicalEventType change = NEW event, never UPDATE.

## 9. Event Fingerprint

SHA256 over event facts only. User-relative intelligence metadata excluded.

### Event Fact Fields (included)

| Field | When |
|---|---|
| `version` | version-update |
| `cveId` | security-cve |
| `activeExploitation` | When present and true |
| `supplyChainImpact` | When present and true |
| `reachableDependency` | When present and true |
| `officialEmergencyAdvisory` | When present and true |

### Excluded Fields

| Field | Why Excluded |
|---|---|
| `securityGate` | User-relative assessment, not event fact |
| `title` | Text content |
| `description` | Text content |
| `score` | Intelligence metadata |
| `signal` | Intelligence metadata |
| `action` | Intelligence metadata |
| `relevanceLevel` | User-relative |
| `impact` | User-relative |
| `actionability` | User-relative |
| `confidence` | User-relative |
| `novelty` | User-relative |
| `momentum` | User-relative |

## 10. Observation Model

An Observation is a discovery or report of an Event by a Radar/Source.

| Field | Description |
|---|---|
| `observationId` | Deterministic: `radar:entity:eventKey:date` |
| `observedAt` | ISO date |
| `radar` | ai\|dev\|app\|sec\|skill |
| `source` | Source name |
| `sourceUrl` | Canonical URL |
| `sourceLevel` | official\|ecosystem\|community\|media\|research |
| `eventKey` | The event this observation maps to |

One Event may have multiple Observations. Observation metadata (title, description) does not affect Event identity.

## 11. Registry Design

### Format: JSONL
### Location: `~/.local/state/horizon/radar-event-registry.jsonl`

### Event Facts (fingerprint-affecting)

```json
{
  "eventKey": "string",
  "entity": "string",
  "canonicalEventType": "string",
  "firstSeen": "YYYY-MM-DD",
  "lastSeen": "YYYY-MM-DD",
  "occurrences": "integer",
  "latestFingerprint": "sha256hex16",
  "fingerprintHistory": [{"fingerprint": "...", "date": "YYYY-MM-DD"}],
  "sources": [{"sourceName": "...", "sourceUrl": "...", "sourceLevel": "...", "firstSeen": "..."}]
}
```

### Intelligence Metadata (NOT fingerprint-affecting)

```json
{
  "securityGate": "string",
  "score": "integer",
  "signal": "string",
  "action": "string",
  "radars": ["ai", "dev"]
}
```

Intelligence metadata changes do NOT create Event UPDATE.

## 12. Score Policy Integration

Phase 4 Frozen Rule: `duplicate=true` -> Hard Filter

| Event State | duplicate | materialChange | Phase 4 Behavior |
|---|---|---|---|
| NEW | false | false | Normal scoring |
| DUPLICATE | true | false | Hard Filter (frozen rule) |
| UPDATE | false | true | Normal scoring (materialChange consumed by Phase 5B) |

Phase 4 Score Policy 2.0 is NOT modified.

## 13. Over-Merge Guard

Same entity does NOT imply same event. Different canonicalEventType = different event.

| Scenario | Same Entity? | Same Event? | Why |
|---|---|---|---|
| GPT-6 release vs GPT-6 pricing change | Yes | NO | Different eventType |
| Claude Code version-update vs Claude Code incident | Yes | NO | Different eventType |
| F5 BIG-IP security-cve vs F5 BIG-IP security-cisa-kev | Yes | NO | Different eventType (resolved) |
| Atuin release day 1 vs Atuin release re-reported day 2 | Yes | YES (DUPLICATE) | Same eventKey |
| ZCode on AI radar vs ZCode on Dev radar | Yes | YES (DUPLICATE) | Same eventKey, cross-radar |

## 14. Ambiguous Fallback

Principle: Prefer false split over false merge. Two events temporarily separate is lower risk than swallowing a real new event.

When identity is ambiguous, default to DUPLICATE only if eventKey matches exactly. If there is any doubt about entity or identifier mapping, treat as NEW.

### Logged Ambiguous Cases

| Entity | Fixtures | Issue | Resolution |
|---|---|---|---|
| https-capture | 12, 16 | "limited free" vs "lifetime free" — same promotion or new pricing? | DUPLICATE (conservative) |
| obra/superpowers | 42, 47 | Same skill or different artifact in suite? | DUPLICATE (conservative) |

Both require Phase 5B for full resolution.

## 15. Historical Simulation Results

### Metrics

| Metric | Value |
|---|---|
| Candidates | 50 |
| Unique Events | 39 |
| NEW | 39 |
| DUPLICATE | 11 |
| UPDATE | 0 |

### Key Case Verifications

| Entity | Fixtures | Result | Match |
|---|---|---|---|
| Atuin | 11, 19 | NEW + DUPLICATE | YES |
| ZCode | 4, 9, 23, 26 | NEW + 3x DUPLICATE | YES |
| DeepSeek-V4.1 | 3, 7 | NEW + DUPLICATE | YES |
| CometixCode | 24, 28 | NEW + DUPLICATE | YES |
| azure-skills | 41, 48 | NEW + DUPLICATE | YES |
| obra/superpowers | 42, 47 | NEW + DUPLICATE (ambiguous) | YES |
| SkillOpt | 44, 50 | NEW + DUPLICATE | YES |
| F5 CVE/KEV | 33, 38 | 2x NEW (separate events) | YES |
| HTTPS Capture | 12, 16 | NEW + DUPLICATE (ambiguous) | YES |

### Collision Audit

| Check | Result |
|---|---|
| False Merge | 0 |
| False Split | 0 |
| Ambiguous | 2 (logged, conservative fallback) |

## 16. Contract Extension Proposal

Phase 5A requires contract extension for:

| Field | Type | Source |
|---|---|---|
| `eventKey` | string | Phase 5A Event Identity |
| `eventState` | enum (NEW\|DUPLICATE\|UPDATE) | Phase 5A Event Identity |
| `materialChange` | boolean | Phase 5A/5B |
| `canonicalEventType` | string | Phase 5A Event Type Registry |

`duplicate` already exists in Contract V2.

Formal Contract is NOT modified. Extension proposal only.

## 17. Radar Boundary

| Phase | Scope |
|---|---|
| 5A (this document) | Event Identity, Same-event Dedup, Minimal UPDATE boundary |
| 5B | Full Material Change Engine, semantic diff, importance delta, event lifecycle |
| 5C | Primary Radar Ownership, Cross-Radar Collision Resolution, relatedRadars |

## 18. Production Protection

- Horizon: NOT MODIFIED
- Score Policy: UNCHANGED (frozen 2.0)
- Publisher: NOT RUN
- Production Radar: NOT RUN
- Git: NOT RUN
- Deploy: NOT RUN
