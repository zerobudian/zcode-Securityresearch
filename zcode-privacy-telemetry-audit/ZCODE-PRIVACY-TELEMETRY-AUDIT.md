# ZCODE_PRIVACY_TELEMETRY_AUDIT
### Deep source audit of repository / workspace snapshot, telemetry, uploads, logs, model context and privacy behavior

**Audit date:** 2026-09-21
**Auditor:** independent security / privacy researcher (self-contained, offline-reviewable package)

---

## Audited version (must be read as the frozen baseline)

| Item | Value |
|---|---|
| Repository | https://github.com/zai-org/ZCode.git |
| Branch | `main` |
| Commit SHA (HEAD) | `872ad960de7ec172591f7e1952f7849229f94521` |
| Commit subject | `feat: open source` |
| Commit author / date | `wuweiqi` / 2026-09-21 05:14:32 +0800 |
| Git history depth | 2 commits (`Initial commit` + `feat: open source`); **no tags** |
| Package version (`package.json`) | `3.14.0` |
| Desktop/CLI runtime version | `ZCODE_VERSION` (build-time) |
| Audit reference commit | `872ad960de7ec172591f7e1952f7849229f94521` |
| Graph/evidence SHA-256 root | See `SHA256SUMS.txt` |

> The upstream history contains a single squashed “open source” commit on top of `Initial commit`. There is **no granular public git log** of the historical telemetry/snapshot changes, so the historical timeline in this report is reconstructed from (a) current source, (b) public issue/community claims, and (c) cross-references inside the code. Everything marked historical is treated as an *investigation lead*, never as current fact.

---

## Executive summary (technical)

At the audited commit, ZCode is a **large Electron + Node/Clients** product (React web + host/agent services). The central privacy-related findings are:

1. **No cloud “repo snapshot / workspace archive / scan-and-index upload” pipeline is present in the audited source.** Every historical keyword associated with the reported feature — `repoSnapshotIndexingEnabled`, `repoSnapshotIndexingUserConfigured`, `lastAcceptedManifestHash`, `uploadCredentialHandle`, `captureStage`, `snapshot/upload-credential`, `repo-snapshot`, `repo-snapshot.tar.gz.enc`, `AES-256-CTR`, `RSA-OAEP`, `~/.zcode/v2/checkpoints/` cloud-upload semantics — is **absent** from the current `packages/` tree. The only object-storage (Alibaba OSS `x-oss-*`) upload that exists is the **user-initiated feedback attachment** flow. The term “checkpoint” in the current tree refers to **local Git checkpoints** (diffs stored under `~/.zcode/v2/checkpoints/`), not to a cloud snapshot. *(CONFIRMED/NOT FOUND — see Findings 1-4.)*

2. **Product telemetry is real and enabled by default** (`ZCODE_TELEMETRY_ENABLED = true`, hardcoded). Two outbound telemetry channels exist: a **custom event report** channel and **ARMS RUM** (performance/error monitoring). Both derive their endpoint from **runtime environment variables** (`ZCODE_TELEMETRY_REPORT_ENDPOINT`, `ZCODE_ARMS_RUM_ENDPOINT`); neither is a hardcoded URL baked into every build. The event payload carries **account/user_id when logged in**, a **persistent device identifier** (`device_mid` in `~/.zcode/v2/telemetry-state.json`), OS/version, app version, and bounded **sanitized** free-text, plus `talk_id`/`message_id` conversation links. A redaction layer (`telemetryRedaction.ts`, `redactArmsEventBatch`) normalizes URLs/paths/emails/credentials and model identity **on the copy that leaves the machine**. *(CONFIRMED — Findings 5-7.)*

3. **No evidence that telemetry carries raw prompt text, source code, tool arguments, or workspace paths in the audited paths.** The event payload’s only free-text fields (`event_text`, `event_extra_detail`) are passed through `sanitizeTelemetryEventDetail`, and the model/provider identifiers are whitelist-normalized. This is distinct from, and must not be confused with, **model inference**, which by design transmits the conversation and tool context needed to do the task. *(CONFIRMED for the audited event paths; inference by design.)*

