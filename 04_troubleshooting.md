# Troubleshooting Log

## Common Issues & Fixes

### 1. "Current boot/recovery have been destroyed"
- **Cause:** AVB (Android Verified Boot) check failed because the custom recovery was unsigned.
- **Fix:** Flash an empty/disabled VBMeta image.
  ```bash
  fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
  ```

### 2. "Volume Full" when flashing TWRP
- **Cause:** The generic Poco M3 TWRP image (134MB) was larger than the physical recovery partition (100MB).
- **Fix:** You used `mkbootimg` to repack the TWRP ramdisk with the device's Stock Kernel. This stripped out the large generic kernel and made the image ~53MB.

### 3. ADB Unauthorized / Offline in Recovery
- **Cause:** The ported recovery lacked the correct USB controller initialization in the kernel command line.
- **Fix:** Added `androidboot.usbcontroller=4e00000.dwc3` to the `mkbootimg` arguments during the porting process.

### 4. Parted "Partition is being used"
- **Cause:** Trying to delete `cache` while it was mounted.
- **Fix:** Force unmount via `adb shell umount -f /cache` before running the `rm` command.

### 5. Flashing Typo (Critical)
- **Incident:** Flashed `vbmeta_vendor.img` to `vbmeta_system` partition.
- **Consequence:** Would break boot chain for system verification.
- **Fix:** Immediately reflashed both `vbmeta_system` and `vbmeta_vendor` with their correct respective images.

### 6. "Failed to write partition table" in FastbootD
- **Cause:** Stock `super.img` contained metadata locked to 7.8GB, conflicting with the new 12GB physical size.
- **Fix:** Created new `super_empty.img` with `lpmake` defining 12GB size.
