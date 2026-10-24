# Final Verification Console Output

## Available evidence

**[recovery-verification-final.txt](recovery-verification-final.txt)** contains Ubuntu terminal output supplied by the lab participant after recovery, showing:

- `/proc/mdstat`: `/dev/md0` RAID1, `[2/2] [UU]` with `/dev/sdc` and `/dev/sdd`.
- `lsblk`: OS disk `/dev/sda` and RAID filesystem on `/dev/md0`.
- `df -hT`: ext4 mounted at `/mnt/raid1-lab`.
- `sha256sum -c`: `/mnt/raid1-lab/raid1-test.bin: OK`.

## Provenance and completeness

The participant generated a local Ubuntu verification text file by appending command output with `tee -a`, then pasted its contents into the conversation. The same console text was committed here. The reproducible examples in [commands.md](../commands.md) use a generic output directory rather than reproducing the original directory name. This is **not a verified byte-for-byte transfer from the Ubuntu filesystem**, and it is not a comprehensive historical terminal audit log.

The final two filesystem/checksum pairs repeat because the same checks were appended again. They are retained rather than silently modifying the source material.

The copied log does **not** include `sudo mdadm --detail`; the array's detailed healthy and degraded status was instead demonstrated in earlier screenshots and captured in [the incident report](../incident-report.md). The copy also lacks a capture timestamp. Do not infer one from GitHub's commit time.

For status before and during failure, see [transcribed terminal excerpts](../evidence/observed-terminal-excerpts.md). For original final screenshots, see the [evidence folder](../evidence/README.md).
