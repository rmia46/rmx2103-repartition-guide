# Partitioning Guide (The Surgery)

**WARNING:** This process wipes all user data and carries a high risk of bricking.

## Prerequisites
1.  **Booted into Custom Recovery** (Ported TWRP).
2.  **ADB Access** established.
3.  **Parted Binary** pushed to `/sbin/`.

## The Procedure

### 1. Backup
Ensure all critical partitions are backed up before proceeding.

```bash
adb pull /dev/block/sda7 cache.img
adb pull /dev/block/sda8 recovery.img
adb pull /dev/block/sda9 vbmeta_system.img
adb pull /dev/block/sda10 vbmeta_vendor.img
adb pull /dev/block/sda11 metadata.img
```

### 2. Analysis
Use `parted` to identify the sector boundaries.
- **Example Super:** 8712s - 1929735s (~7.8 GB)
- **Example Userdata:** 2073128s - 30658554s

Calculate the new layout to increase the `super` partition size (e.g., to 12 GiB).

### 3. The Resize Operation (Parted)
Run a batch command in ADB Shell to delete partitions and recreate them with shifted sectors.

**Example New Layout:**
- **Super (6):** 8712s - 3154439s (12 GiB)
- **Cache (7):** Shifted accordingly
- **Recovery (8):** Shifted accordingly
- ... and so on.

### 4. Restoration (Fastboot)
After partitioning, reboot to the Bootloader to reload the table and restore the data.

```bash
# Flash the "Obstacles" back to new sectors
fastboot flash recovery recovery.img
fastboot flash cache cache.img
fastboot flash vbmeta_system vbmeta_system.img
fastboot flash vbmeta_vendor vbmeta_vendor.img
fastboot flash metadata metadata.img

# Format the moved Userdata
fastboot format userdata

# Flash the Super container
# Note: Flashing a stock super.img will reset metadata to stock size.
# See 05_logical_partitioning.md for the metadata hack.
```
