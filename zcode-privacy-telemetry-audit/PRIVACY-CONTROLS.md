# ZCode — Privacy Control Matrix

Commit `872ad960de7ec172591f7e1952f7849229f94521` · 2026-09-21

Distinct concepts: **UI disabled** vs **collection disabled** vs **local storage disabled** vs **upload disabled** are not the same. `?` = unknown / server-side.

| Feature | Default | User Control | Stops Collection | Stops Local Storage | Stops Upload/Egress |
|---|---|---|---|---|---|
| Model inference | on (core function) | use/don't use; provider/baseURL self-host | N/A (using = governing) | N/A | No separate switch (inference is the product) |
| Custom product telemetry | `ZCODE_TELEMETRY_ENABLED = true` (hardcoded) | ⚠ no source-default UI toggle; env endpoint can be omitted | ⚠ unclear UI; build flag is const | events not persisted user-side beyond device id | Only if runtime `ZCODE_TELEMETRY_REPORT_ENDPOINT` is unset (launcher-level) |
| ARMS RUM | enabled if `ZCODE_ARMS_RUM_ENDPOINT` set | ⚠ env omittable | ⚠ | deviceMid persisted | unset env ⇒ SDK not initialized (appARMSBootstrap.ts:266-268) |
| Local sessions/task storage | on | delete locally | N/A local | removal via deletion | no cloud sync found |
| Git checkpoints | on (auto capture) | workspace feature | — | local | no upload |
| Feedback attachments | off until user files ticket | explicit attach | N/A | N/A | only on explicit send; abort supported |
| Chat attachment upload | on (pasted/inline only) | — | N/A | N/A | local paths not uploaded; pasted content sent at send-time |
| Conversation share | off until user shares | explicit | N/A | N/A | only on explicit share |
| OAuth login | off until login | logout | N/A | token store | only on login-driven flows |

Key takeaways:
- **Telemetry has no source-default UI flip.** The on/off that actually matters is the runtime env endpoint plus the build constant — a launcher/distribution decision, not a user settings toggle in the published source.
- **Inference cannot be “turned off” separately from using the tool.**
- **There is no repo/workspace snapshot setting** (feature absent), so column SNAPSHOT is N/A.