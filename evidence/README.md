# Evidence Inventory

## Observations verified during the lab

| Stage | Observed terminal evidence | Availability |
|---|---|---|
| Healthy baseline | `[2/2] [UU]`, original members `/dev/sdb` and `/dev/sdc` | Visible in conversation; screenshot not yet added to repository |
| Initial integrity | 10 MiB file; `sha256sum -c` returned `OK` | Visible in conversation; screenshot not yet added |
| Degraded state | `[2/1] [_U]`, `clean, degraded`, `/dev/sdb` marked faulty | Visible in conversation; screenshot not yet added |
| Recovered state | `[2/2] [UU]`, `clean`, active `/dev/sdc` + `/dev/sdd`, zero failed | Screenshot provided in conversation |
| Final file integrity | `raid1-test.bin: OK` | Same final screenshot |

### Screenshot note

The **original final screenshot** is retained in the offline documentation package. It may be added here as `03-recovered-raid1.png` if binary upload succeeds. Until then, do not treat an absent link as published evidence.

Other original screenshots can be exported from the conversation or VM and then added to this directory. Do not fabricate image evidence, raw logs or SHA-256 hash strings.

### Privacy

Before publishing additional assets, review for passwords, tokens, session cookies, system identifiers and unintended personal details. The already-reviewed lab screenshot contains a local username, hostname and RAID UUID but no visible credentials.