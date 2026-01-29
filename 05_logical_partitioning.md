# Logical Partitioning & Metadata Hack

## The Problem (Part 2)
After physically resizing the `super` partition to 12GB, flashing the stock `super.img` caused a conflict. The stock metadata header was "locked" to the original 7.8GB size, causing FastbootD to reject larger GSIs with "Not enough space" and "Failed to write partition table".

## The Solution: Custom Metadata
To utilize the new 12GB physical space, You had to destroy the stock metadata and create a new "Blueprint" (super_empty.img) that acknowledges the full 12GB.

## Steps Taken

### 1. Unpacking the Stock Super
You used `lpunpack` to extract the individual partition images from the stock `super_bd.img`. This allows you to restore the critical Realme drivers (`vendor`, `my_*`) while leaving out the bulky stock `system` and `product`.

```bash
mkdir -p ~/cook/stock_super_contents
lpunpack /data/systools/android/rmx2103/super_bd.img ~/cook/stock_super_contents
```

### 2. Generating the 12GB Metadata
You used `lpmake` to build a new logical structure. 
- **Device Size:** 12,884,901,888 bytes (12 GiB)
- **Metadata Size:** 65536
- **Partitions:** Defined all stock partitions (`vendor`, `my_product`, etc.) as `readonly`.

**The Command:**
```bash
lpmake -d 12884901888 \
       -m 65536 \
       -s 2 \
       -g qti_dynamic_partitions:12880707584 \
       -p "system:readonly:0:qti_dynamic_partitions" \
       -p "vendor:readonly:0:qti_dynamic_partitions" \
       ... (all other partitions) ... \
       -F -S \
       -o ~/cook/super_empty_sparse.img
```

### 3. Flashing the New Blueprint
In **Bootloader** mode:
```bash
fastboot flash super ~/cook/super_empty_sparse.img
```

### 4. Restoring Components in FastbootD
You entered **FastbootD** to flash the extracted `.img` files into their new logical slots.
```bash
fastboot flash vendor vendor.img
fastboot flash odm odm.img
# ... (flashed all my_* images) ...
```

### 5. Final GSI Install
With the metadata now allowing up to 12GB of total logical space, You finally flashed the Gapps GSI:
```bash
fastboot flash system infinity-os-g-system.img
```
