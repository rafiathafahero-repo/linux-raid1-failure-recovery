# Evidence Inventory — Linux RAID1 Failure & Recovery

| Evidence | Actual availability | Verified observation |
|---|---|---|
| Original healthy RAID1 checkpoint | Transcribed in [observed-terminal-excerpts.md](observed-terminal-excerpts.md) | `[2/2] [UU]`, /dev/sdb + /dev/sdc |
| Original file checksum | Transcribed in excerpts | 10 MiB test file; baseline SHA-256 check `OK` |
| ext4 mounted | Transcribed in excerpts | /dev/md0 mounted as ext4 at /mnt/raid1-lab |
| Simulated failure | Transcribed in excerpts | `[2/1] [_U]`, `clean, degraded`, failed /dev/sdb |
| Recovery and checksum | Transcribed in excerpts; **original screenshot available offline** | `[2/2] [UU]`, /dev/sdc + /dev/sdd, failed=0, checksum `OK` |
| Unedited terminal log | **Not collected** | Pending optional fresh log capture |

## Image files to upload

1. `03-recovered-raid1.png` — original final checkpoint screenshot, provided in the offline evidence ZIP.
2. `01-healthy-raid1.png` — original screenshot from the conversation, **only if recoverable**.
3. `02-degraded-raid1.png` — original screenshot from the conversation, **only if recoverable**.

The listed PNGs are **not currently in this GitHub folder**. Do not present the transcript excerpts as original machine-generated logs or create artificial images to fill gaps. Repeating failure injection solely for screenshots is unnecessary.

## Optional live log from Ubuntu

```bash
mkdir -p ~/aws-dco-evidence
{
  date -Is
  cat /proc/mdstat
  sudo mdadm --detail /dev/md0
  df -hT /mnt/raid1-lab
  sha256sum -c ~/raid1-before.sha256
} | tee ~/aws-dco-evidence/recovery-verification.txt
```

If you run this, upload the unedited resulting file to `logs/recovery-verification.txt`. These commands are observational, not changes to RAID membership.

**Privacy:** Check images and logs for secrets before making them public. The reviewed final screenshot exposes the local VM username, hostname and array UUID but no visible password or token.
