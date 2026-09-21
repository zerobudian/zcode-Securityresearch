# ZCode — Network Egress Inventory

Audit date: 2026-09-21
Repository: https://github.com/zai-org/ZCode.git
Commit: `872ad960de7ec172591f7e1952f7849229f94521` ("feat: open source", 2026-09-21 05:14:32 +0800)
Branch: `main` · Package version: 3.14.0 · No git tags

This inventory lists every network-capable path identified in the audited commit. Rows where an endpoint is resolved from a **runtime environment variable** mean the built artifact does **not** embed a hardcoded URL; actual egress depends on the launcher/distribution setting that variable. Such rows are flagged `ENV-RESOLVED`.

The type column uses the taxonomy from the main report: **A** Model Inference, **B** Product Telemetry, **C** Snapshot/Checkpoint, **D** Explicit Upload, **E** Auth/OAuth, **F** Content/Distribution.

---

## Confirmed egress paths

| # | Source file | Symbol / Function | Caller | Trigger | Method | Endpoint | Host | Payload | Headers | Auth | Auto | User-init | Configurable | Disable | Sensitive potential | Evidence | Type |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | packages/services/src/telemetry/telemetryCore.ts | `createTelemetryCore.reportEvent` → `sendReport` → `sendReportAttempt` | UI/agent telemetry supervisors (EVID-TELEMETRY-006) | app launch, daily active, session/create, send, agent_step, message_completion, context_compaction, tool lifecycle | POST | `ZCODE_TELEMETRY_REPORT_ENDPOINT` (env; rewritten via `rewriteZcodeEndpointUrl`) | ENV-RESOLVED | events: event_id, timezone, language, element_name, event_region, event_type, event_text, event_extra_detail (sanitized), user_id, screen_resolution, app_version, os category/version, device_mid, mac_id="", marketing_params, talk_id, message_id | `buildZCodeSourceHeadersFromContext` (app/version/arch/OS/releaseChannel/lang/tz/deviceMid) | account/user_id when logged in | Yes | No | Endpoint via env | `ZCODE_TELEMETRY_REPORT_ENDPOINT` absent → no report (then `ZCODE_TELEMETRY_ENABLED=true` persists) | event free-text redacted via EVID-REDACTION-001; no raw prompt/code/source paths in payload | EVID-TELEMETRY-001 | B |
| 2 | packages/desktop/src/main/appARMSBootstrap.ts | `startArmsRum()` → `armsRum.init` | Desktop `whenReady` | app start | POST (batch) | `ZCODE_ARMS_RUM_ENDPOINT` (env) | ENV-RESOLVED | RUM/PV/perf/webvitals, jsError, consoleError, crash, api, rpc, longTask summaries; redacted before egress | SDK | device via `user.name=deviceMid` | Yes | No | Endpoint via env | unset env → SDK not initialized (appARMSBootstrap.ts:266-268) | errors stripped of paths/emails/secrets by `redactArmsEventBatch`; no source code content | EVID-TELEMETRY-002 | B |
| 3 | packages/services/src/feedback/feedbackHttpClient.ts | `FeedbackHttpClient.uploadFile` | feedback ticket flow | user opens feedback & attaches file | POST | `/feedback/attachment/upload-credential` (baseUrl from options) | ZAI service (baseUrl) | {ticket_id, message_id?, file_name, size} | JSON | yes (session) | No | Yes | baseUrl configured at runtime | not invoking this API = nothing sent | file metadata; no OSS secret embedded | EVID-ATTACHMENT-002 | D |
| 4 | packages/services/src/feedback/feedbackHttpClient.ts | `uploadOssForm` | call #3 | after credential issued | POST | OSS host from credential (`credential.oss.host`) | server-provided OSS bucket | multipart form-data: file + `key/policy/x-oss-signature/x-oss-signature-version/x-oss-credential/x-oss-security-token/x-oss-date/callback/success_action_status` | multipart | OSS ephemeral signature in credential | No | Yes | No (server-issued credential) | abort supported | attachment bytes the user chose to attach to feedback | EVID-ATTACHMENT-002 | D |
| 5 | packages/services/src/providers/api/nodeApiClient.ts / node.ts:2399 | `NodeApiClient` (host API client) | model provider / resolver | agent task inference & API | per API | `buildRuntimeZCodeApiUrl(process.env, "/api/v1")` | ENV-RESOLVED (`ZCODE_BASE_URL`) | API-specific (conversation, tools, auth) | buildZCodeSourceHeaders + auth | yes (account) | Yes for inference | No (model-driven) | baseUrl via env | model off / account off | model request includes conversation/tool context (LEGITIMATE inference — see Q25) | EVID-TELEMETRY-007 | A |
| 6 | packages/provider-node/src/zcode-builtin-download.ts | `downloadBuiltin` | provider config sync | provider config load | GET | `/api/v1/client/configs?origin=...` | ENV-RESOLVED | none concerned | auth | yes | Yes | No | yes | — | none (remote config) | — | A/config |
| 7 | packages/services/src/session/offPeakServerClient.ts | OffPeak client | v4 session | off-peak / background model task | per API | `${origin}/api/v1/off-peak${path}` | ENV-RESOLVED | session/task payloads | auth | yes | Yes | No | yes | — | session data (legitimate model path) | — | A |
| 8 | packages/services/src/oauth/providers/zaiProviderAdapter.ts & apps zaiBusinessTokenResolver.ts | OAuth provider adapter | `/api/oauth/authorize`, `/api/v1/oauth/token`; login URL | user login | GET/POST | `https://chat.z.ai` OR `VITE_ZCODE_BASE_URL` | chat.z.ai / configured | OAuth code/token exchange | OAuth | yes | No | Yes | yes | — | auth tokens | EVID-TELEMETRY-004 (web env) + webZaiOAuthConfig.ts | E |
| 9 | packages/services/src/providers/zaiBusinessTokenResolver.ts | ZaiBusinessTokenResolver | business-named auth | token resolution | POST | configured login URL | ENV-RESOLVED | credentials | — | refresh token | Yes | No | yes | — | auth | — | E |
| 10 | packages/web/src/share/conversationSharePreviewClient.ts | preview client | share preview | opening a share link | GET | `{baseUrl}/shares/{shareCode}/preview` | configured | none | JSON | none/public | Yes | No (auto on load) | — | — | returns preview of a shared conversation | — | F |
| 11 | packages/services/src/node.ts:1638,2378 | Server MCP usage + provider baseURL | Server MCP | MCP runtime | GET | `/api/v1/mcp/usage` | ENV-RESOLVED | quota | identity headers | yes | Yes | No | — | — | auth/identity | — | A/remote |
| 12 | packages/zcode-server-cli/src/server-core/http.ts | local server | serving zcode server | local HTTP server | GET/POST | localhost routes `/api/server-info`, `/api/rpc-host-capability` | localhost | local | — | — | — | — | — | local only, no external host | — | F (local) |
| 13 | packages/desktop/src/main/desktopRuntimeEnv.ts:232 | remote asset CDN | asset loading | runtime | GET | `ZCODE_REMOTE_ASSET_CDN_BASE_URL` | ENV-RESOLVED | static assets | — | — | Yes | No | — | — | none (assets) | — | F |

> Note on `#12`: a *locally-bound* HTTP server is not an egress path; it is listed for completeness only and is not an outbound network channel.

---

## Confidence

- Rows 1-5 were verified against actual source content read during this audit; payload field lists are direct from source.
- Rows whose endpoint is `ENV-RESOLVED` could not be dynamically exercised offline; actual host is set by the distribution launcher. See Limitations in the main report.
- No path building a **workspace/repo archive + OSS upload** (the historical "repo snapshot" feature) was found; the only object-storage upload is feedback attachments (row 4). See `evidence/call-chains/snapshot-checkpoint.md`.

See also CSV: `network-egress.csv`.