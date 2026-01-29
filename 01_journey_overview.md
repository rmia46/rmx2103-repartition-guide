# The Journey: Resizing Realme C21Y/7i (RMX2103) Partitions

## Overview
This documentation covers the process of manually resizing the physical `super` partition on a Realme 7i (RMX2103) device to accommodate a larger GSI (Android 16 GApps).

## The Problem
- **Goal:** Install Android 16 GSI (Euclid OS, GApps variant).
- **Blocker:** The physical `super` partition (~7.8GB) is too small to fit the system image + vendor + product, causing "Not enough space" errors.
- **Constraint:** `parted` (the tool needed to resize physical partitions) requires an unmounted `/data` partition, which is impossible on a running OS or Stock Recovery.
- **Roadblock:** No official TWRP or Custom Recovery exists for this specific device variant ("myrtle" / RMX2103 / bengal).

## The Solution Strategy
1.  **Environment:** Boot into a Custom Recovery (TWRP/OrangeFox) to gain a root shell with unmounted partitions.
2.  **Porting:** Since no recovery exists, port one from a similar device (Poco M3 - Snapdragon 662).
    - Use the **Kernel & DTB** from the Stock Recovery (to ensure booting).
    - Use the **Ramdisk** from the Poco M3 TWRP (to provide the interface and tools).
    - Use `mkbootimg` to stitch them together.
3.  **Booting:** Flash this hybrid "Frankenstein" recovery.
4.  **Resizing:** With ADB access established in recovery, use `parted` to delete `userdata` and `super`, then recreate them with new boundaries (e.g., 12GB Super).
5.  **Metadata Overhaul:** Bypassing the stock 7.8GB metadata limit by using `lpmake` to generate a new 12GB metadata blueprint. Extract stock partitions with `lpunpack` and flash them individually.
6.  **GSI Deployment:** Finally, flash the Android 16 GApps GSI into the newly expanded space.
