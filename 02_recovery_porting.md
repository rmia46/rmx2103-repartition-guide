# Recovery Porting Guide: Poco M3 to Realme RMX2103

## Theory
Recoveries (like TWRP) consist of a **Kernel** (hardware drivers) and a **Ramdisk** (the actual program/UI).
- **Poco M3 (Citrus):** Same SoC (Snapdragon 662) as our device.
- **Realme RMX2103:** Our target device.

By swapping the kernel, we trick the Poco M3 software into running on the Realme hardware.

## Steps Taken

### 1. Preparation
We extracted the `recovery.img` from the stock firmware and downloaded a `twrp-poco-m3.img`.

### 2. Unpacking
We used `unpackbootimg` to separate the components of both images.

```bash
unpackbootimg -i stock_recovery.img -o stock_out
unpackbootimg -i donor_twrp.img -o donor_out
```

### 3. Repacking (The "Frankenstein" Command)
We used `mkbootimg` to combine:
- **Kernel:** From Stock
- **DTB/DTBO:** From Stock (Critical for screen/hardware mapping)
- **Ramdisk:** From Donor (Poco M3)
- **Cmdline:** Used Stock cmdline + `androidboot.selinux=permissive` (to bypass security checks) + `androidboot.usbcontroller=4e00000.dwc3` (to fix ADB).

**The Command:**
```bash
mkbootimg --kernel stock_out/kernel \
    --ramdisk donor_out/ramdisk \
    --dtb stock_out/dtb \
    --recovery_dtbo stock_out/recovery_dtbo \
    --cmdline "..." \
    -o ported_recovery.img
```

### 4. Flashing
Since `fastboot boot` is disabled on Realme devices, we had to flash it directly:
```bash
fastboot flash recovery ported_recovery.img
fastboot reboot recovery
```

### 5. Troubleshooting
- **Issue 1:** "Current boot destroyed" -> Fixed by checking AVB/VBMeta.
- **Issue 2:** "Volume Full" -> The raw Poco M3 image was too big (134MB) for the partition (100MB). Repacking with the stock kernel reduced the size to ~53MB.
- **Issue 3:** Encryption Password -> Ignored, as we intend to wipe `/data` anyway.

### 6. Advanced Modding & Branding
After successfully booting the ported recovery, we noticed it still identified as "Xiaomi Poco M3" (the donor). We performed additional steps to correct this.

**Ramdisk Modification:**
1.  **Unpack:** We unpacked the ported image again.
    ```bash
    unpackbootimg -i ported_recovery.img
    ```
2.  **Extract Ramdisk:** The ramdisk is a gzipped cpio archive.
    ```bash
    mkdir ramdisk_files
    cd ramdisk_files
    gzip -dc ../ramdisk.gz | cpio -i
    ```
3.  **Edit Properties:** We modified `prop.default` to change the device identity.
    - Replaced `Xiaomi` with `Realme`
    - Replaced `chime` / `SM6115` with `RMX2103`
    - Replaced `ro.product.board` with `bengal`
    - Added Author Tag: `ro.twrp.author=Roman Mia`
4.  **Repack:**
    ```bash
    find . | cpio -o -H newc | gzip > ../ramdisk-mod.gz
    ```
5.  **Final Build:** Repacked the boot image with the *modified* ramdisk.
    ```bash
mkbootimg ... --ramdisk ramdisk-mod.gz -o Roman_TWRP_RMX2103.img
```