4. **Local persistence is comprehensive but on-device.** Sessions/scripts: `~/.zcode/v2/sessions/`, `~/.zcode/v2/tasks-index.sqlite`. Device identity: `~/.zcode/v2/telemetry-state.json`. Logs: `~/.zcode/v2/logs/YYYY-MM-DD.log`, 14-day retention. Git checkpoints: `~/.zcode/v2/checkpoints/`. **No client-side evidence of automatic server-side conversation-history sync, cloud restore, or repo upload.**

5. **Attachments are NOT pre-uploaded on selection.** The composer run-path for **local-path attachments performs zero upload** (the reference carries the absolute path); only inlined `dataBase64`/`textContent` (pasted images, text) are transmitted through a put transaction (needed to send pasted content to the provider). Feedback attachments use a separate credential → OSS flow that is **user-initiated**. *(CONFIRMED — Findings 8-9.)*

### Top findings (summary)

| ID | Severity | Title | Status |
|---|---|---|---|
| FINDING-001 | INFO | Historical cloud repo-snapshot pipeline absent in current source | NOT FOUND (historical) |
| FINDING-002 | INFO | “Checkpoint” = local Git checkpoint, no upload | CONFIRMED |
| FINDING-003 | MEDIUM | Telemetry enabled by default; two channels; device+account correlation | CONFIRMED |
| FINDING-004 | LOW | Telemetry endpoint/ARMS endpoint resolved from env at runtime | CONFIRMED (env-gated) |
| FINDING-005 | LOW | Telemetry payload binds device_mid + user_id + talk/message ids | CONFIRMED |
| FINDING-006 | MEDIUM | ARMS RUM is full product monitoring (perf/errors); redacted before egress | CONFIRMED |
| FINDING-007 | LOW | Attachment: local files not uploaded; pasted content inlined | CONFIRMED |
| FINDING-008 | LOW | Feedback attachments → OSS via server-issued credential | CONFIRMED |
| FINDING-009 | INFO | Server-side retention/training of received telemetry not determinable | UNKNOWN — SERVER SIDE |

Full detailed findings in section **Findings**.

---

## Scope & methodology

**Scope:** the open-source repository `zai-org/ZCode` at the audited commit, focus on data handling: model inference context, product telemetry, snapshot/checkpoint, uploads, feedback, logging, redaction, conversation storage, sharing, auth, MCP/plugins, and remote environments. Desktop/CLI/web entry points and the shared service/agent layers were all examined. Binary release artifacts (built Electron apps, closed-source server) were **not** in scope because only source is published.

**Methodology** (per the audit contract):
1. Clone official repo; freeze commit; record metadata.
2. Inventory all network-capable primitives and trace each to endpoint + payload + gate.
3. Trace local data touchpoints (workspace files, `.git`, secrets, terminal, prompts, attachments, logs).
4. Verify each historical privacy claim: claim → locate source → call graph → reachability → config gate → payload → destination → dynamic verification → conclusion.
5. Classify each into CONFIRMED / STILL PRESENT / CHANGED / PARTIALLY FIXED / FIXED / NOT FOUND / NOT VERIFIABLE.
6. Build evidence artifacts (source, snippets, call-chains, hashes, manifest) for independent re-review.

**Reachability caveat that governs the whole report:** several outbound paths take their endpoint from **runtime environment variables**. The audited *source* proves such a path *can* egress; whether it actually does at end-user runtime depends on the launcher/distribution setting the variable. The report states this explicitly wherever it applies and does **not** assert a specific host that source does not pin. Dynamic traffic capture (mitmproxy) was **not** possible in this sandbox; see **Dynamic verification**.

**Security boundary:** analysis used only public source and a synthetic throwaway repo; no production account, no real credentials, no probing of live services.

---

## Architecture & remote/cloud boundary

```
 ┌─ Local computer ─────────────────────────────────────────────┐
 │  Workspace files / .git / terminal / prompts / attachments     │
 │    │                                                           │
 │    ▼                                                           │
 │  ZCode desktop (Electron) + host/agent Node services           │
 │   ├── Context Builder ──► Model endpoint  (inference, by design)│
 │   ├── Product Telemetry (event report + ARMS RUM)              │
 │   ├── User feedback attachment → OSS via server credential     │
 │   └── Local stores: ~/.zcode/v2/sessions, logs, checkpoints,   │
 │        telemetry-state.json                                    │
 └──────┬───────────────────────────────────────────────────────┘
        │ HTTPS
        ▼
 ZCode service (https://zcode.z.ai default; ZCODE_BASE_URL override)
   /api/v1/... (inference, configs, off-peak, mcp/usage, share, oauth)
        │
        ├── Model provider(s) — e.g. bigmodel.cn — receive inference
        ├── ARMS/SLS telemetry sink (if endpoint set)
        └── Object storage (OSS) — feedback attachments only
```

