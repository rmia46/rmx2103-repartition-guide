# Recovery Porting Guide: Poco M3 to Realme RMX2103

## Theory
Recoveries (like TWRP) consist of a **Kernel** (hardware drivers) and a **Ramdisk** (the actual program/UI).
- **Poco M3 (Citrus):** Same SoC (Snapdragon 662) as the RMX2103.
- **Realme RMX2103:** The target device.

By swapping the kernel, the Poco M3 software is tricked into running on the Realme hardware.

## Steps to Port

### 1. Preparation
Extract the `recovery.img` from the stock firmware and download a `twrp-poco-m3.img`.

### 2. Unpacking
Use `unpackbootimg` to separate the components of both images.

```bash
unpackbootimg -i stock_recovery.img -o stock_out
unpackbootimg -i donor_twrp.img -o donor_out
```

### 3. Repacking (The "Frankenstein" Command)
Use `mkbootimg` to combine the following:
- **Kernel:** From Stock
- **DTB/DTBO:** From Stock (Critical for screen/hardware mapping)
- **Ramdisk:** From Donor (Poco M3)
- **Cmdline:** Use Stock cmdline + `androidboot.selinux=permissive` (to bypass security checks) + `androidboot.usbcontroller=4e00000.dwc3` (to fix ADB).

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
Since `fastboot boot` is disabled on many Realme devices, flash it directly:
```bash
fastboot flash recovery ported_recovery.img
fastboot reboot recovery
```

### 5. Troubleshooting
- **Issue 1:** "Current boot destroyed" -> Fix by flashing a patched/disabled VBMeta.
- **Issue 2:** "Volume Full" -> The raw donor image might be too large. Repacking with the stock kernel usually reduces the size significantly.
- **Issue 3:** Encryption Password -> Can be ignored if the intention is to wipe `/data`.

### 6. Advanced Modding & Branding
To change the device identity from "Xiaomi Poco M3" to "Realme 7i":

**Ramdisk Modification:**
1.  **Unpack:** Unpack the ported image again.
    ```bash
    unpackbootimg -i ported_recovery.img
    ```
2.  **Extract Ramdisk:** 
    ```bash
    mkdir ramdisk_files
    cd ramdisk_files
    gzip -dc ../ramdisk.gz | cpio -i
    ```
3.  **Edit Properties:** Modify `prop.default` to change the device identity.
    - Replace `Xiaomi` with `Realme`
    - Replace `chime` / `SM6115` with `RMX2103`
    - Replace `ro.product.board` with `bengal`
    - Add Author Tag: `ro.twrp.author=Your Name`
4.  **Repack:** 
    ```bash
    find . | cpio -o -H newc | gzip > ../ramdisk-mod.gz
    ```
5.  **Final Build:** Repack the boot image with the *modified* ramdisk.
    ```bash
mkbootimg ... --ramdisk ramdisk-mod.gz -o Roman_TWRP_RMX2103.img
```
