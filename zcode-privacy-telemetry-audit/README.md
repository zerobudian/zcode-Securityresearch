# ZCode Privacy & Telemetry Audit Package

A self-contained, offline-reviewable technical audit of data flow, telemetry, uploads, logs,
repo-snapshot behavior, model context and privacy controls in **Z.ai / zai-org ZCode**.

## Audit target

| Field | Value |
|---|---|
| Repository | https://github.com/zai-org/ZCode.git |
| Branch | `main` |
| **Commit SHA** | `872ad960de7ec172591f7e1952f7849229f94521` |
| Commit | `feat: open source` (wuweiqi, 2026-09-21 05:14:32 +0800) |
| Package version | `3.14.0` (no git tags) |
| Audit date | 2026-09-21 |

## Report entry points

- **Main report:** `ZCODE-PRIVACY-TELEMETRY-AUDIT.md` (executive summary, all chapters, findings, Q1–Q32).
- **Network egress:** `NETWORK-EGRESS.md` + `network-egress.csv`
- **Endpoints:** `ENDPOINTS.md` + `endpoint-matrix.csv`
- **Logging:** `LOGGING.md`
- **Policy/NOTICE vs code:** `POLICY-VS-CODE.md`
- **Historical timeline:** `VERSION-TIMELINE.md`
- **Matrices:** `SENSITIVE-DATA-MATRIX.md`, `PRIVACY-CONTROLS.md`
- **Diagram:** `data-flow.mmd` (Mermaid)

## How to read Evidence IDs

Any claim in the reports references an Evidence ID (e.g. `EVID-TELEMETRY-001`). Each ID maps
(in `EVIDENCE-MANIFEST.csv` / `.md`) to:

> Upstream path · Commit SHA · copied local path · symbol · line range · SHA-256 · why relevant

Evidence lives under:
```
evidence/
  source/       full source files preserved verbatim
  snippets/     minimal excerpts (none needed at this commit)
  call-chains/  entry→function→HTTP doc for key findings
  runtime/      dynamic-test records (NOT executed in this sandbox)
  screenshots/  UI evidence (none captured)
upstream/       LICENSE + NOTICE preserved from zai-org/ZCode
```

## Verify the evidence was not modified

```bash
sha256sum -c SHA256SUMS.txt
```
`SHA256SUMS.txt` also includes, per evidence file, the **upstream SHA-256** marker lines so you
can compare the packaged copy to a fresh checkout at the pinned commit.

## Find the upstream original

```bash
git clone https://github.com/zai-org/ZCode.git && cd ZCode
git checkout 872ad960de7ec172591f7e1952f7849229f94521
```
Then open the `Upstream path` listed in the evidence manifest and the line range in the reports.
Evidence copies match these files verbatim (no logic/format changes; any redaction is boxed and
labeled `REDACTED BY AUDIT`).

## Dynamic verification status

**Not executed in this sandbox.** `methodology/test-procedure.md` describes a canary-workspace +
mitmproxy procedure. The environment lacked a runnable Electron build and an authorized test
account. Conclusions in the reports are therefore **source-based** (reachable code paths) unless a
claim is explicitly marked CONFIRMED-with-runtime-evidence, of which there were none here.

## Limitations (read before citing)

1. **Source ≠ shipped binary.** Only source is published; a release build could differ. Verify release artifacts separately.
2. **Environment-gated endpoints.** Telemetry/ARMS destinations come from runtime env variables; the source proves the path exists, not that every build enables it.
3. **Squashed git history.** The repo is 2 commits, no tags; removal dates of historical features are not independently verifiable from this repo.
4. **Server-side behavior** (retention, training, deletion, endpoint config) is **UNKNOWN — SERVER SIDE** where not derivable from client source.
5. No legal violation is asserted anywhere without supporting evidence.

License notice for redistributed upstream files: see `THIRD-PARTY-NOTICES.md`.