Boundaries are **not** merged in this report: *local computer* vs *ZCode service* vs *object storage* vs *model provider* vs *third-party telemetry sink* are treated as separate trust domains. Nothing in the audited source uploads a workspace/repo archive; only inference (conversation/tool context), telemetry, OAuth, config, and user-initiated feedback attachments cross the boundary.

Mermaid render: `data-flow.mmd`.

---

## Network egress (summary)

13 rows catalogued (`NETWORK-EGRESS.md`, CSV `network-egress.csv`). Categorization:

- **A. Model inference** (rows 5,6,7,11): conversation/tool context to ZCode service default `https://zcode.z.ai/api/v1` (or `ZCODE_BASE_URL`/provider baseURL). Required to perform the agent task.
- **B. Product telemetry** (rows 1,2): custom event report + ARMS RUM; endpoints env-resolved, payload sanitized.
- **D. Explicit upload** (rows 3,4): feedback attachment credential + OSS upload (user-initiated).
- **E. Auth/OAuth** (rows 8,9).
- **F. Content/distribution** (rows 6,10,12,13): config, share preview, local server, asset CDN.

There is **no** category **C (snapshot/checkpoint upload)** in the current tree.

---

## Model inference (normal agent request — Q25)

**What a normal agent request actually contains:** the audited code transmits the *conversation and tool context required to complete the current task*: user prompt, prior messages, tool calls and their results, and code context **selected by the agent/tooling** (searched/read ranges, diffs, repo-map/context windows). This is **not** an unconditional upload of the whole repository.

- The codebase has **no** holistic “archive the whole workspace and POST it” function for inference.
- Context is assembled by a context builder / agent loop that includes only what the turn needs; whole files are read only when a tool (e.g. file read) selects them.
- **Having read-access to the repo is not the same as uploading the repo each request.** No supporting evidence for “every request uploads the entire repo” was found.

`DEFAULT_ZCODE_MODEL_CONTEXT_BUDGET_STRATEGY` and context-budget constants confirm bounded context assembly (node.ts:496,2248).

Auto helper model calls (title generation, summarization, context compaction) exist as normal model usage; e.g. `context_compaction` events and compaction telemetry. These are **inference**, not telemetry-of-content.

---

## Telemetry (Q17-Q20)

**Channels (2):**
1. Custom event report — `telemetryCore.ts` (`reportEvent`, `reportAppLaunch`, `reportAppDailyActive`). Payload fields listed in `evidence/call-chains/telemetry-report.md`.
2. ARMS RUM — `appARMSBootstrap.ts` (perf, web vitals, JS errors, native crash, api/rpc). Redacted in `beforeReport`.

**Event types observed (from `conversationTelemetrySupervisor.ts`):** `send_btn` (eventType `ck`), `agent_step` (`agent_trace`), `message_completion` (`agent_trace`), `context_compaction` (`agent_trace`), plus `session_create`, `app_launch`, `app_daily_active`, tool-lifecycle events.

**Identifiers:** `user_id` (account-linked when logged in), `device_mid` (persistent device UUID from `~/.zcode/v2/telemetry-state.json`), `talk_id`/`message_id` (conversation/message links), `event_id`, `mac_id` (hardcoded `""`), OS/version, `app_version`, `screen_resolution`, `client_timezone`, `client_language`, `marketing_params` (OAuth attribution). This binds device → account → conversation, i.e. **account-linked & conversation-linked**, not purely anonymous.

**Does telemetry carry prompt/source/paths?** For the audited event paths, the only free-text (`event_text`, `event_extra_detail`) passes through `sanitizeTelemetryEventDetail` (EVID-REDACTION-001), which strips URLs-of-query, normalizes paths/emails/tokens/AWS/GitHub/OpenAI keys to placeholders and bounds to 2 KiB; model identity is whitelist-normalized. **No raw source-code content appears in the audited telemetry payload.**

