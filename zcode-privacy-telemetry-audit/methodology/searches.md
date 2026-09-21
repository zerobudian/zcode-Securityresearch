# Methodology — Search Terms

Actual keyword sets used against `packages/` (glob `*.ts`) at commit `872ad960d…`. A reader can rerun these to confirm negatives/positives.

## Network primitives (egress candidates)
```
fetch  undici  axios  got  http.request  https.request  WebSocket  EventSource
XMLHttpRequest  navigator.sendBeacon  electron/net  FormData  multipart
Sentry  PostHog  OpenTelemetry  analytics  telemetry  RUM  arms
OAuth  OSS  S3  object storage  gRPC  RPC  mcp
```

## Historical repo-snapshot leads (expect NEGATIVE)
```
repoSnapshotIndexingEnabled  repoSnapshotIndexingUserConfigured
lastAcceptedManifestHash  uploadCredentialHandle  captureStage
snapshot/upload-credential  repo-snapshot  repo-snapshot.tar.gz.enc
AES-256-CTR  RSA-OAEP  pendingUpload  uploadCredential
```
Result: **0 matches** in audited `packages/`.

## Positive-hit surfaces (current behavior)
```
repoSnapshot|repositorySnapshot|workspaceSnapshot   -> none
uploadOss|uploadOssForm|uploadCredential  -> feedback only (feedbackHttpClient.ts)
x-oss-signature|x-oss-security-token      -> feedback upload flow only
ZCODE_TELEMETRY_ENABLED                   -> = true (env.ts)
ZCODE_TELEMETRY_REPORT_ENDPOINT           -> env-loaded (env.ts)
ZCODE_ARMS_RUM_ENDPOINT                   -> env-loaded (env.ts)
checkpoint|gitCheckpointStore|refs/zcode/checkpoints -> LOCAL git checkpoint
resolveRuntimeZCodeEndpointOrigin|buildRuntimeZCodeApiUrl  -> zcodeEndpoint.ts defaults
DEFAULT_ZCODE_ENDPOINT_ORIGIN             -> https://zcode.z.ai
LOG_RETENTION_DAYS                        -> 14
telemetry-state.json                      -> device identity file
```

## Data/redaction
```
redact|sanitize|mask|scrub  -> telemetryRedaction.ts, redactArmsEventBatch, update-feed URL redact
sqlite|sessions|tasks-index|conversation -> local stores
```

## Where this audit may have gaps (for the reviewer)
- UI settings bindings (desktop settings pages) for toggles/env were only sampled.
- Server-side (retention/training) code is out of scope.
- Built binaries could differ from published source.
Users re-running these searches should also glob outside `*.ts` (JSON/config/registers, CI, e2e fixtures) and inspect `apps/` tree.