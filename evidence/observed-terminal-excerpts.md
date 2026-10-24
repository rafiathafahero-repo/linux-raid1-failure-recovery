# Observed Terminal Excerpts (Transcribed, Not Raw Logs)

**Provenance:** Manually transcribed from the user's terminal outputs and screenshots shared during the 24 October 2026 training lab. These excerpts are **not** an original terminal log or fabricated screenshots.

## Healthy RAID1 baseline

```text
md0 : active raid1 sdc[1] sdb[0]
      1057856 blocks super 1.2 [2/2] [UU]
```

## Original 10 MiB test file and checksum

```text
-rw-r--r-- 1 root root 10M Oct 24 08:14 /mnt/raid1-lab/raid1-test.bin
/mnt/raid1-lab/raid1-test.bin: OK
```

The original SHA-256 digest is not reproduced because the 64-character hash was not visible in the provided checkpoint.

## ext4 mount

```text
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/md0       ext4  983M  280K  915M   1% /mnt/raid1-lab
```

## Simulated degraded RAID1

```text
md0 : active raid1 sdc[1] sdb[0](F)
      1057856 blocks super 1.2 [2/1] [_U]

State           : clean, degraded
Active Devices  : 1
Working Devices : 1
Failed Devices  : 1
```

A separate checksum **while degraded** was not visible in the screenshot. The sole active member was /dev/sdc.

## Recovered RAID1 and final checksum

```text
md0 : active raid1 sdd[2] sdc[1]
      1057856 blocks super 1.2 [2/2] [UU]

State           : clean
Active Devices  : 2
Working Devices : 2
Failed Devices  : 0

/mnt/raid1-lab/raid1-test.bin: OK
```

The **original screenshot** of this final verification is available in the offline evidence bundle, but is not yet uploaded as a PNG to this repository.

## Evidence limitations

- Original screenshot files for the initial healthy and degraded states have not yet been collected into the repository.
- An original, unedited Ubuntu terminal log has not yet been provided.
- No physical disk replacement, production incident, reboot persistence or complete-volume checksum was performed or claimed.
