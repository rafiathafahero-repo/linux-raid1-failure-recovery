# Simulated Storage Incident Report — LAB-RAID-001

| Field | Value |
|---|---|
| Date | 24 October 2026, WIB |
| Classification | **Planned failure-injection exercise** (not an unplanned production incident) |
| Environment | Ubuntu 26.04 LTS guest on Oracle VirtualBox |
| Storage | Linux `mdadm` software RAID1 array `/dev/md0` |
| Initial members | `/dev/sdb` and `/dev/sdc` |
| Intentionally faulted member | `/dev/sdb` |
| Surviving member during fault | `/dev/sdc` |
| Virtual replacement | `/dev/sdd` |
| Outcome | **Recovered** — clean, two active members, sample-file checksum `OK` |
| Production impact | None; isolated training VM with no claimed production workload |

## Incident overview

The exercise intentionally changed one RAID1 member to *faulty* with `mdadm --fail` to observe a degraded software mirror and recover redundancy with an unused virtual disk. No physical failure was diagnosed and no real drive was replaced. This is a **simulated member failure**, not a real hardware incident.

## Timeline (observed, not a complete audit trail)

| Stage / guest metadata time | Observation | Supporting evidence |
|---|---|---|
| Initial, metadata creation time 07:00:48 | Two healthy members, `[2/2] [UU]` | [Earlier terminal excerpt](evidence/observed-terminal-excerpts.md) |
| Baseline before injection | Mounted ext4; test file created (10 MiB); saved SHA-256 manifest verification `OK` | [Terminal excerpt and manifest](evidence/observed-terminal-excerpts.md) |
| Degraded condition (metadata updated 08:43:53) | Faulty `/dev/sdb`; one active member; `clean, degraded`, `[2/1] [_U]` | [Degraded excerpt](evidence/observed-terminal-excerpts.md) |
| Recovery action | Faulty member removed; unused `/dev/sdd` inserted into array | [Command record](commands.md) |
| Recovered condition (metadata updated 09:02:58) | Clean, two active members, zero failed, `[2/2] [UU]` | [Original recovery screenshot](evidence/03-recovered-raid1.png) |
| Final verification | ext4 mounted and 10 MiB sample file matches original manifest | [Filesystem screenshot](evidence/04-filesystem-sha256-verification.png) · [Console output](logs/recovery-verification-final.txt) |

The times shown are from `mdadm` *Creation Time* and *Update Time* output; **they are not exact timestamps for every action**.

## Detection and technical analysis

~~~text
md0 : active raid1 sdc[1] sdb[0](F)
      1057856 blocks super 1.2 [2/1] [_U]
~~~

The `mdadm --detail` diagnostic at that stage showed `State: clean, degraded`, one working/active member and one failed. The array still had a surviving `/dev/sdc` member, but **mirror redundancy was lost**. Removing the surviving member would risk losing access to the data.

**Root cause of the simulated event:** deliberate use of `mdadm --manage /dev/md0 --fail /dev/sdb`. No actual hardware-failure root cause was investigated.

## Corrective action within the lab

1. Confirmed the array was degraded and identified `/dev/sdc` as the still-active member.
2. Checked that `/dev/sdd` was the dedicated unused replacement virtual disk.
3. Removed the already-faulty `/dev/sdb` from the software array.
4. Added `/dev/sdd` to the array and checked recovery completion.
5. Verified the final array state and test-file checksum against the pre-failure baseline.

## Resolution and verification

~~~text
md0 : active raid1 sdd[2] sdc[1]
      1057856 blocks super 1.2 [2/2] [UU]
~~~

| Test | Final observed result |
|---|---|
| `mdadm` state | `clean` |
| Active / working / failed | 2 / 2 / 0 |
| Active RAID members | `/dev/sdc`, `/dev/sdd` |
| Filesystem | ext4 on `/dev/md0`, mounted `/mnt/raid1-lab` |
| 10 MiB test file integrity | `/mnt/raid1-lab/raid1-test.bin: OK` |

![Original screenshot of clean recovered RAID1 and SHA-256 OK](evidence/03-recovered-raid1.png)

## What remains unverified

- There is no captured result of the checksum check **specifically during the degraded interval**, although the command was requested in the exercise.
- Neither a complete resync-progress trace nor measured application availability was collected.
- No reboot, persistent mount, automatic array assembly, backup restore or full-filesystem integrity check was tested.
- No physical drive, controller, SMART/BMC diagnostic or production ticket/change-management process was involved.

## Operational reflection (conceptual, not performed)

Real data center operations would additionally require a service-impact assessment, incident ticket, asset/slot verification, safety procedures, backup checks, approval/escalation paths, correct replacement part, monitoring of the rebuild and validation of the repaired service. RAID1 does not replace backups.

**Lab conclusion:** The controlled software RAID1 member failure and virtual replacement were successful; the rebuilt array was healthy and the specific 10 MiB file retained its original SHA-256 integrity.
