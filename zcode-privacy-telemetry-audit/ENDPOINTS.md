# ZCode — Endpoint Matrix

Commit `872ad960de7ec172591f7e1952f7849229f94521` · 2026-09-21

Default origins (zcodeEndpoint.ts): ZCode API `https://zcode.z.ai` · BigModel `https://bigmodel.cn` · ZAI OAuth `https://chat.z.ai` · ZAI business `https://api.z.ai`.

Paths marked `ENV` are resolved from a runtime env variable; the default origin/appended &`/api/v1` path is shown.

| Endpoint | Method | Trigger | Data | Auth | Automatic | Disable | Evidence |
|---|---|---|---|---|---|---|---|
| `ZCODE_TELEMETRY_REPORT_ENDPOINT` (env) | POST | launch/daily-active/session/agent events | telemetry event (sanitized) + ids | account/user_id | Yes | clear env | EVID-TELEMETRY-001 |
| `ZCODE_ARMS_RUM_ENDPOINT` (env) | POST | app start | RUM/perf/errors/devicemid | none | Yes | clear env | EVID-TELEMETRY-002 |
| `https://zcode.z.ai/api/v1` (or ZCODE_BASE_URL) + API path | REST | inference & product APIs | conversation/tool context etc. | account | Yes | model off / self-host baseURL | EVID-TELEMETRY-007 |
| `{origin}/api/v1/client/configs` | GET | provider config load | none | yes | Yes | — | EVID-TELEMETRY-007 |
| `{origin}/api/v1/off-peak*` | per API | background/off-peak model task | session/task payload | account | Yes | — | — |
| `{origin}/api/v1/mcp/usage` | GET | MCP runtime start | quota | identity headers | Yes | — | — |
| `{origin}/feedback/attachment/upload-credential` | POST | user attaches file to feedback | ticket/message/file meta | session | No (user) | skip | EVID-ATTACHMENT-002 |
| `credential.oss.host` (server-issued) | POST | after credential | multipart file + x-oss-* fields | OSS signature | No (user) | abort | EVID-ATTACHMENT-002 |
| `https://chat.z.ai/api/oauth/authorize`, `{origin}/api/v1/oauth/token` | GET/POST | user login | OAuth code/token | OAuth | No (user) | logout | EVID-TELEMETRY-004 |
| `{origin}/shares/{code}/preview` | GET | open share link | none (public preview) | none | Yes (on load) | — | — |
| `https://api.z.ai/...` (business) | per API | business-named auth | auth | refresh token | Yes | logout | — |
| `https://zcode.z.ai/docs` | GET | help docs | none | none | manual | — | — |
| `ZCODE_REMOTE_ASSET_CDN_BASE_URL` | GET | asset load | static assets | none | Yes | env | — |
| localhost `/api/server-info`, `/api/rpc-host-capability` | GET/POST | local server runtime | local | none | local | — local only — | EVID-SESSION-001 |

CSV: `endpoint-matrix.csv`.