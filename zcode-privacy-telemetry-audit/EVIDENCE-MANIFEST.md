# Evidence Manifest

Commit `872ad960de7ec172591f7e1952f7849229f94521` · all evidence files are verbatim upstream copies (SHA-256 recorded).

| Evidence ID | Upstream path | Local evidence path | SHA-256 | Upstream lines | Symbol | Description |
|---|---|---|---|---|---|---|
| EVID-ATTACHMENT-001 | packages/ui/src/v4/composer/attachmentUpload.ts | EVID-ATTACHMENT-001__attachmentUpload.ts | `2b6575494b6279441a6fbe7bdc2b3c12fe433d90c05e9b7edec649a248c94b8c` | 80 | uploadComposerAttachment | local-path=zero upload; base64 inline |
| EVID-ATTACHMENT-002 | packages/services/src/feedback/feedbackHttpClient.ts | EVID-ATTACHMENT-002__feedbackHttpClient.ts | `bd899d5667df89586d66bb882aa0c8b3df9ef24636fda37aed1af26639426c6b` | 1087 | uploadFile/uploadOssForm | feedback attachment credential+OSS upload |
| EVID-FEEDBACK-001 | packages/services/src/feedback/feedbackService.ts | EVID-FEEDBACK-001__feedbackService.ts | `a2b54dca0f77e647d41cde35eb44d8e37636b6b026515c9cd4c728f2135d3d24` | 276 | FeedbackService | feedback orchestration |
| EVID-LOGGING-001 | packages/desktop/src/main/logger.ts | EVID-LOGGING-001__logger.ts | `b0f7a8418c48b40648a56eff354565649d924b96c1f047555a97ac5c9ff6ede1` | 111 | getLogDir/write | main log writer to ~/.zcode/v2/logs |
| EVID-LOGGING-002 | packages/desktop/src/main/logRetention.ts | EVID-LOGGING-002__logRetention.ts | `9447cfca3719e51b9836b18556f520481233288d64de2467fe8e7cdf4c492e1f` | 98 | LOG_RETENTION_DAYS | 14-day log cleanup |
| EVID-PROTO-001 | packages/shared/src/zcode-protocol-v4/snapshot.ts | EVID-PROTO-001__snapshot.ts | `6ab497d6b75eece5eadbfd60f43c721010d4b1a0ab74bcdce6ced5512226cf82` | 508 | conversationSnapshotSchema | conversation snapshot state schema |
| EVID-REDACTION-001 | packages/shared/src/telemetryRedaction.ts | EVID-REDACTION-001__telemetryRedaction.ts | `321952311f5e36187edbe7b365c51ceded8f67869c0faffec986b370e6540d60` | 257 | redactTelemetryText/Url/ModelId | outbound telemetry redaction |
| EVID-SESSION-001 | packages/services/src/paths.ts | EVID-SESSION-001__paths.ts | `e20fbeb4e88e9021bd099a50ce8b2bbc90ff823eefb57253d1f1ea11b6f7d1df` | 255 | getAppConfigDir | local store paths: sessions/checkpoints/logs/tasks |
| EVID-SNAPSHOT-001 | packages/services/src/git/repo/gitCheckpointStore.ts | EVID-SNAPSHOT-001__gitCheckpointStore.ts | `a56acc3a3e471051c170c69fb7e4bcffa177bd4a974ed561b742bc4d29b8cb93` | 98 | GitCheckpointStore.save | local git checkpoint metadata |
| EVID-SNAPSHOT-002 | packages/services/src/git/repo/gitCheckpointHelpers.ts | EVID-SNAPSHOT-002__gitCheckpointHelpers.ts | `ce8ea69f78a77bfa25bf3ae3ed4854957a63ea7591042617afc4bef335b016cf` | 207 | getCheckpointRefName | checkpoint ref/diff helpers |
| EVID-TELEMETRY-001 | packages/services/src/telemetry/telemetryCore.ts | EVID-TELEMETRY-001__telemetryCore.ts | `f3839b539fd9a56bacb3fb18297bfd751179b3965bdb62ff1a0055f9abfa04cc` | 635 | createTelemetryCore/reportEvent/sendReport | event report payload/retry |
| EVID-TELEMETRY-002 | packages/desktop/src/main/appARMSBootstrap.ts | EVID-TELEMETRY-002__appARMSBootstrap.ts | `5c7004e7def4c97a582b841ebde3dbe67cdfdf3ff0ff2ba11b68de5ddf24245a` | 268 | startArmsRum/init | ARMS RUM bootstrap + redaction gate |
| EVID-TELEMETRY-003 | packages/services/src/device/deviceMid.ts | EVID-TELEMETRY-003__deviceMid.ts | `f57e0f2e80cad0b1a3569da7e9d5f286a83489e5cd01feb7849dc3ed74832372` | 212 | ensureDeviceMid | device identity file ~/.zcode/v2/telemetry-state.json |
| EVID-TELEMETRY-004 | packages/shared/src/env.ts | EVID-TELEMETRY-004__env.ts | `69c7f96325027d874be61367da3e80f92e751508070275305209ad739da65d2c` | 63 | ZCODE_TELEMETRY_ENABLED/..._ENDPOINT | telemetry env flags |
| EVID-TELEMETRY-005 | packages/services/src/providers/api/nodeApiNetwork.ts | EVID-TELEMETRY-005__nodeApiNetwork.ts | `1492c5cbcb1eb44ba01c676a33626490dc05839b4a9c7cf5ce23fc7fd1ff6813` | 252 | createHostApiNetworkTransport | HTTP transport/proxy for host API |
| EVID-TELEMETRY-006 | packages/ui/src/v4/telemetry/conversationTelemetrySupervisor.ts | EVID-TELEMETRY-006__conversationTelemetrySupervisor.ts | `03f4eb4e253ce22bff108efbff99028e4c8a441e3f63bd8d0572ec57af1f7ae2` | 1958 | telemetry events | event catalog: send_btn/agent_step/message_completion/context_compaction |
| EVID-TELEMETRY-007 | packages/services/src/providers/api/nodeApiClient.ts | EVID-TELEMETRY-007__nodeApiClient.ts | `4792b3c9692c6e6483b8cc91814b6587f38feb1a070103de85295633dd1c49bb` | 176 | NodeApiClient | model/API client (inference path) |

Call-chains under `evidence/call-chains/`:
- snapshot-checkpoint.md (local Git checkpoint, no cloud upload)
- telemetry-report.md
- arms-rum.md
- feedback-attachment-upload.md
