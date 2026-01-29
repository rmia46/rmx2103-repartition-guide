# RMX2103 Partition Expansion Project

This repository contains a comprehensive guide for physically and logically resizing the `super` partition on the Realme 7i (RMX2103 / Snapdragon 662).

## ⚠️ DISCLAIMER
**PROCEED AT YOUR OWN RISK.** These operations involve modifying the physical partition table (GPT). One mistake can result in a hard-brick. Always have a full stock firmware backup and the MsmDownloadTool ready.

## Guide Directory
1.  [**Journey Overview**](01_journey_overview.md) - High-level project summary.
2.  [**Recovery Porting**](02_recovery_porting.md) - How to build a functional recovery.
3.  [**Physical Partitioning Guide**](03_partitioning_guide.md) - Steps for `parted` and GPT modification.
4.  [**Logical Metadata Hack**](05_logical_partitioning.md) - Utilizing the new space with `lpmake`.
5.  [**Theory & Concepts**](06_theory_and_concepts.md) - Deep dive into Android partition logic.
6.  [**Troubleshooting**](04_troubleshooting.md) - Common errors and fixes.

## Key Accomplishments
- **Physical Super Size:** Expanded from 7.8GB to 12.0GB.
- **Goal:** Enabled the installation of large GSIs (Euclid OS, Pixel Experience GApps) that previously failed due to space constraints.

## Credits
- **Maintainer:** Roman Mia (rmia46)
- **Assistance:** Gemini CLI
