# ZCode — Policy / NOTICE vs Source

Commit `872ad960de7ec172591f7e1952f7849229f94521` · 2026-09-21

Status: **MATCH** · **PARTIAL** · **AMBIGUOUS** · **POSSIBLE GAP** · **UNKNOWN**
Source evidence shown per row. No legal-violation judgement is made without evidence.

| Official statement (NOTICE.md) | Source evidence | Status |
|---|---|---|
| “共享 Agent 执行适配器默认不提供操作系统沙箱…不能当作系统级隔离保证” | Node/host agent executes tools under OS account; no OS-sandbox wrapper found in services. | MATCH |
| “工作区级 MCP 纳入自动连接；启用 MCP 可能使用配置中命令、环境变量、认证头或 OAuth” | Server-MCP usage + cred resolution + identity headers (node.ts:1638,2378). | MATCH |
| “内嵌浏览器可截图/录制页面；导入登录状态可能访问私人数据” | Brain/Browser Use + embedded webview (desktop). | MATCH/PARTIAL |
| “任务快照、Git 检查点和会话恢复不能替代对全部本地文件备份” | Git checkpoints + sessions are local (gitCheckpointStore, paths.ts). Consistent: no cloud archive claimed. | MATCH |
| “Computer Use 包为不可用占位实现” | `packages/zcode-cua/index.js` placeholder. | MATCH |
| Telemetry / event reporting disclosed? | NOTICE does not explicitly quantify telemetry; payload is device/account-linked and env-gated. | POSSIBLE GAP (disclosure vs behavior) |
| “另一端…在 Web 或远端工作区操作可能发生在服务端/SSH/WSL/容器” | Remote env support (agentProxyEnv, ZCODE_BASE_URL). | MATCH |
| Lifecycle hooks may receive prompts/paths/tool args/results and can modify tool input | SessionStart/UserPromptSubmit/PreToolUse/PostToolUse hooks exist; NOTICE is explicit. | MATCH |

### Privacy Policy vs code (Q32)
- No audited **separate end-user privacy policy** in the repo. Claims made by any external ZAI privacy page about retention/training/deletion of telemetry, inference, shared conversations cannot be checked against client source.
- Where such claims exist they are marked **UNKNOWN — SERVER SIDE** or **POSSIBLE GAP** until server-side corroboration.
- **Notable potential gap:** source-default telemetry is **on** (`ZCODE_TELEMETRY_ENABLED = true`) with no source-visible UI toggle; if a policy implies “telemetry only with consent”, that would be a REAL/policy discrepancy — but because it depends on the shipped UI and env wiring, it is recorded as **POSSIBLE GAP**, not asserted as a violation.

Evidence rows: EVID-TELEMETRY-001/002, EVID-ATTACHMENT-002, EVID-SNAPSHOT-001/002, EVID-SESSION-001.