# Call Chain: Telemetry Event Report

Audit: ZCode privacy / telemetry
Commit: `872ad960de7ec172591f7e1952f7849229f94521`

## 1. Renderer custom event → shared supervisor → reportEvent

Entry: UI custom telemetry event construction
↓ *EVID-TELEMETRY-006__conversationTelemetrySupervisor.ts* (packages/ui/src/v4/telemetry/)
↓ e.g. line 763, 1236, 1809, 1900, 1950
↓ builds `{}` with `elementName`, `eventRegion`, `eventType`, `eventExtraDetail`
↓
`input` → IPC to host/main telemetry client

## 2. reportEvent → sendReport

`createTelemetryCore().reportEvent(...)`
↓ *EVID-TELEMETRY-001__telemetryCore.ts*:496-536
↓
`sendReport(input, context, eventId, userId, deviceMid)`
↓ telemetryCore.ts:429-460
↓ bounded loop `attempt = 1..REPORT_MAX_ATTEMPTS (2)`; retryable on network/408/429/5xx with 300ms/1000ms backoff
↓
`sendReportAttempt(endpoint, requestBody, headers, userId)`

## 3. Payload shape (telemetryCore.ts:383-406)

Field                    | Source
-------------------------|----------------------------------------
event_id                 | session_create → deterministic SHA-256(userId,talkId); else randomUUID
client_timezone          | runtime context
client_language          | runtime context
element_name             | e.g. send_btn / agent_step / message_completion
event_region             | "app"
event_type               | e.g. "ck" / "agent_trace"
event_text               | optional free text (payload-supplied)
event_extra_detail       | `sanitizeTelemetryEventDetail(...)` — redacted copy (EVID-REDACTION-001)
user_id                  | account-linked if logged in
screen_resolution        | context
app_version              | ZCODE_VERSION
device_os_category       | macos / windows / linux
device_os_version        | os.release()
device_mid               | persistent device UUID (EVID-TELEMETRY-003)
mac_id                   | always ""
marketing_params         | JSON, OAuth login attribution (optional)
talk_id / message_id     | conversation/message links (when available)

## 4. Endpoint selection (telemetryCore.ts:408-413)

`endpoint = rewriteZCodeEndpointUrl(ZCODE_TELEMETRY_REPORT_ENDPOINT, resolvedOrigin)`
- `ZCODE_TELEMETRY_REPORT_ENDPOINT` read at runtime from `process.env`, NOT embedded in build (EVID-TELEMETRY-004, env.ts:53-54).
- IPv6 `mac_id` field is hardcoded empty (`mac_id: ""`, telemetryCore.ts:402).

## 5. Headers / identity

`buildZCodeSourceHeadersFromContext(...)` — app version, platform, arch, OS version, release channel, language, timezone, deviceMid.

## 6. Persistence / device identity

Entry: `reportEvent` → `ensureTelemetryDeviceMid(deviceMidOptions)`
↓ *EVID-TELEMETRY-003__deviceMid.ts*
↓ state file `~/.zcode/v2/telemetry-state.json` (or `{home}/.zcode/v2/telemetry-state.json` for CLI/server)
↓ lock: `telemetry-state.lock`
↓ deviceMid persisted once, reused across app runs → device-linkable.

## 7. Ingestion / correlation

Local JSONL log path: `~/.zcode/v2/logs/YYYY-MM-DD.log` (desktop main).
ARMS RUM separate channel — see `evidence/call-chains/arms-rum.md` (in report body).

## Note on reachability

`ZCODE_TELEMETRY_ENABLED` is a hardcoded `true` (env.ts:50). Actual egress additionally requires the runtime `ZCODE_TELEMETRY_REPORT_ENDPOINT` env to be non-empty; distribution/launcher determines whether it is set. Not verifiable offline from source alone → flagged in report.