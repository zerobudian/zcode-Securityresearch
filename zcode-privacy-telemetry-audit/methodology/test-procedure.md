# Methodology — Test Procedure (dynamic verification)

**Status: NOT EXECUTED** in this sandbox. Dynamic traffic capture and canary-workspace testing were designed but not run because the environment did not provide a runnable ZCode build (Electron + native deps) or an authorized integration account. This file documents the reproduction procedure any reviewer can follow.

## 1. Synthetic canary workspace (no real secrets)
Create a throwaway repo with fake markers (all values fictitious):
```
CANARY_NORMAL_FILE_<random>
CANARY_ENV_SECRET_<random>
CANARY_GIT_HISTORY_SECRET_<random>
CANARY_IGNORED_FILE_<random>
CANARY_UNTRACKED_FILE_<random>
CANARY_MCP_SECRET_<random>
```
Check whether each appears in scanner / manifest / checkpoint / archive / logs / network payload.

## 2. Network capture (redact credentials)
Run ZCode under `mitmproxy`/`ZEDMO` with a fixed CA; observe host/method/path/body-schema/body-size/timing. Redact `Authorization`, `Cookie`, `AccessToken`, `SecurityToken`, `Signature`, API keys before storing results in `evidence/runtime/`.

## 3. Assertions to test
- No POST body contains the canary workspace path or `.git` objects.
- Telemetry body, if it egresses, does not contain canary source content.
- Setting any telemetry toggle (if present) removes telemetry egress.
- Attaching a local file does not trigger upload until Send.

## 4. Where to record
`evidence/runtime/` (sanitized request metadata + results) and `evidence/screenshots/` (UI state only, no personal info). None produced yet → sections marked accordingly.