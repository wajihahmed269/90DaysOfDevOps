# Day 13 – Linux Volume Management (LVM)

## Goal

Learn how **LVM (Logical Volume Manager)** works in Linux and how it helps manage storage more flexibly.

LVM allows you to:
- Combine multiple disks
- Resize storage easily
- Manage logical volumes without affecting applications

---

# Step 1 – Check Current Storage

Commands used:

```bash
lsblk
pvs
vgs
lvs
df -h
```

Purpose:

- `lsblk` → shows block devices
- `pvs` → shows physical volumes
- `vgs` → shows volume groups
- `lvs` → shows logical volumes
- `df -h` → shows mounted filesystems and usage

---

# Step 2 – Create a Virtual Disk (if no spare disk)

```bash
dd if=/dev/zero of=/tmp/disk1.img bs=1M count=1024
losetup -fP /tmp/disk1.img
losetup -a
```

Explanation:

- `dd` creates a **1GB virtual disk file**
- `losetup` attaches the file as a **loop device**
- `losetup -a` displays the loop device name (example: `/dev/loop0`)

---

# Step 3 – Create Physical Volume

```bash
pvcreate /dev/loop0
pvs
```

Explanation:

- `pvcreate` converts a disk into an **LVM physical volume**
- `pvs` confirms the physical volume creation

Example output:

```
PV         VG   Fmt  Attr PSize  PFree
/dev/loop0      lvm2 ---  1.00g  1.00g
```

---

# Step 4 – Create Volume Group

```bash
vgcreate devops-vg /dev/loop0
vgs
```

Explanation:

- `vgcreate` creates a **Volume Group**
- Volume groups combine one or more physical volumes.

Example output:

```
VG        #PV #LV #SN Attr   VSize  VFree
devops-vg   1   0   0 wz--n- 1.00g 1.00g
```

---

# Step 5 – Create Logical Volume

```bash
lvcreate -L 500M -n app-data devops-vg
lvs
```

Explanation:

- `-L 500M` → size of logical volume
- `-n app-data` → name of logical volume
- `devops-vg` → volume group

Example output:

```
LV        VG        Attr       LSize
app-data  devops-vg -wi-a----- 500.00m
```

---

# Step 6 – Format and Mount

Format the logical volume:

```bash
mkfs.ext4 /dev/devops-vg/app-data
```

Create mount directory:

```bash
mkdir -p /mnt/app-data
```

Mount the volume:

```bash
mount /dev/devops-vg/app-data /mnt/app-data
```

Verify:

```bash
df -h /mnt/app-data
```

Example output:

```
Filesystem                    Size Used Avail Use%
/dev/mapper/devops--vg-app--data  488M  1M  452M   1% /mnt/app-data
```

---

# Step 7 – Extend Logical Volume

Extend logical volume:

```bash
lvextend -L +200M /dev/devops-vg/app-data
```

Resize filesystem:

```bash
resize2fs /dev/devops-vg/app-data
```

Verify new size:

```bash
df -h /mnt/app-data
```

Now the logical volume should be **700MB instead of 500MB**.

---

# Commands Used

```
lsblk
pvs
vgs
lvs
df -h
dd
losetup
pvcreate
vgcreate
lvcreate
mkfs.ext4
mount
lvextend
resize2fs
```

---

# Screenshots

Add screenshots of:

- `lsblk`
- `pvs`
- `vgs`
- `lvs`
- `df -h /mnt/app-data`
- `lvextend` result

---

# What I Learned

1. **LVM allows flexible storage management** compared to traditional disk partitions.
2. Storage is organized into **Physical Volumes → Volume Groups → Logical Volumes**.
3. Logical volumes can be **extended without deleting or recreating partitions**, which is very useful for production servers.

---

# Real DevOps Use Case

LVM is commonly used for:

- Extending disk space in **production servers**
- Managing storage in **cloud virtual machines**
- Expanding **log directories, databases, and application storage**

---

# DevOps Progress

Day 13 completed as part of the **#90DaysOfDevOps challenge** 🚀
