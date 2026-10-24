# Observed Terminal Excerpts — Earlier Checkpoints

**Provenance:** These are transcriptions from participant-provided Ubuntu terminal outputs and screenshots in the 24 October 2026 lab conversation. They are **not automatically captured raw logs**. For original final screenshots see [the evidence inventory](README.md); for copied final console output see [recovery-verification-final.txt](../logs/recovery-verification-final.txt).

## Healthy array: original members

~~~text
md0 : active raid1 sdc[1] sdb[0]
      1057856 blocks super 1.2 [2/2] [UU]
~~~

## ext4 mount: initial state

~~~text
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/md0       ext4  983M  280K  915M   1% /mnt/raid1-lab
~~~

## Sample file and SHA-256 baseline

~~~text
-rw-r--r-- 1 root root 10M Oct 24 08:14 /mnt/raid1-lab/raid1-test.bin
/mnt/raid1-lab/raid1-test.bin: OK
~~~

The participant subsequently supplied the original manifest contents:

~~~text
75ee7a5aa44e04519b4df6cbd5c12692af9ea036688da65f8ca227e4ae97e7f3  /mnt/raid1-lab/raid1-test.bin
~~~

That digest is included from the participant's actual text, **not independently recalculated by this document**.

## Controlled failure: degraded RAID1

~~~text
md0 : active raid1 sdc[1] sdb[0](F)
      1057856 blocks super 1.2 [2/1] [_U]

State           : clean, degraded
Active Devices  : 1
Working Devices : 1
Failed Devices  : 1
~~~

The SHA-256 result **during the degraded interval** was not visible in the preserved screenshot. The one active member at this stage was `/dev/sdc`.

## Recovery: replacement member

~~~text
md0 : active raid1 sdd[2] sdc[1]
      1057856 blocks super 1.2 [2/2] [UU]

State           : clean
Active Devices  : 2
Working Devices : 2
Failed Devices  : 0

/mnt/raid1-lab/raid1-test.bin: OK
~~~

Original image: [03-recovered-raid1.png](03-recovered-raid1.png).

## Additional final evidence

The [second original screenshot](04-filesystem-sha256-verification.png) shows the mounted ext4 filesystem with final SHA-256 `OK`; the [console output](../logs/recovery-verification-final.txt) also records the two active members and the mounted filesystem.

**Not tested:** reboot persistence, physical drive replacement, whole-array data integrity, production availability or hardware failure diagnostics.
