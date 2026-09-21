# ZCode — Logging Behavior

Commit `872ad960de7ec172591f7e1952f7849229f94521` · 2026-09-21

## Locations & retention
- **Desktop main-process logs:** `~/.zcode/v2/logs/YYYY-MM-DD.log` (`logger.ts` → `getAppConfigDir()/logs`; `E2E` override respected). See EVID-LOGGING-001.
- **Retention:** `LOG_RETENTION_DAYS = 14`; startup cleanup deletes older `YYYY-MM-DD.log` files (`logRetention.ts:4,6,35-96`). See EVID-LOGGING-002.
- **Levels:** `debug | info | warn | error`; main-writer logs `source` (main/renderer/etc.), `[level]`, timestamp (logger.ts:73-110).
- **CLI/Stdio agent logs:** `~/.zcode/cli/log/zcode-*.jsonl` (`zcodeTaskServiceAdapter.ts:246-247`); older paths may exist under `.zcode`.

## What can be logged (the real question for privacy)
Logging is **call-site driven**: whether a given line contains prompt / source / tool args depends on *what the call site passes*. Findings:
- **Telemetry retry logging explicitly logs only sanitized attempt metadata** and forbids payload/response/raw error: `telemetryLogger.debug(... 'retry event=... eventId=... attempt=...')` — telemetryCore.ts:443-446. This is an explicit, documented privacy choice.
- **No evidence that the logging core writes Authorization headers, cookies, access tokens, or raw secrets.** Updater URLs are redacted for logs via `redactUpdateFeedUrlForLog` (autoUpdater.ts:659).
- **ARMS/console-and-crash:** console errors are collected by ARMS (consoleError collector) but lifecycle-labeled events are filtered and the outbound batch is redacted; local logs retain original values by design (redaction is on the egress copy only).

## Conclusion
Local logs may contain call-site context (including file paths and, at some sites, tool/terminal output). The dedicated telemetry path deliberately excludes sensitive payloads. **Claims that the logger dumps prompt/source/tokens by default were not substantiated by source**; remaining unknown call-sites are marked NOT VERIFIABLE. Logs are local, 14-day rotated, and not auto-uploaded (only the user-initiated feedback flow can carry attachments; log export exists as a user command `DesktopCommandIds.ExportLogs`).

Evidence: EVID-LOGGING-001, EVID-LOGGING-002, EVID-TELEMETRY-001.