**Q18 `/api/v1/event/report`:** the current event-report endpoint is resolved at runtime from `ZCODE_TELEMETRY_REPORT_ENDPOINT`; the public `event/report` path in the current source appears only in a comment (telemetryCore.ts:441). The exact live path is **not pinned** in source and is env/gate dependent.

---

## Repo snapshot special audit (Q1-Q14)

This was the top-priority area. Full evidence in `evidence/call-chains/snapshot-checkpoint.md`.

**Q1 Current repo/workspace snapshot?** No cloud snapshot pipeline. A **local Git checkpoint** feature exists (`gitCheckpointStore.ts`, `gitCheckpointHelpers.ts`) that stores per-workspace checkpoint **metadata/diffs** under `~/.zcode/v2/checkpoints/{workspaceHash}/{checkpointId}.json` and a git ref `refs/zcode/checkpoints/{workspaceHash}/{checkpointId}`. It does **not** create an encrypted tarball and does **not** upload.

**Q2 Upload?** No. No upload-credential, no OSS/S3 write, no `lastAcceptedManifestHash` acknowledgement. Only the feedback attachment path writes to OSS.

**Q3 Trigger?** Local checkpoints are tied to session/task lifecycle (workspace worker integration). No background workspace scanner + archive + upload loop exists for cloud.

**Q4 Automatic?** The local checkpoint capture is automatic as part of the recovery feature; there is no automatic cloud upload of any kind.

**Q5-Q7 `.git` / objects / reflog / history / deleted secrets?** Because there is no workspace archive step at all, there is **no code path that packages `.git`, git objects, or reflog for upload**. Therefore “deleted-but-in-git-history secrets enter a snapshot” is **not reachable** in the audited commit. (This is a *removal*, not a guarantee about other data handling.)

**Q8 `.env` filtering:** Within normal *inference/file-read* context building, files are read by explicit tool selection, not bulk-archived, so `.env` is not shipped wholesale. No dedicated cloud-snapshot secret-scanner/entropy filter exists because the cloud-snapshot feature no longer exists. (The redaction layer targets telemetry, not file archiving.)

**Q9 Secret filter vs git history:** no such filter exists — because there is no git-history-archiving feature in this commit.

**Q10-Q11 `repoSnapshotIndexingEnabled=false`:** The keyword does not exist. It **cannot** gate capture/upload because that capture/upload does not exist. (The question is moot for the current source.)

**Q12-Q13 Destination / who decrypts:** No archive is produced or transmitted; therefore no encryption/decryption owner applies. The historical AES-256-CTR + RSA-OAEP claims could not be located in the audited commit. *(Historical topics only.)*

**Q14 Local snapshot retention:** Git checkpoint metadata is local; retention is not separately documented in the audited source (managed with deletion/cleanup utilities). Column marked INFO.

**Conclusion:** the historical cloud repo-snapshot feature is **NOT FOUND / REMOVED** in this commit; the local Git-checkpoint feature remains and is on-device only.

---

## Settings gate reality (Q28-Q29)

- **Telemetry:** `ZCODE_TELEMETRY_ENABLED` is a hardcoded `true` (env.ts:50). There is **no user-facing UI toggle in source that flips this constant**. Actual egress is gated by whether the runtime env endpoint is set. A user **cannot** flip the telemetry flag from the published source-default UI; the realistic control is launcher-level (omit the endpoint env var) or uninstalling. This is an important, honest caveat.
- **Inference:** controlled by using/not using the product; not a privacy toggle.
- **Feedback attachments:** user-initiated.
- **`.git`/checkpoint:** local; cleanup via local settings.

**Q29 “What still leaves the machine after toggling ‘off’?”:** since there is no repo snapshot at all, the remaining egress is: (a) model inference (by design, not toggleable separately from using the tool), (b) telemetry if the env endpoint is present and telemetry not disabled at build/launch, (c) user-initiated OAuth/feedback/share.

---

## Secret handling / redaction (Q8-Q9, section 24)

- **Local file access:** files are read on-demand by explicit tool selection (inference context). No bulk secret scanner for cloud upload exists (feature absent).
- **Telemetry redaction** (`telemetryRedaction.ts`): `redactTelemetryText` (URL query stripped, paths/emails/keys→placeholders, bounded 2048B, scan limit 4096B), `redactTelemetryUrl` (local paths → `local_file`, route segments high-cardinality → `{segment}`), model/provider normalization to builtin whitelist else `custom`. Applied **to the outbound copy** (ARMS `beforeReport` runs redaction last, after ingest/summary) while local logs/crash archives keep original values. This answers *when* redaction happens: **before upload, not merely before UI display.**

