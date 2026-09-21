# Third-Party Notices

This audit package redistributes a **minimal subset** of source files from

- **Project:** ZCode
- **Repository:** https://github.com/zai-org/ZCode.git
- **Upstream owner:** zai-org (Z.ai)
- **Pinned commit:** `872ad960de7ec172591f7e1952f7849229f94521`

The files under `evidence/source/` and `upstream/` originate from that repository and are included **solely for security research, evidence, and independent re-review**. Their copyright remains with the original authors; we do **not** claim authorship or relicense them. See the copied `upstream/LICENSE` and `upstream/NOTICE.md`.

Audit finding text, reports, CSVs, call-chains and this package structure are original work of the audit author.

Any file containing a real credential/token that could not be included is documented as `REDACTED BY AUDIT`; in practice **no secrets were included** (see secrets scan below).

Rebuilders must honor the upstream license when creating derivatives from `evidence/source/*`.

Original upstream third-party notice (bundled deps) is at `zcode-upstream/THIRD-PARTY-NOTICES.md` in the source checkout; we do not redistribute bundled third-party binaries.