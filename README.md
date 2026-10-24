# Linux Software RAID1 Failure & Recovery Lab

**Independent technical lab | Ubuntu Linux · Oracle VirtualBox · mdadm · ext4 · SHA-256**

> **Scope:** Controlled Linux **software RAID in a virtual machine**. This is not a physical server repair, a hardware RAID controller exercise, a production incident, or an AWS-operated environment.

## Overview

Configured a two-member RAID1 array on dedicated VirtualBox disks, formatted the resulting array as ext4, created a 10 MiB test file and confirmed its SHA-256 baseline. Intentionally marked one RAID member faulty, identified the degraded array, removed the faulty member and added a separate replacement virtual disk. Final diagnostics confirmed a clean two-member array and an unchanged test-file checksum.

## Learning objectives

- Distinguish the OS disk from dedicated training disks before storage changes.
- Create and inspect Linux software RAID1 with `mdadm`.
- Recognize healthy `[UU]` versus degraded `[_U]` states.
- Preserve the surviving array member and restore redundancy with a replacement disk.
- Collect meaningful verification evidence and document the limits of a simulation.

## Lab environment

| Component | Configuration |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Guest OS | Ubuntu 26.04 LTS (reported during the lab) |
| Host | `ubuntu-bootcamp` |
| RAID utility | `mdadm` 4.5 |
| Array | `/dev/md0`, RAID1, metadata v1.2 |
| Original members | `/dev/sdb` and `/dev/sdc` (1 GiB virtual disks) |
| Replacement disk | `/dev/sdd` (1 GiB virtual disk) |
| OS disk | `/dev/sda`, 30 GiB — explicitly **not modified** |
| Filesystem and mount | ext4 on `/dev/md0`, mounted at `/mnt/raid1-lab` |
| Test data | 10 MiB `raid1-test.bin` |

**Safety:** These device names applied only to this particular VM session. Before using similar commands elsewhere, verify device identity with `lsblk` and ensure the targets are expendable.

## Procedure and observed results

### 1. Healthy baseline

Created `/dev/md0` from `/dev/sdb` and `/dev/sdc`. The initial array status was:

~~~text
md0 : active raid1 sdc[1] sdb[0]
      1057856 blocks super 1.2 [2/2] [UU]
~~~

Formatted **only `/dev/md0`**, mounted it and confirmed `df -hT /mnt/raid1-lab` reported ext4 (983M filesystem; 915M available). Created a 10 MiB test file and verified a SHA-256 baseline stored **outside** the test array.

### 2. Controlled member failure

Ran `sudo mdadm --manage /dev/md0 --fail /dev/sdb` to mark one RAID member faulty. Observed:

~~~text
md0 : active raid1 sdc[1] sdb[0](F)
      1057856 blocks super 1.2 [2/1] [_U]
~~~

`mdadm --detail` showed `State: clean, degraded`, one active and one failed device. The sole active member was `/dev/sdc`. **Redundancy was unavailable in this state.**

The intermediate SHA-256 check while degraded was requested, but its output was not visible in the supplied evidence; no claim of independently verified degraded-state checksum is made.

### 3. Replacement and recovery

Removed the failed `/dev/sdb` member and added the unused `/dev/sdd` replacement. The final array status was:

~~~text
md0 : active raid1 sdd[2] sdc[1]
      1057856 blocks super 1.2 [2/2] [UU]
~~~

Final `mdadm --detail` reported `State: clean`, two active devices, zero failed devices, and both `/dev/sdc` and `/dev/sdd` in `active sync`. The test file was checked against the saved SHA-256 manifest:

~~~text
/mnt/raid1-lab/raid1-test.bin: OK
~~~

## Results and scope

| Verification | Result |
|---|---|
| Initial RAID1 two-member healthy state | Verified |
| Degraded state after deliberate member failure | Verified |
| Replacement disk joined; array returned to `[UU]` | Verified |
| Test-file SHA-256 check before failure and after recovery | Verified |
| SHA-256 check specifically while degraded | Not independently captured |
| Reboot assembly and automatic mounting | Not tested |
| Physical drive replacement or hardware RAID | Not tested |

The `[UU]` status indicates both RAID slots are active. The checksum demonstrates integrity **for the tested file**, not every block on the storage device.

## Documentation

- [Incident report](incident-report.md) — controlled incident, diagnosis, recovery and limitations.
- [Command reference](commands.md) — ordered commands with safety notes.
- [Evidence inventory](evidence/README.md) — verified observations and screenshot availability.
- [Logs](logs/README.md) — how to add original unedited terminal output.

## CV summary

> Configured and troubleshot a Linux software RAID1 array using dedicated VirtualBox disks; simulated member failure, performed virtual disk replacement and recovery, and validated final array health and test-file integrity using mdadm and SHA-256.

**Lab performed 24 October 2026 (WIB).** This documentation is based on terminal outputs and screenshots collected during the guided exercise, without representing virtual lab experience as production operations.