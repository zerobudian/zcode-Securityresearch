# Call Chain: "Checkpoint" — Local Git Checkpoints (NOT a cloud repo snapshot)

Audit: ZCode privacy / telemetry
Commit: `872ad960de7ec172591f7e1952f7849229f94521`

## Entry point (workspace-level Git checkpoint)

Workspace worker / git integration triggers checkpoint capture on session events.
↓ *EVID-SNAPSHOT-002__gitCheckpointHelpers.ts* (packages/services/src/git/repo/)
↓ ref name: `refs/zcode/checkpoints/{workspaceHash}/{checkpointId}` (getCheckpointRefName, line 28-31)
↓ diff built via git name-status / file diff (parseNameStatus etc.)
↓
*EVID-SNAPSHOT-001__gitCheckpointStore.ts* (packages/services/src/git/repo/gitCheckpointStore.ts)
↓ `save(meta: GitCheckpointMeta)`
↓ checkpointDir = `{appConfigDir}/checkpoints/{workspaceHash}`
↓ file = `{checkpointDir}/{checkpointId}.json`
↓ write manifest atomic (writeManifestAtomic)

## Where it stops

- The checkpoint metadata JSON contains *diff metadata* (added/deleted/modified paths + smaller diffs), not a full workspace archive, not `.git` object database.
- Everything is written into **local** `~/.zcode/v2/checkpoints/`. No upload, no object storage, no remote acknowledgment, no encryption-wrapped tarball, no upload-credential flow.

## Confirmed absence of the historical cloud-snapshot pipeline

- `repoSnapshotIndexingEnabled` / `repoSnapshotIndexingUserConfigured` → **no match** anywhere in `packages/`. (Grep, commit SHA above)
- `lastAcceptedManifestHash`, `uploadCredentialHandle`, `captureStage`, `snapshot/upload-credential`, `repo-snapshot` / `repo-snapshot.tar.gz.enc`, `AES-256-CTR`, `RSA-OAEP` → **no match** anywhere in `packages/`.
- The only object-storage (OSS, Alibaba `x-oss-*`) upload in the tree is the **feedback attachment** path — see `evidence/call-chains/feedback-attachment-upload.md`.

## Reachability conclusion

There is **no reachable code path** in the audited commit that builds a workspace/repo archive, encrypts it, acquires an upload credential, and pushes it to object storage for the purpose of "repo scanning / indexing snapshot". The word "checkpoint" in the current tree refers to the local, user-recoverable Git checkpoint feature. The historical cloud-snapshot feature is **NOT FOUND / REMOVED** in this commit.