**Redaction is not applied to inference**, which by design sends the task-relevant content.

---

## Upload destination & path (Q12, section 13)

Feedback attachment: client `POST /feedback/attachment/upload-credential` (ZCode service) → server returns short-lived OSS credential (`oss.host`, policy, `x-oss-*` signature fields, `max_size`, `callback`) → client streams multipart to the OSS host with signature fields; OSS posts `callback` to server. This is the only object-storage path. See `evidence/call-chains/feedback-attachment-upload.md` and `buildOssFormFields` (feedbackHttpClient.ts:1054).

---

## Retry / persistence (Q15-Q16, section 14)

- **Telemetry:** bounded retry `attempt=1..2`, backoff 300ms (429→1000ms); pending reports held in-memory (`pendingReports` set) and `flushPendingReports` drains on shutdown with a timeout. **Not** durable across restarts — a crash before flush loses in-flight events. Device id persists on disk.
- **Feedback attachment:** per-folder retry helper (e.g. line ~920 retries with delay) bounded; honors abort.
- **Repo snapshot:** none.

So for telemetry: failures are retried within a run and drained on graceful exit; there is **no on-disk pending queue** for telemetry across crashes. *Q16: restart does not resume a pending upload because there is no such upload.*

---

## Attachments (Q23, section 21)

- **Normal chat:** `attachmentUpload.ts` maps a local-path attachment to a ref carrying the **absolute path, zero upload**; only `dataBase64` (pasted screenshots) or `textContent` are uploaded via a put transaction (begin/chunk/commit) with abort. This happens as part of the send transaction, **not** at drag/pick time. → **Q23: no pre-send upload on selection.**

---

## Feedback / diagnostics (Q22, section 22)

Feedback attachments (`FeedbackHttpClient.uploadFile`) upload the user-selected file to OSS. The audited code does **not** auto-attach screenshots/logs/account-hash/wokspace-metadata by default to these tickets; attachment is explicit. Whether the UI pre-checks a “include logs” box is a UI concern not exhaustively enumerated here; the upload *call* is gated on the user path.

---

## Logging (Q21, section 23)

- Location: `~/.zcode/v2/logs/YYYY-MM-DD.log` (desktop main; E2E uses a worker dir). CLI/Stdio agent logs: `~/.zcode/cli/log/zcode-*.jsonl` (also `apt` legacy `.zcode` folders).
- Retention: `LOG_RETENTION_DAYS = 14` (logRetention.ts:4).
- Level: debug/info/warn/error; main-writer in `logger.ts`.
- **Can logs record prompt/code/tool args?** Logging uses structured `logger.*` calls; telemetry retries deliberately log only **sanitized attempt metadata** and explicitly do NOT log payload/response/raw error (telemetryCore.ts:441-446). The agent/CLI may log lifecycle diagnostics; whether a given log line contains tool arguments depends on the call site. **No evidence found that Authorization headers / cookies / tokens are written by the logging core**; a separate `redactUpdateFeedUrlForLog` pattern exists for updater URLs. This topic is discussed in `LOGGING.md`; where a call site was not enumerated it is marked NOT VERIFIABLE.

---

## Redaction (section 24)

Covered above: telemetry text/URL/model redaction applied before egress (`telemetryRedaction.ts`, `redactArmsEventBatch`). Scope is telemetry/crash, not inference.

---

## Conversation storage (Q24, section 25)

- **On-disk:** sessions at `~/.zcode/v2/sessions/{workspaceHash}/{taskId}.json` (+ `.deleted.json`), index `~/.zcode/v2/tasks-index.sqlite` (paths.ts:183-217). Tasks/scripts persisted locally (`tasks-index.sqlite`).
- **Cloud:** a **conversation share** service exists (`conversationShareService`, `ConversationShareHttpClient` → `{baseUrl}/shares/...`), activated only when the user explicitly shares/publishes. **No automatic server sync of conversation history was found.** Whether the server stores shared/uploaded conversations is **UNKNOWN — SERVER SIDE**.

