# Troubleshooting Log

## Common Issues & Fixes

### 1. "Current boot/recovery have been destroyed"
- **Cause:** AVB (Android Verified Boot) check failed because the custom recovery was unsigned.
- **Fix:** Flash an empty/disabled VBMeta image to both `vbmeta` and `vbmeta_system`.
  ```bash
  fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
  ```

### 2. "Volume Full" when flashing TWRP
- **Cause:** The donor TWRP image is larger than the physical recovery partition (e.g., 100MB).
- **Fix:** Use `mkbootimg` to repack the TWRP ramdisk with the device's Stock Kernel. This usually strips out unnecessary drivers and reduces the size.

### 3. ADB Unauthorized / Offline in Recovery
- **Cause:** The ported recovery lacks the correct USB controller initialization in the kernel command line or the ramdisk `prop.default`.
- **Fix:** Add `androidboot.usbcontroller=4e00000.dwc3` (or the correct path for the SoC) to the `mkbootimg` arguments.

### 4. Parted "Partition is being used"
- **Cause:** Attempting to delete or resize a partition while it is mounted (e.g., `/cache`).
- **Fix:** Force unmount via `adb shell umount -f /path` before running `parted`.

### 5. "Failed to write partition table" in FastbootD
- **Cause:** Stock metadata headers are often hardcoded to a specific physical size (e.g., 7.8GB), causing conflicts when the physical partition is larger.
- **Fix:** Create a new `super_empty.img` with `lpmake` defining the new physical size (e.g., 12GB).
