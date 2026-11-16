# Case note: Rootcheck "trojaned file" alert on /usr/bin/md5sum

**Date:** 2026-11-16 (NZDT)
**Analyst:** Navi
**Verdict:** False positive (benign)
**Severity (Wazuh):** Level 7 (medium)
**Status:** Closed, no action needed on the host. Rule tuning recommended.

## Summary (BLUF)
Wazuh rootcheck flagged `/usr/bin/md5sum` on `ubuntu-server` as a trojaned file. I verified the file against Ubuntu's package checksums and it is unmodified. The alert is a false positive caused by a generic rootcheck signature matching strings in a normal binary.

## The Five Ws
- **Who:** Agent `001` (`ubuntu-server`, `10.10.10.10`), reported to `wazuh-manager`. Rule `510`, "Host-based anomaly detection event (rootcheck)".
- **What:** Alert title "Trojaned version of file detected" for `/usr/bin/md5sum`. Signature used: `bash|^/bin/sh|file\.h|proc\.h|/dev/|^/bin/.*sh` (Generic).
- **When:** Latest alert seen 2026-11-16 17:33:30 NZDT. The rule had fired 78 times at that point (`rule.firedtimes`). I did not record the first-seen time.
- **Where:** File `/usr/bin/md5sum` on the Ubuntu target VM in the isolated lab network (10.10.10.0/24).
- **Why (assessment):** The signature flags any file containing certain common strings such as `/dev/` or `bash`. The file belongs to the package `coreutils-from-uutils` (the Rust rewrite of the core utilities), so it is expected software and likely to contain such strings.

## Evidence (facts)
1. Wazuh alert fields: `rule.id: 510`, `rule.level: 7`, `data.file: /usr/bin/md5sum`, `decoder.name: rootcheck`, `rule.firedtimes: 78`.
2. `dpkg -S /usr/bin/md5sum` returned `coreutils-from-uutils: /usr/bin/md5sum`, so the file is owned by an installed Ubuntu package.
3. `dpkg -V coreutils-from-uutils` returned no output, meaning all files in the package match the stored checksums (nothing modified).

## Judgment (inference)
- The generic signature is matching strings inside a legitimate binary. This is my conclusion from the evidence above, not something Wazuh stated.

## Recommended action
- Close the alert as a false positive.
- Tune or suppress rootcheck trojan-signature detections for files owned by trusted packages, so this alert stops repeating.
- Still to check: whether other files in `/usr/bin` trigger rule 510 (search `rule.id:510` and review the `data.file` column).

## What I learned
- An alert firing is not the same as a real compromise. Verify with a trusted source (package checksums) before closing or escalating.
- Keep facts and judgment separate in the note.