**Q24 share fields:** sharing transmits the chosen messages/conversation payload through the share API; attachments referenced by absolute local path in chat do **not** upload unless they were inlined/pasted. The exact share schema is not exhaustively re-listed here; the share call chains are enumerable in source.

---

## Auth / credentials (section 27)

- OAuth against `DEFAULT_ZAI_OAUTH_ORIGIN = https://chat.z.ai` (`ZAI_OAUTH_ORIGIN` override), token `/api/v1/oauth/token`, client id default `client_P8X5CMWmlaRO9gyO-KSqtg`.
- `zaiBusinessTokenResolver` resolves business-named auth tokens; tokens persisted via OAuth credential repo (storage form/encryption of tokens is a server/storage detail — see `LOGGING.md`/Limitations; no evidence of token logging).
- Provider API baseURL configurable (`api.baseUrl`), enabling self-host/live-backend use (relevant to which server receives inference).

---

## MCP / plugins / skills (section 28)

- Workspace-level MCP is auto-connected at runtime; enabling MCP may read configured env, auth headers, tokens, OAuth (see NOTICE). Server MCP identity headers + `/api/v1/mcp/usage` quota queries are legitimate resource-accounting, not content upload.
- Skill dirs/configs are local; no evidence of automatic skills-cloud-sync in the audited commit (marked NOT VERIFIABLE where absent).

---

## Browser / remote (sections 29-30)

- Embedded browser + Browser Use: can read pages, screenshots, recordings; imported login state may expose user data to sites visited; visiting a page contacts that site. NOTICE governor: “访问网页本身也会与网站及其加载的服务通信.”
- Computer Use placeholder returns unavailable (NOTICE; `packages/zcode-cua/index.js`).
- Remote workspaces: operations may execute server/SSH/WSL/container side; network reachability/credentials depend on deployment. No default OS sandbox in the shared agent adapter (NOTICE).

---

## Historical issues & timeline (section 32-33)

`VERSION-TIMELINE.md` contains the full table. Headline:

- Reported feature: cloud workspace/repo snapshot + AES-256-CTR/RSA-OAEP encrypted tarball + OSS upload + `repoSnapshotIndexingEnabled`.
- Current source: **all keywords absent**; only local Git checkpoint + feedback OSS remain.
- Interpretation (code-fact only, no motive inference): the cloud-snapshot capability is **NOT FOUND in the audited open-source commit**. Because the repo was squashed into a single open-source commit, within-repo commit-diff history for this feature **does not exist** for independent verification of *when* it was removed. Therefore the classification for individual removal steps is **NOT VERIFIABLE (history squashed)**, and for the current presence is **NOT FOUND**.

---

## Policy / NOTICE vs source (Q31-Q32)

`POLICY-VS-CODE.md` maps the NOTICE’s claims to source. Headline status:

- NOTICE’s operational/warning disclosures (permissions, hooks, MCP auto-connect, browser/remote execution, no sandbox) are broadly **MATCH** or **PARTIAL** with supporting source.
- On the specific “snapshot/upload” question, the NOTICE uses the phrase “任务快照、Git 检查点和会话恢复…” — the audited code provides **local** task/session state and Git checkpoints, so the statement is consistent with source (a cloud-snapshot upload is not described, and none exists). Marked **MATCH / PARTIAL**.
- **Privacy policy vs code:** no separate end-user privacy policy asserting “we do not upload repos” could be audited from source. Where a policy claim exists that can’t be checked from client code, it is marked **UNKNOWN / depends on server**. **No legal violation is asserted without evidence.**

---

## Training claims (section 63)

**No evidence** in the audited client source of an explicit “training / model-improvement / optimization opt-in” flag, nor of the server training a model from uploaded repos. Where telemetry/inference data reaches a ZAI/3P sink, its server-side training use is:
> Server-side model-training use cannot be determined from the audited client source.

Server retention/deletion/processing: **UNKNOWN — SERVER-SIDE BEHAVIOR** wherever not derivable client-side.

---

## Findings (detailed)

