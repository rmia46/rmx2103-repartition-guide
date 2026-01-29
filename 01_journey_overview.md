# The Journey: Resizing Realme C21Y/7i (RMX2103) Partitions

## Overview
This documentation covers the process of manually resizing the physical `super` partition on a Realme 7i (RMX2103) device to accommodate a larger GSI (Android 16 GApps).

## The Problem
- **Goal:** Install Android 16 GSI (Euclid OS, GApps variant).
- **Blocker:** The physical `super` partition (~7.8GB) was too small to fit the system image + vendor + product, causing "Not enough space" errors.
- **Constraint:** `parted` (the tool needed to resize physical partitions) requires an unmounted `/data` partition, which is impossible on a running OS or Stock Recovery.
- **Roadblock:** No official TWRP or Custom Recovery existed for this specific device variant ("myrtle" / RMX2103 / bengal).

## The Solution Strategy
1.  **Environment:** We needed a Custom Recovery (TWRP/OrangeFox) to gain a root shell with unmounted partitions.
2.  **Porting:** Since no recovery existed, we "ported" one from a similar device (Poco M3 - Snapdragon 662).
    - We took the **Kernel & DTB** from the Stock Recovery (to ensure booting).
    - We took the **Ramdisk** from the Poco M3 TWRP (to provide the interface and tools).
    - We used `mkbootimg` to stitch them together.
3.  **Booting:** We flashed this hybrid "Frankenstein" recovery.
4.  **Resizing:** With ADB access established in recovery, we used `parted` to delete `userdata` and `super`, then recreated them with new boundaries (12GB Super).
5.  **Metadata Overhaul:** We encountered a "Not enough space" error because the stock metadata was hardcoded to 7.8GB. We used `lpmake` to generate a new 12GB metadata blueprint, extracted stock partitions with `lpunpack`, and flashed them individually.
6.  **GSI Deployment:** Finally, we flashed the Android 16 GApps GSI into the newly expanded space.
