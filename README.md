# Horizon Radar Frozen Policies

Immutable, versioned behavioral policy artifacts for Horizon Radar V2. Contract schemas remain owned and distributed by `horizon-contracts`; this repository does not copy Contract schemas.

Runtime consumers must pin a full Git commit SHA (for example through a Git submodule gitlink). Do not consume branch heads, tags alone, or `latest`.

See `distribution-manifest.json`, `radar/policy-package-map-v1.json`, and `radar/baselines/runtime-v1/radar-runtime-baseline-v1.json` for artifact hashes and the supported runtime baseline. Score Policy 2.0 is retained as a historical verification baseline; Score Policy 2.0.1 is active in runtime baseline 1.
