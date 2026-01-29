# RMX2103 Partition Expansion Project

This repository contains documentation and methodology for physically resizing the `super` partition on the Realme 7i (RMX2103) Snapdragon 662 variant.

## ⚠️ DISCLAIMER
**PROCEED AT YOUR OWN RISK.** These operations involve modifying the physical partition table (GPT). One mistake can result in a hard-brick (EDL mode required for recovery). Always have a full stock firmware backup and the MsmDownloadTool ready.

## Document Directory
1.  [**Journey Overview**](01_journey_overview.md) - The high-level story of the project.
2.  [**Recovery Porting**](02_recovery_porting.md) - How to build a working TWRP/OrangeFox for this device.
3.  [**Physical Partitioning Guide**](03_partitioning_guide.md) - Precise steps for `parted` sector math.
4.  [**Logical Metadata Hack**](05_logical_partitioning.md) - Bypassing the 7.8GB metadata limit with `lpmake`.
5.  [**Theory & Concepts**](06_theory_and_concepts.md) - Deep dive into GPT, Dynamic Partitions, and Sector Math.
6.  [**Troubleshooting**](04_troubleshooting.md) - Common errors and their solutions.

## Summary of Achievement
- **Physical Super Size:** 7.8GB -> 12.0GB
- **Result:** Successfully installed large Android 16 GApps GSIs that previously failed with "Not enough space".
- **Maintainer:** Roman (rstation) & Gemini CLI
