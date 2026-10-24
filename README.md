# Linux RAID1 Failure Simulation & Recovery Lab

**Date:** 24 October 2026 (WIB) · **Platform:** Ubuntu Linux on Oracle VirtualBox · **Tools:** mdadm, lsblk, ext4, sha256sum

> **Scope and authenticity:** This is an **independent virtual-machine lab** using Linux **software RAID1**. The failure was deliberately injected with `mdadm --fail`. It is **not** a physical drive replacement, hardware RAID procedure, production incident, or work on production infrastructure.

## Project at a glance

| Test | Observed result |
|---|---|
| Create two-member RAID1 | `/dev/md0` with `/dev/sdb` + `/dev/sdc` — `[2/2] [UU]` |
| Establish test-data baseline | 10 MiB file and saved SHA-256 manifest; verification `OK` |
| Simulate a member failure | `/dev/sdb` flagged faulty; array became `clean, degraded` — `[2/1] [_U]` |
| Replace failed virtual member | Removed `/dev/sdb`, added unused `/dev/sdd` |
| Verify recovery | `clean`; `/dev/sdc` + `/dev/sdd` active; `[2/2] [UU]`; 0 failed |
| Check sample-file integrity | Baseline manifest verification after recovery: **`OK`** |
| Confirm usable filesystem | `/dev/md0` formatted ext4, mounted at `/mnt/raid1-lab` |

**Outcome:** RAID1 redundancy was restored in the VM and the tested file still matched its pre-failure checksum. This establishes **sample-file integrity**, not validation of every filesystem block or proof of physical-disk reliability.

## Real screenshots from the lab

**1. RAID recovery and checksum verification**

![Original Ubuntu terminal screenshot showing RAID1 clean with 2 active devices and SHA-256 OK](evidence/03-recovered-raid1.png)

**2. Mounted ext4 filesystem and SHA-256 verification**

![Original Ubuntu terminal screenshot showing the ext4 mount on md0 and test file checksum OK](evidence/04-filesystem-sha256-verification.png)

See the [evidence inventory](evidence/README.md) for provenance and a distinction between original screenshots and transcribed earlier checkpoints.

## Environment and storage layout

| Component | Lab configuration |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Guest OS | Ubuntu 26.04 LTS (as reported during the exercise) |
| Hostname | `ubuntu-bootcamp` |
| mdadm | Version 4.5; RAID metadata 1.2 |
| OS disk | `/dev/sda`, ~30 GiB; not used as a RAID member |
| Original RAID members | `/dev/sdb` and `/dev/sdc`, separate ~1 GiB virtual disks |
| Replacement | `/dev/sdd`, a third ~1 GiB virtual disk |
| Array | `/dev/md0` (RAID1) |
| Filesystem | ext4 mounted at `/mnt/raid1-lab` |
| Integrity sample | `raid1-test.bin`, 10 MiB, created on the array |

### Logical RAID topology

~~~text
INITIAL (HEALTHY)           FAILURE (DEGRADED)        RECOVERED (HEALTHY)
 /dev/md0 [UU]               /dev/md0 [_U]             /dev/md0 [UU]
  /dev/sdb (active)           /dev/sdb (faulty)         /dev/sdd (active)
  /dev/sdc (active)           /dev/sdc (active)         /dev/sdc (active)
~~~

Device identifiers describe this specific VM, not universal Linux disk identities.

## What was done

### Stage 1 — Baseline and filesystem

Inspected the virtual disks with `lsblk`, then created the software RAID1 array and checked the initial health:

~~~text
md0 : active raid1 sdc[1] sdb[0]
      1057856 blocks super 1.2 [2/2] [UU]
~~~

Formatted only `/dev/md0` as ext4, mounted it, and created a 10 MiB test file. The SHA-256 manifest was saved in `/home/rafi/raid1-before.sha256`, outside the test array.

### Stage 2 — Controlled failure and diagnosis

Deliberately marked `/dev/sdb` faulty using `mdadm`:

~~~text
md0 : active raid1 sdc[1] sdb[0](F)
      1057856 blocks super 1.2 [2/1] [_U]
~~~

`mdadm --detail` showed `clean, degraded`, one active device and one failed device. The remaining `/dev/sdc` member was intentionally preserved. At this point, the array had **no remaining mirror redundancy**.

### Stage 3 — Replacement and verification

Removed the failed `/dev/sdb` member, added the unused `/dev/sdd` member, and inspected the recovered array:

~~~text
md0 : active raid1 sdd[2] sdc[1]
      1057856 blocks super 1.2 [2/2] [UU]
~~~

Final detailed diagnostics confirmed `State: clean`, `Active Devices: 2`, and `Failed Devices: 0`. The saved hash manifest produced:

~~~text
/mnt/raid1-lab/raid1-test.bin: OK
~~~

The later evidence capture also reported an ext4 filesystem of 983M and 905M available after the 10 MiB test write.

## Evidence and technical documentation

| Resource | Contents |
|---|---|
| [Incident report](incident-report.md) | Detection, sequence of actions, resolution and limitations |
| [Command reference](commands.md) | Commands from the exercise, including destructive-operation warnings |
| [Evidence inventory](evidence/README.md) | Original screenshots, transcripts and missing evidence |
| [Terminal excerpts](evidence/observed-terminal-excerpts.md) | Earlier checkpoints transcribed from participant outputs |
| [Final verification output](logs/recovery-verification-final.txt) | Console text copied from Ubuntu and provided for publication |
| [Log provenance](logs/README.md) | What the uploaded log does and does not prove |

## Important limitations and lessons

- **RAID1 is availability/redundancy, not a backup.** Mirroring also propagates accidental file deletion and some data corruption.
- `[UU]` shows two active members in this Linux software RAID; it does not replace application tests, disk-health tests, or monitoring.
- SHA-256 `OK` checks **only** the 10 MiB test file against its saved baseline.
- The exact degraded-phase SHA-256 output was **not captured**; no claim of an independently verified degraded-phase checksum is made.
- No reboot/automatic RAID assembly, persistent mount configuration, SMART/BMC monitoring, controller diagnostics, physical drive swap or production change procedure was tested.
- The removed `/dev/sdb` disk can still appear as `linux_raid_member` in `lsblk` because RAID metadata can remain even after removal from the active array. The active array membership is verified through `/proc/mdstat` and `mdadm --detail`.

## Practical takeaways

This lab illustrates how to establish a storage baseline, recognize a degraded mirror, replace a simulated failed member, and verify both array health and sample-file integrity after recovery. Keeping the failure and recovery observations together also makes the procedure easier to review and reproduce.

Physical-system recovery has additional requirements beyond this experiment, such as component identification, hardware diagnostics, safe handling procedures, change control, rebuild monitoring, and service validation. These were not performed in this virtual lab.
