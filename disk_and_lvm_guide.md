# Disk Management and LVM Guide

## Adding or Attaching a Volume

### Step 1: Attach the volume
- **Cloud platforms** (AWS, Azure, GCP): attach a new volume to your instance.
- **Physical or virtual servers** (Proxmox, VMware): create and attach a virtual disk.

Check if the system detects it:
```bash
lsblk
```
Sample output:
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda   202:0    0   50G  0 disk
└─xvda1
xvdb   202:16   0  100G  0 disk  # <-- new volume
```

### Step 2: Partition and format the disk
```bash
sudo fdisk /dev/xvdb
```
- Create a new partition (`n`) and write (`w`).
- Format the partition, e.g., ext4:
```bash
sudo mkfs.ext4 /dev/xvdb1
```

### Step 3: Decide mount point
| Use Case | Recommended Mount Point | Notes |
|----------|-----------------------|-------|
| Logs/temporary data | /var/log, /var/tmp | Prevents filling / |
| Website/app data | /var/www, /srv, /data | Keeps data separate from OS |
| User files | /home | Common for multi-user systems |
| General storage | /mnt/data or /data | Safe, flexible choice |
| Docker volumes | /var/lib/docker | For container storage |

**Best practice:** Do not mount new volumes directly on `/`.

### Step 4: Mount the volume
```bash
sudo mkdir /data
sudo mount /dev/xvdb1 /data
```

**Make permanent after reboot:**
```bash
sudo blkid
sudo nano /etc/fstab
```
Add:
```
UUID=<your-uuid>  /data  ext4  defaults,nofail  0  2
```
Test:
```bash
sudo mount -a
```

## Unmounting a Disk

### Step 1: Check mounted disks
```bash
lsblk
df -h
```
Example:
```
/dev/xvdb1           100G  0 part /data
```

### Step 2: Unmount safely
```bash
sudo umount /data
sudo umount /dev/xvdb1
```
- If "target is busy":
```bash
sudo lsof +f -- /data
```
- Stop processes using the disk and retry.

### Step 3: Remove from fstab (optional)
```bash
sudo nano /etc/fstab
```
- Comment out or remove the line for this mount point.

## LVM (Logical Volume Manager)

### What is LVM?
- A storage management layer in Linux between physical disks and file systems.
- Provides flexible storage management compared to traditional partitioning.

### How LVM Works
| Level | Name | Description |
|-------|------|-------------|
| 1️⃣ | Physical Volume (PV) | Actual disk or partition (`/dev/sdb1`) |
| 2️⃣ | Volume Group (VG) | Pool of storage combining PVs |
| 3️⃣ | Logical Volume (LV) | Virtual partitions created from VG |

### Step 3: Create LVM Physical Volume
```bash
sudo pvcreate /dev/xvdb
sudo pvs
```

### Step 4: Create Volume Group
```bash
sudo vgcreate vg_data /dev/xvdb
sudo vgs
```

### Step 5: Create Logical Volume
```bash
sudo lvcreate -L 100G -n lv_storage vg_data
sudo lvs
```

### Step 6: Format the LV
```bash
sudo mkfs.ext4 /dev/vg_data/lv_storage
```

### Step 7: Mount the LV
```bash
sudo mkdir /mnt/storage
sudo mount /dev/vg_data/lv_storage /mnt/storage
df -h
```

### Step 8: Make Permanent
```bash
sudo nano /etc/fstab
```
Add:
```
/dev/vg_data/lv_storage  /mnt/storage  ext4  defaults  0  2
```
Test:
```bash
sudo mount -a
```
### Extend the existing Volume Group

Assuming your VG is named vg_data:
```bash
sudo vgextend vg_data /dev/xvdc


Check VG:

sudo vgs
```
Now your VG has additional 50 GB free for creating new LVs or extending existing ones.

### Optional — Extend an existing Logical Volume

If you want to grow an LV (e.g., lv_storage) using the new space:

```bash
sudo lvextend -L +50G /dev/vg_data/lv_storage
```

Or grow to use all free space in VG:

```bash
sudo lvextend -l +100%FREE /dev/vg_data/lv_storage
```
### Resize the filesystem

After extending the LV, resize the filesystem (for ext4):

```bash
sudo resize2fs /dev/vg_data/lv_storage
```
### Why Mount LVM?
- LV is raw storage; mounting attaches it to the filesystem.
- Mount point is where Linux accesses the LV.
- Example:
```bash
sudo mkdir /mnt/storage
sudo mount /dev/vg_data/lv_storage /mnt/storage
cd /mnt/storage
touch testfile
ls
```

**Best Practice:** Do not mount LVM directly on `/` (root). Keep root for OS files only to avoid system issues and allow easy expansion.