### FINDING-001 — Historical cloud repo-snapshot pipeline absent
- **Severity:** INFO · **Confidence:** CONFIRMED (source) / evidence of absence
- **Component:** snapshot/upload · **Trigger:** n/a · **Data:** n/a
- **Historical status:** reported public topic · **Current status:** NOT FOUND
- **Evidence:** grep over `packages/` — `repoSnapshotIndexingEnabled`, `repoSnapshotIndexingUserConfigured`, `lastAcceptedManifestHash`, `uploadCredentialHandle`, `captureStage`, `snapshot/upload-credential`, `AES-256-CTR`, `RSA-OAEP`, `repo-snapshot` → 0 matches. Call-chain `evidence/call-chains/snapshot-checkpoint.md`.
- **Impact:** removes the highest-risk historical concern from the current source.
- **Recommendation:** keep monitoring; verify release binaries separately (source ≠ shipped package).

### FINDING-002 — “Checkpoint” is a local Git checkpoint
- **Severity:** INFO · **Confidence:** CONFIRMED (reachable source path)
- **Component:** git/recovery · **Trigger:** session/task lifecycle
- **Data:** diff metadata, local · **Destination:** none (local disk)
- **Evidence:** EVID-SNAPSHOT-001, EVID-SNAPSHOT-002.

### FINDING-003 — Product telemetry enabled by default, 2 channels, device+account correlation
- **Severity:** MEDIUM · **Confidence:** CONFIRMED (source)
- **Component:** telemetry · **Trigger:** app launch / events
- **Data:** usage events, sanitized free text, identifiers · **Destination:** runtime-configured endpoint
- **User control:** no source-default UI toggle flips `ZCODE_TELEMETRY_ENABLED`; endpooint env-gated
- **Evidence:** EVID-TELEMETRY-001/002/003/006; env.ts:50.
- **Impact:** repeated device-linked behavioral metrics to a third-party RUM sink.
- **Recommendation:** make telemetry opt-in with an explicit, source-reachable toggle; document data categories.

### FINDING-004 — Telemetry/ARMS endpoints env-resolved
- **Severity:** LOW · **Confidence:** CONFIRMED
- **Evidence:** env.ts:53-58; appARMSBootstrap.ts:266-268.
- **Recommendation:** keep env pinning and document launcher defaults.

### FINDING-005 — telemetry payload identity linkage
- **Severity:** LOW · **Confidence:** CONFIRMED
- **Evidence:** telemetryCore.ts:383-406; EVID-TELEMETRY-003.
- Data is account-linked + device-linked + conversation-linked (not purely anonymous).

### FINDING-006 — ARMS RUM full monitoring, redacted before egress
- **Severity:** MEDIUM · **Confidence:** CONFIRMED
- **Evidence:** appARMSBootstrap.ts; EVID-REDACTION-001.
- Perf/errors captured; redaction on outbound copy.

### FINDING-007 — Attachments: local files not uploaded; pasted content inlined
- **Severity:** LOW (informational, favorable) · **Confidence:** CONFIRMED
- **Evidence:** attachmentUpload.ts:53-79; EVID-ATTACHMENT-001.
- No pre-send upload on selection.

### FINDING-008 — Feedback attachment → OSS via server credential
- **Severity:** LOW · **Confidence:** CONFIRMED
- **Evidence:** feedbackHttpClient.ts:105-150,932-1075; EVID-ATTACHMENT-002.

### FINDING-009 — Server-side retention/training not determinable
- **Severity:** INFO · **Confidence:** UNKNOWN
- **Evidence:** none in client source.
- Marked **UNKNOWN — SERVER SIDE**.

### FINDING-010 — `ZCODE_TELEMETRY_ENABLED` is a hardcoded default (no UI flip in source)
- **Severity:** LOW/MEDIUM · **Confidence:** CONFIRMED
- **Evidence:** env.ts:50.

### FINDING-011 — Git history squashed ⇒ removal timeline not independently verifiable
- **Severity:** INFO · **Confidence:** CONFIRMED (2 commits, no tags)
- **Impact:** historical “when removed” is NOT VERIFIABLE from this repo snapshot.

---

## Answers to Q1–Q32

