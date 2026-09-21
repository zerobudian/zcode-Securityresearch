# Methodology — Commands

All commands run in the audit sandbox on 2026-09-21. No credentials were used.

## Clone & freeze
```bash
git clone https://github.com/zai-org/ZCode.git zcode-upstream
cd zcode-upstream
git remote -v
git log -1 --format='%H%n%ci%n%an%n%subject'
git branch --show-current
git tag            # (none)
git log --oneline  # (2 commits: Initial commit, feat: open source)
grep -m1 '"version"' package.json   # 3.14.0
```

## Evidence-preserving copy
```bash
mkdir -p /workspace/zcode-privacy-telemetry-audit/evidence/{source,snippets,call-chains,runtime,screenshots}
# copy the audited key files into evidence/source/EVID-*.ts (see EVIDENCE-MANIFEST.csv)
```

## Searches (keyword inventory)
Keyword grep sets are listed in `searches.md`. Ripgrep via the platform Grep tool with `glob=*.ts` was used; initial `grep --include=*.ts` shell globs failed under zsh (`no matches found`), so tool-based rg was preferred.

## Hash / integrity
```bash
cd /workspace/zcode-privacy-telemetry-audit
find . -type f -not -name SHA256SUMS.txt -exec sha256sum {} \; | sort -k2 > SHA256SUMS.txt
sha256sum zcode-privacy-telemetry-audit.zip
cd /workspace/zcode-privacy-telemetry-audit
zip -r -q ../zcode-privacy-telemetry-audit.zip . 
# zip created one directory level above to keep unpack top-level = README.md (see README)
```

## Verification recipe (for reviewers)
```bash
sha256sum -c SHA256SUMS.txt
# locate upstream original: https://github.com/zai-org/ZCode/tree/872ad960de7ec172591f7e1952f7849229f94521 + <original path>
# re-check a line range by reading the same path at the pinned commit.
```

No real credentials or secrets were recorded.