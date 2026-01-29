# Theory & Concepts: Deep Dive

This document explains the principles, architecture, and mathematics behind advanced Android partitioning.

## 1. Android Storage Architecture

Modern Android devices (Android 10+) use a hierarchical storage system.

### The Physical Layer (GPT)
The storage chip (UFS/eMMC) is divided into **Physical Partitions** defined by the **GUID Partition Table (GPT)**.
- **Tools:** `parted`, `gdisk`.
- **Key Partitions:** `super` (the container), `userdata` (user files), and bootloaders.

### The Logical Layer (Dynamic Partitions)
The `super` partition is a physical container for **Logical** partitions.
- **Mechanism:** The `super` partition contains a **Metadata Header**. This header maps logical names (`system`, `vendor`) to physical sectors inside the `super` partition.
- **The Challenge:** Resizing the *physical* `super` partition does not update the *metadata*. You must use `lpmake` to expand the logical boundaries.

---

## 2. Repartitioning Mathematics

### Units: Sectors vs. Bytes
- **Sector:** The smallest addressable unit of storage.
- **Standard:** Usually **4096 bytes (4KB)** per sector for UFS.
- **Verification:** Always check sector size with `parted /dev/block/sda print`.

### The Formulas

#### A. Calculating Partition Size in Sectors
$$ \text{Sectors} = \frac{\text{Desired Size (Bytes)}}{\text{Sector Size (Bytes)}} $$

#### B. Calculating End Sector
$$ \text{End} = \text{Start} + \text{Length} - 1 $$

---

## 3. Recovery Porting Theory

### The Kernel vs. Ramdisk
A `recovery.img` contains:
1.  **Kernel:** Hardware drivers (display, touch, storage). Must match the device hardware.
2.  **Ramdisk:** The software environment (TWRP/OrangeFox). Can be borrowed from similar CPUs.

### The Porting Logic
By combining the **Stock Kernel** (to ensure the screen/touch works) with a **Donor Ramdisk** (to provide the TWRP interface), you can create a functional custom recovery for devices that don't have one.

---

## 4. Risks & Cautions

1.  **Partition Alignment:** Always try to align partitions to 1MB boundaries (256 sectors in 4KB/sector devices) for optimal performance.
2.  **Sequential Order:** Partitions on the GPT must be contiguous. If you expand a partition, all subsequent partitions must be deleted and recreated at shifted offsets.
3.  **Wiping Data:** Modifying the `userdata` start/end points will corrupt the filesystem, requiring a full format.
