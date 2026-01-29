# Partitioning Guide (The Surgery)

**WARNING:** This process wipes all user data and carries a high risk of bricking.

## Prerequisites
1.  **Booted into Custom Recovery** (Ported TWRP).
2.  **ADB Access** established.
3.  **Parted Binary** pushed to `/sbin/`.

## The Procedure (Executed)

### 1. Backup
We created `super_bd.img` from stock files and backed up the small partitions that act as obstacles between `super` and `userdata`.

```bash
adb pull /dev/block/sda7 cache.img
adb pull /dev/block/sda8 recovery.img
adb pull /dev/block/sda9 vbmeta_system.img
adb pull /dev/block/sda10 vbmeta_vendor.img
adb pull /dev/block/sda11 metadata.img
```

### 2. Analysis
We used `parted` to identify the sector boundaries.
- **Original Super:** 8712s - 1929735s (~7.8 GB)
- **Original Userdata:** 2073128s - 30658554s

We calculated a new layout to give `super` **12 GiB** (3145728 sectors).

### 3. The Resize Operation (Parted)
We ran a batch command in ADB Shell to delete partitions 6-12 and recreate them with shifted start/end sectors.

**New Layout:**
- **Super (6):** 8712s - 3154439s (12 GiB)
- **Cache (7):** Shifted to 3154440s
- **Recovery (8):** Shifted to 3269128s
- ... and so on.
- **Userdata (12):** Starts at 3297832s (Shrunk).

### 4. Restoration (Fastboot)
After partitioning, we rebooted to the Bootloader to reload the table and restore data.

```bash
# Repair the "Obstacles"
fastboot flash recovery recovery.img
fastboot flash cache cache.img
fastboot flash vbmeta_system vbmeta_system.img
fastboot flash vbmeta_vendor vbmeta_vendor.img
fastboot flash metadata metadata.img

# Format the moved Userdata
fastboot format userdata

# Flash the Super container
# (See 05_logical_partitioning.md for why flashing the stock super here is not enough)
fastboot flash super super_bd.img
```
