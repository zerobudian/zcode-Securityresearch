# Call Chain: ARMS RUM (performance / error monitoring)

Audit: ZCode privacy / telemetry
Commit: `872ad960de7ec172591f7e1952f7849229f94521`

Entry: Desktop app `app.whenReady()` before creating BrowserWindow (see index.ts).
↓ *EVID-TELEMETRY-002__appARMSBootstrap.ts* (packages/desktop/src/main/)
↓ `startArmsRum()` → `armsRum.init({ enable: true, endpoint: ZCODE_ARMS_RUM_ENDPOINT, ... })` (lines 168-264)
↓ collectors: jsError, consoleError, crash, application, api, rpc; browserCollectors: ARMS_BROWSER_COLLECTORS
↓ tracing: enable, sample = prod ? 0.1 : 1
↓ sessionConfig.sampleRate = 1
↓ `beforeReport`: filter/enrich native crash, `ingestArmsApiEventsFromBatch`, `enrichLongTaskAttribution`, **`redactArmsEventBatch(events)`** before egress
↓
HTTP POST to `ZCODE_ARMS_RUM_ENDPOINT` (SLS / version-controlled endpoint).

## Gating / reachability (appARMSBootstrap.ts:266-268)

```
export const armsInitPromise =
  ZCODE_TELEMETRY_ENABLED && ZCODE_ARMS_RUM_ENDPOINT ? startArmsRum() : Promise.resolve();
```
- `ZCODE_TELEMETRY_ENABLED` = `true` (hardcoded, env.ts:50).
- `ZCODE_ARMS_RUM_ENDPOINT` read at **runtime from env** (env.ts:57-58), not embedded.
- ⇒ If the runtime environment does not set `ZCODE_ARMS_RUM_ENDPOINT`, the ARMS SDK is never initialized and no RUM traffic egresses. Device identity via `ensureDesktopDeviceMidSync()` → `~/.zcode/v2/telemetry-state.json`.

## Notes

- `collectors.consoleError = true` means console errors may be captured; a dedicated lifecycle marker filter drops packaged exceptions (`ZCODE_AGENT_LIFECYCLE_LOG_MARKER`).
- Redaction (`redactArmsEventBatch`) is applied last, on the copy that leaves the machine; local logs / crash archives keep original values.
- Outcome: this is classified **product telemetry / diagnostics**, not a code-content upload.