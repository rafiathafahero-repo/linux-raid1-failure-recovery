# Incident Report — LAB-RAID-001

| Field | Value |
|---|---|
| Incident class | Planned storage-failure simulation |
| Environment | Ubuntu Linux in Oracle VirtualBox |
| Array | `/dev/md0` software RAID1 |
| Original members | `/dev/sdb` and `/dev/sdc` |
| Faulted member | `/dev/sdb` (deliberately marked faulty) |
| Replacement | `/dev/sdd` (dedicated 1 GiB virtual disk) |
| Date | 24 October 2026, WIB |
| Operational impact | Training VM only; no production workload |
| Status | **Recovered and verified** |

## Incident description

A controlled RAID1 failure was initiated to practice degraded-array detection, evidence-based troubleshooting, virtual member replacement and post-recovery verification. This was a **software status failure injection**, not physical disk damage.

## Timeline and evidence

| Stage | Observation |
|---|---|
| 07:00:48 | RAID metadata creation time in final `mdadm --detail`; original disks `/dev/sdb` and `/dev/sdc` |
| Initial state | `[2/2] [UU]`, both original members active |
| File baseline | ext4 mounted; 10 MiB file created; original SHA-256 check returned `OK` |
| 08:43:53 | RAID update time visible while `clean, degraded`, one active and one failed device |
| Recovery | Failed member removed, unused `/dev/sdd` added |
| 09:02:58 | RAID update time visible after recovery to `clean` |
| Final integrity check | `[2/2] [UU]` and `/mnt/raid1-lab/raid1-test.bin: OK` |

Times above were reported by the guest RAID metadata, **not a complete event-by-event audit trail**.

## Detection

~~~text
md0 : active raid1 sdc[1] sdb[0](F)
      1057856 blocks super 1.2 [2/1] [_U]
~~~

`mdadm --detail /dev/md0` corroborated:
- `State: clean, degraded`
- `Active Devices: 1`
- `Failed Devices: 1`

The surviving member was `/dev/sdc`; removing it would have placed data availability at risk.

## Recovery actions

1. Verified the degraded array and identity of the surviving member.
2. Used `mdadm --manage /dev/md0 --remove /dev/sdb` to remove the faulty member.
3. Used `mdadm --manage /dev/md0 --add /dev/sdd` to add the previously unused replacement disk.
4. Checked `/proc/mdstat` and `mdadm --detail` for clean two-member status.
5. Confirmed that the saved test-file checksum returned `OK`.

## Resolution

| Check | Observed result |
|---|---|
| RAID state | `clean` |
| Active / working / failed | 2 / 2 / 0 |
| Active members | `/dev/sdc` and `/dev/sdd` |
| RAID indicator | `[2/2] [UU]` |
| SHA-256 manifest validation | `/mnt/raid1-lab/raid1-test.bin: OK` |

**Outcome: PASS** for virtual software RAID1 recovery and integrity validation of the 10 MiB sample file.

## Remaining gaps

- Degraded-phase checksum output was not captured in evidence, even though the command was part of the requested workflow.
- No complete recovery-progress time series or independent reboot/automount persistence test.
- No real drive swap, BMC/controller diagnostic, SMART evaluation or production change management.
- No guarantee regarding the integrity of all RAID blocks, application availability or physical hardware reliability.

## Operational takeaway

In a real data center, an array showing only one healthy RAID1 member requires careful incident triage, asset identification, safety/change approvals, backup evaluation, physical part verification and post-repair validation. Those real-world procedures are discussed here, **not claimed as performed**.