| Q | Answer |
|---|---|
| Q1 | Current repo/workspace snapshot: **No** (only local Git checkpoint) |
| Q2 | Upload: **No** (only feedback attachments to OSS) |
| Q3 | Trigger: local checkpoints on session/task lifecycle |
| Q4 | Automatic: yes for local checkpoints; no cloud upload |
| Q5-Q6 | .git/objects/reflog/history into snapshot: **No** (no archive pipeline) |
| Q7 | Deleted-secrets-in-history entered snapshot: **No** via snapshot path |
| Q8-Q9 | .env filtering / git-history secret filter: N/A (feature absent); telemetry has redaction |
| Q10-Q11 | `repoSnapshotIndexingEnabled=false`: keyword absent; moot |
| Q12 | Snapshot destination: n/a; feedback→OSS |
| Q13 | Who can decrypt: n/a (no archived upload) |
| Q14 | Local snapshot data: git-checkpoint metadata on disk |
| Q15-Q16 | Retry/restart: telemetry bounded retry, in-memory drain, not durable-across-crash; no pending upload to resume |
| Q17 | Telemetry channels: event report + ARMS RUM |
| Q18 | `event/report` current role: runtime-resolved report endpoint; exact path env-dependent |
| Q19 | Identifiers: user_id, device_mid, talk/message_id, event_id, OS, app_version |
| Q20 | Prompt/source/paths in telemetry: **no raw content** in audited event paths (sanitized) |
| Q21 | Logs may include prompt/code/tool args: **possible at call sites; not in telemetry-retry logs; not in logging core** — call-site-dependent, partly NOT VERIFIABLE |
| Q22 | Feedback uploads: user-selected attachments only |
| Q23 | Pre-send attachment upload: **No** on selection |
| Q24 | Share fields: messages/conversation payload via share API; attachments by ref unless inlined |
| Q25 | Normal inference context: conversation + task-selected code, not whole repo |
| Q26 | Inference vs telemetry: inference = task context; telemetry = usage/perf diagnostics |
| Q27 | Snapshot: none (local checkpoint only) |
| Q28 | Non-disableable automatic requests: no source-default UI toggle for telemetry; inference follows product use |
| Q29 | After toggling off: telemetry could still egress if env endpoint present (no UI gate); inference follows use |
| Q30 | vs historical: cloud-snapshot feature NOT FOUND in current commit |
| Q31 | NOTICE vs source: MATCH / PARTIAL, no snapshot-upload discrepancy |
| Q32 | Privacy policy vs code: any uncheckable server claims marked UNKNOWN; no unsupported legal assertion |

---

## Limitations

- **Binary vs source:** only source is published; a shipped binary could differ. Verify release artifacts separately.
- **Environment-gated endpoints:** telemetry/ARMS actually egress only if the runtime env variables are set; the audit proves the path exists, not that a particular build enables it.
- **Squashed history:** cannot independently date the removal of the historical snapshot feature within this repo.
- **Dynamic verification:** no live traffic capture executed; see `methodology/`.
- **Server retention/training:** not derivable from client source → UNKNOWN — SERVER SIDE.
- **UI toggle enumeration:** whether a particular settings page binds a telemetry switch to the env/gate is a UI breadth that was only partially enumerated; flagged accordingly.

---

## Recommendations

1. Ship an **explicit, reachable, source-level opt-in/opt-out** for product telemetry (currently a hardcoded `true`).
2. Document the **launcher-set telemetry/ARMS endpoints** so users know *if* and *where* telemetry reports go.
3. Keep the local Git-checkpoint design (no cloud archive) and document retention/cleanup.
4. Publish a **privacy policy** that can be audited against client source (server behavior currently UNKNOWN).
5. Any future re-introduction of cloud repo indexing should be **opt-in, .git-aware, and covered by a policy**.
6. Add generated **runtime traffic** evidence via a documented mitm harness in `methodology/test-procedure.md`.

---

## Conclusion

At commit `872ad960de7ec172591f7e1952f7849229f94521`, ZCode ships **no automatic repository/workspace snapshot upload** and no pending-upload retry pipeline. Product telemetry (custom events + ARMS RUM) is enabled by default, device- and account-correlated, sanitized at the outbound boundary, and resolved from runtime env endpoints. Attachment handling does not pre-upload local files on selection; only pasted content and user-initiated feedback attachments cross the boundary. The historical cloud-snapshot feature is **NOT FOUND** in the audited open-source commit; because the git history is squashed, its removal timeline is **not independently verifiable** from this repository. Server-side retention/training and the exact production telemetry endpoints remain **UNKNOWN / server-side**.

See `README.md` for how to re-verify every conclusion from the packaged evidence.