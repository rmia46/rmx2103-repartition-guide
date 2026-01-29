# Theory & Concepts: Deep Dive

This document explains the underlying principles, architecture, and mathematics behind the advanced modifications performed on the Realme RMX2103.

## 1. Android Storage Architecture

Modern Android devices (Android 10+) use a hierarchical storage system designed for flexibility and safe updates (Project Treble).

### The Physical Layer (GPT)
At the lowest level, the storage chip (UFS or eMMC) is divided into **Physical Partitions** defined by the **GUID Partition Table (GPT)**.
- **Tools:** `parted`, `gdisk`, `sgdisk`.
- **Identifiers:** `/dev/block/sda1`, `/dev/block/sda2` (or `mmcblk0p1`).
- **Critical Partitions:**
    - `xbl`, `abl` (Bootloaders - DO NOT TOUCH).
    - `boot` (Kernel).
    - `userdata` (Your photos, apps).
    - `super` (The container).

### The Logical Layer (Dynamic Partitions)
The `super` partition is a **Physical** partition that acts as a container for **Logical** partitions.
- **Mechanism:** The `super` partition starts with a **Metadata Header** (Geometry). This header maps "logical" names (like `system_a`) to physical "extents" (ranges of sectors) inside the `super` partition.
- **The Catch:** Resizing the *physical* `super` partition (using `parted`) does **not** update this Metadata Header. The OS will still see the old size until you generate new metadata (using `lpmake`).

---

## 2. Repartitioning Mathematics

### Units: Sectors vs. Bytes
- **Sector:** The smallest addressable unit of storage.
- **UFS Standard:** Usually **4096 bytes (4KB)** per sector.
- **Rule:** Always verify sector size with `parted /dev/block/sda print`.

### The Formulas

#### A. Calculating Partition Size in Sectors
$$ \text{Sectors} = \frac{\text{Desired Size (Bytes)}}{\text{Sector Size (Bytes)}} $$

*Example: You want a 12 GiB partition.*
$$ 12 \times 1024^3 = 12,884,901,888 \text{ bytes} $$
$$ \frac{12,884,901,888}{4096} = 3,145,728 \text{ sectors} $$

#### B. Calculating End Sector
$$ \text{End} = \text{Start} + \text{Length} - 1 $$

---

## 3. Recovery Porting Theory

Why did You mix parts from a Realme 7i and a Poco M3?

### The Kernel vs. Ramdisk
A `recovery.img` consists of two main parts packed together:
1.  **Kernel (`zImage`):** The core of the OS. It contains the **Drivers** for your specific display, touch panel, storage controller, and battery.
2.  **Ramdisk:** A small filesystem loaded into memory. It contains the **Programs** (TWRP executable, ADB daemon, init scripts).

### The "Porting" Logic
- **Problem:** You had no TWRP for RMX2103.
- **Solution:**
    - **Stock Kernel:** Kept the RMX2103 kernel.
    - **Custom Ramdisk:** Took the ramdisk from Poco M3 (Snapdragon 662 - same CPU).
- **Result:** The Poco M3 software runs on top of the RMX2103 hardware drivers.
