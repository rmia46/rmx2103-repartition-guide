# Logical Partitioning & Metadata Hack

## The Concept
After physically resizing the `super` partition (e.g., to 12GB), flashing a stock `super.img` will reset the internal metadata to the stock size (e.g., 7.8GB). This causes FastbootD to reject larger GSIs because it believes the "logical" room is still small.

## The Solution: Custom Metadata
To utilize the new physical space, you must create a new "Blueprint" (`super_empty.img`) that acknowledges the full physical capacity.

## Steps to Execute

### 1. Unpack the Stock Super
Extract the individual partition images from the stock `super.img`. This allows you to selectively restore critical drivers (`vendor`, `my_*`) while leaving space for a custom `system`.

```bash
mkdir -p stock_super_contents
lpunpack super_bd.img stock_super_contents
```

### 2. Generate the New Metadata
Use `lpmake` to build a new logical structure. 
- **Device Size:** Set to the new physical size (e.g., 12 GiB).
- **Partitions:** Define all required partitions (`system`, `vendor`, `my_product`, etc.).

**Example Command:**
```bash
lpmake -d 12884901888 \
       -m 65536 \
       -s 2 \
       -g qti_dynamic_partitions:12880707584 \
       -p "system:readonly:0:qti_dynamic_partitions" \
       -p "vendor:readonly:0:qti_dynamic_partitions" \
       ... \
       -F -S \
       -o super_empty_sparse.img
```

### 3. Flash the New Metadata
In **Bootloader** mode:
```bash
fastboot flash super super_empty_sparse.img
```

### 4. Restore Components in FastbootD
Enter **FastbootD** and flash the extracted `.img` files into their respective slots.
```bash
fastboot flash vendor vendor.img
fastboot flash odm odm.img
# ... flash other required partitions ...
```

### 5. Final GSI Install
Now that the metadata reflects the full 12GB space, flash the GSI:
```bash
fastboot flash system your_gsi_image.img
```
