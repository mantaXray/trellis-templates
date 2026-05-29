# Firmware Memory Placement

> Use this guide for large static objects, DMA buffers, task stacks, linker edits, cacheable memory, display buffers, storage buffers, and map-file reviews.

## Memory Ownership Rules

- Do not let large `static` or global objects silently consume the default RAM region.
- Record the intended memory area for large buffers, persistent state, DMA buffers, display buffers, and filesystem buffers.
- Keep CPU-only state, DMA-facing buffers, display-facing buffers, and low-power/backup state in areas that match the MCU memory map.
- Low-end MCU projects may have only one SRAM region, but the ownership note still matters for future ports.

## Section Selection

Choose placement by access pattern, not only by size:

| Object type | Placement intent |
|---|---|
| CPU-only parser, command, status, or context state | General-purpose SRAM away from scarce stack/heap regions when possible |
| DMA-facing RX/TX buffer | DMA-accessible SRAM with required alignment and cache maintenance |
| UI/display frame or draw buffer | Display-capable memory documented by the BSP/linker script |
| Filesystem/log/storage state | Non-DMA application memory unless a driver requires DMA access |
| Backup/RTC retention data | Backup or retention domain, with reset behavior documented |

## DMA And Cache Rules

- DMA-facing buffers need the MCU's required alignment, commonly cache-line alignment on cache MCUs.
- CPU code invalidates cache before reading data written by DMA.
- CPU code cleans cache before starting DMA from CPU-written TX backing.
- Queue descriptors may reference shared buffers only while ownership prevents reuse.
- Moving a DMA buffer to a new region requires rechecking peripheral access, MPU/cache attributes, and alignment.

## Map-File Check

After an IAR or linker rebuild that changes buffers, stacks, memory sections, or large static objects:

1. Open the map file or equivalent memory report.
2. Find default RAM, stack, heap, and named section usage.
3. Look for unexpectedly large `.bss` or `.data` objects in scarce regions.
4. Treat meaningful RAM swings as review items, especially when task stacks, parsers, display, storage, or DMA buffers changed.
5. Record the result or the reason a build/map check was not possible.

## Pre-Commit Checklist

- [ ] Every new large or persistent object has intentional placement or a documented default-RAM reason.
- [ ] DMA-facing buffers have alignment and clean/invalidate rules.
- [ ] Display memory is not consumed by unrelated storage/log/protocol state.
- [ ] Map-file output or a skipped-build reason is recorded when memory placement changed.
