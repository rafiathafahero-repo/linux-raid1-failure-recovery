# Command Reference — Linux Software RAID1 Lab

> **Warning:** This is a record of a controlled VM training exercise, **not** a production runbook or a safe copy-paste script. Device names below were verified for one specific VirtualBox guest only. Verify all target devices and backup/authorization status before storage writes. **Never run the destructive steps on an OS or valuable data disk.**

## 1 — Inspect the dedicated virtual disks

~~~bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo wipefs -n /dev/sdb /dev/sdc /dev/sdd
sudo mdadm --version
~~~

`wipefs -n` only lists signatures and does not erase them. `/dev/sda` was the guest's OS disk and was excluded.

## 2 — Build RAID1 and mount ext4

~~~bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sudo mkfs.ext4 -L RAID1_LAB /dev/md0
sudo mkdir -p /mnt/raid1-lab
sudo mount /dev/md0 /mnt/raid1-lab
df -hT /mnt/raid1-lab
~~~

`mdadm --create` modifies RAID-member metadata, and `mkfs.ext4` formats the array device. The initial run used the default **No** for the optional write-intent bitmap prompt.

## 3 — Create sample data and save its digest separately

~~~bash
sudo dd if=/dev/urandom of=/mnt/raid1-lab/raid1-test.bin bs=1M count=10 status=progress
ls -lh /mnt/raid1-lab/raid1-test.bin
sha256sum /mnt/raid1-lab/raid1-test.bin | tee ~/raid1-before.sha256
sha256sum -c ~/raid1-before.sha256
~~~

`dd` wrote a **regular file on the mounted array**, not raw member disks. The manifest was stored at `~/raid1-before.sha256` on the guest's OS filesystem.

## 4 — Deliberately fail one RAID member

~~~bash
sudo mdadm --manage /dev/md0 --fail /dev/sdb
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sha256sum -c ~/raid1-before.sha256
~~~

This failure-injection command changes the array's state. The degraded `[_U]` result is verified; the output of the SHA-256 check specifically at that moment is **not captured**.

## 5 — Remove the faulty member and insert the spare

~~~bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo wipefs -n /dev/sdd
sudo mdadm --manage /dev/md0 --remove /dev/sdb
sudo mdadm --manage /dev/md0 --add /dev/sdd
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sha256sum -c ~/raid1-before.sha256
~~~

**Critical safety rule:** `/dev/sdc` was the *sole remaining good member* during the degraded interval and was not removed. `/dev/sdd` was the pre-checked expendable replacement disk. Rebuilding writes RAID metadata/data to the new member.

## 6 — Final evidence capture (equivalent commands)

The lab's final verification output was collected in a local text file using `tee`. The commands below illustrate the same capture method with a **generic evidence directory**; they are not presented as a verbatim export of the original shell history:

~~~bash
mkdir -p ~/raid1-evidence
cat /proc/mdstat | tee ~/raid1-evidence/recovery-verification-final.txt
~~~

Additional checks can be appended:

~~~bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS | tee -a ~/raid1-evidence/recovery-verification-final.txt
df -hT /mnt/raid1-lab | tee -a ~/raid1-evidence/recovery-verification-final.txt
sha256sum -c ~/raid1-before.sha256 | tee -a ~/raid1-evidence/recovery-verification-final.txt
~~~

`tee -a` **appends**, so re-running commands resulted in duplicate final lines. The [uploaded text](logs/recovery-verification-final.txt) is the participant-provided console text copied into GitHub from the conversation. No root privileges were needed to collect those read-only checks.

### Cleaner pattern for a future run (not claimed as executed)

~~~bash
{
  date -Is
  cat /proc/mdstat
  lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
  findmnt /mnt/raid1-lab
  df -hT /mnt/raid1-lab
  sha256sum -c ~/raid1-before.sha256
} > ~/raid1-evidence/read-only-status-example.txt
~~~

This **example** would produce a newly captured read-only status file; it is not a replacement for the historical output.

## Beyond scope

The exercise did not test persistent assembly across reboot (`mdadm.conf`/initramfs), `/etc/fstab` automount configuration, physical storage controller tools or hardware replacement.
