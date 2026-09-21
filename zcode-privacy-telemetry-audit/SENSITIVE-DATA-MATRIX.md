# ZCode — Sensitive Data Matrix

Commit `872ad960de7ec172591f7e1952f7849229f94521` · 2026-09-21
Legend: ✔ = yes / present · ✖ = no / not found · ⚠ = partially / conditionally · ? = UNKNOWN

| Data | Local Read | Model/Inference | Telemetry | Snapshot/Upload | Logs | Share |
|---|---|---|---|---|---|---|
| Prompt | ✔ (required to run) | ✔ (task context) | ✖ (no raw prompt in audited event payload; redacted) | ✖ | ⚠ (call-site dependent) | ✔ (when user shares conversation) |
| Source code | ✔ (on-demand tool reads) | ✔ (task-selected code) | ✖ | ✖ | ⚠ (call-site dependent) | ⚠ (whatever is in shared messages) |
| `.git` | ✔ (repo ops) | ✖ (not bulk-sent) | ✖ | ✖ (no archive pipeline) | ⚠ | ✖ |
| `.env` / secrets | ⚠ (read if a tool reads the file) | ⚠ (only if selected context) | ✖ (redaction normalizes secrets to placeholders) | ✖ | ⚠ (not in telemetry-retry logs) | ⚠ (only if in shared content) |
| Terminal | ✔ | ✔ (command output as tool result) | ✖ | ✖ | ⚠ | ⚠ |
| Credentials (auth tokens) | ✔ (local token store) | ✖ (not in inference) | ✖ (redacted; not logged) | ✖ | ✖ (no evidence) | ✖ |
| Device identifier | ✔ (`~/.zcode/v2/telemetry-state.json`) | — | ✔ (`device_mid`) | — | ⚠ | ✖ |
| Workspace path | ✔ | ✖ (paths redacted in telemetry) | ⚠ (not in audited payload; may appear as local log info) | ✖ | ⚠ | ✖ |
| Git remote URL | ✔ | — | ✖ | ✖ | ⚠ | ✖ |

CSV companion: `endpoint-matrix.csv`; network detail: `network-egress.csv`.