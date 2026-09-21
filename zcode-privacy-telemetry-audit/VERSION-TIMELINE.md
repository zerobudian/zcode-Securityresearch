# ZCode — Version / Historical Issue Timeline

Commit `872ad960de7ec172591f7e1952f7849229f94521` · 2026-09-21

> IMPORTANT: The public repo history is **squashed** (2 commits: `Initial commit`, `feat: open source`; no tags). Granular per-feature commit history is therefore **not available** in this repository to date *when* a feature changed. This timeline reconstructs *current source state* versus *publicly-reported historical behavior*. Removal dates are **NOT VERIFIABLE from repo history**.

| Date/epoch | Public report / topic | Current-source behavior | Commit available for diff | Status |
|---|---|---|---|---|
| (reported) | Cloud workspace/repo snapshot with encrypted tarball + OSS upload | No such code path in `packages/` | `872ad96` only (squashed) | NOT FOUND in current commit; removal date NOT VERIFIABLE |
| (reported) | `repoSnapshotIndexingEnabled` / `repoSnapshotIndexingUserConfigured` settings | Keyword **absent** everywhere | squashed | NOT FOUND |
| (reported) | AES-256-CTR + RSA-OAEP encrypted snapshot archive | **Absent** | squashed | NOT FOUND |
| (reported) | `snapshot/upload-credential`, `lastAcceptedManifestHash`, pending resume | **Absent** | squashed | NOT FOUND |
| (reported) | `~/.zcode/v2/checkpoints/` as cloud-checkpoint | Directory is used by **local Git checkpoint** store (`gitCheckpointStore.ts`) | current | CHANGED (local-only) |
| current | Product telemetry (event report + ARMS RUM), device_mid persisted | Present, env-gated | current | CONFIRMED (new behavior) |
| current | Feedback attachments → OSS via server credential | Present, user-initiated | current | CONFIRMED |
| current | `/api/v1/event/report` public path | Only referenced in a comment; actual endpoint env-resolved | current | CHANGED/UNKNOWN |
| current | Local session/task storage (`sessions/`, `tasks-index.sqlite`) | Present on-device | current | CONFIRMED |

**Interpretation (code-fact only, no motive):** the higher-severity historical *cloud repo snapshot* capability is **not present** in the audited open-source commit. Because the source was open-sourced as a single squashed commit, an independent reviewer **cannot** reconstruct the internal commit-by-commit removal timeline from this repo alone — that part is **NOT VERIFIABLE**. The current telemetry architecture (device/account-correlated, env-gated, sanitized) is verifiable from source.