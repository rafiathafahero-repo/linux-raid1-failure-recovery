# Command Reference — Linux Software RAID1

> **Training reference only. Not a production runbook or a safe universal script.** The device names below refer to an isolated VM and must be independently verified before any write operation. In this lab `/dev/sda` was the OS disk and was never a RAID target.

## 1. Inspect disks

~~~bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo wipefs -n /dev/sdb /dev/sdc /dev/sdd
sudo mdadm --version
~~~

`wipefs -n` performs a read-only signature check.

## 2. Create and mount the array

~~~bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sudo mkfs.ext4 -L RAID1_LAB /dev/md0
sudo mkdir -p /mnt/raid1-lab
sudo mount /dev/md0 /mnt/raid1-lab
df -hT /mnt/raid1-lab
~~~

**Warning:** `mkfs.ext4` formats the target. Only the freshly created `/dev/md0` was formatted; member disks and the OS disk were not formatted.

## 3. Write and verify sample data

~~~bash
sudo dd if=/dev/urandom of=/mnt/raid1-lab/raid1-test.bin bs=1M count=10 status=progress
ls -lh /mnt/raid1-lab/raid1-test.bin
sha256sum /mnt/raid1-lab/raid1-test.bin | tee ~/raid1-before.sha256
sha256sum -c ~/raid1-before.sha256
~~~

The SHA-256 baseline manifest was saved to the Ubuntu home directory, outside the test array.

## 4. Simulate failure

~~~bash
sudo mdadm --manage /dev/md0 --fail /dev/sdb
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sha256sum -c ~/raid1-before.sha256
~~~

The degraded-state checksum command was prescribed, but its specific output is not independently captured in the provided screenshots.

## 5. Replace and recover

~~~bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo wipefs -n /dev/sdd
sudo mdadm --manage /dev/md0 --remove /dev/sdb
sudo mdadm --manage /dev/md0 --add /dev/sdd
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sha256sum -c ~/raid1-before.sha256
~~~

`/dev/sdc` was the **sole surviving member** during degradation and was not removed. Adding `/dev/sdd` changes RAID metadata on that virtual disk.

## 6. Optional unedited final log collection

~~~bash
mkdir -p ~/aws-dco-portfolio/01-linux-raid1
{
  date -Is
  cat /proc/mdstat
  sudo mdadm --detail /dev/md0
  df -hT /mnt/raid1-lab
  sha256sum -c ~/raid1-before.sha256
} | tee ~/aws-dco-portfolio/01-linux-raid1/recovery-verification.txt
~~~

This evidence-collection block was **recommended**; its successful execution has not been confirmed. If executed, transfer the actual file into the `logs/` folder. Do not create artificial raw logs or timestamps.

## Persistence not tested

The exercise did not confirm `/etc/mdadm/mdadm.conf` configuration, `/etc/fstab` entries, initramfs assembly, or behavior after reboot.