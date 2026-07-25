---
publish: true
title: x86 Real-Mode Memory Map
created: 2025-12-13
modified: 2026-07-25T01:52:40.837+05:30
tags:
  - low-level
  - hardware
  - learning
  - os
---

# x86 Real-Mode Memory Map

## What lives at `0xB8000` and the rest of the lower 1 MB on real PC hardware (simplified):

```text
0x00000000 ──────────────────────────────────────────────────────────
             INTERRUPT VECTOR TABLE (IVT)
             - Size: 1 KB (1024 bytes)
             - Range: 0x00000000 → 0x000003FF
             - Contains 256 interrupt pointers
               (each pointer = 4 bytes: segment:offset)
             - BIOS + hardware rely on these entries
             - You must NEVER overwrite this area

0x00000400 ──────────────────────────────────────────────────────────
             BIOS DATA AREA (BDA)
             - Size: 256 bytes
             - Range: 0x00000400 → 0x000004FF
             - Stores system state set by BIOS:
                 * Keyboard status
                 * COM/LPT port data
                 * Display mode info
                 * Equipment list
                 * Memory size under 640 KB
             - Also critical. Do NOT touch.

0x00000500 ──────────────────────────────────────────────────────────
             LOW REAL-MODE USABLE RAM (SCRATCH SPACE)
             - Size: ~30 KB
             - Range: 0x00000500 → 0x00007BFF
             - This is the first memory that’s safe for you to use
             - Bootloaders often:
                 * Put temporary stacks here
                 * Read sectors here
                 * Store disk buffers
             - Still under the 1 MB real-mode boundary

0x00007C00 ──────────────────────────────────────────────────────────
             BOOTLOADER (BIOS loads here)
             - Size: exactly 512 bytes (one sector)
             - Range: 0x00007C00 → 0x00007DFF
             - This is where your `boot.asm` executed
             - The BIOS always:
                 * Reads LBA sector 0
                 * Loads into 0x7C00
                 * Sets CS:IP to 0000:7C00
             - A sacred location in IBM PC architecture

0x00007E00 ──────────────────────────────────────────────────────────
             REMAINING LOW RAM UNTIL VIDEO MEMORY
             - Size: ~480 KB
             - Range: 0x00007E00 → 0x0009FFFF
             - Often used by early loaders:
                 * Move kernel here temporarily
                 * Prepare GDT/IDT
                 * Setup buffers
                 * Enable A20 line
             - This memory is still safe but messy

0x000A0000 ──────────────────────────────────────────────────────────
             VGA GRAPHICS MEMORY (FRAMEBUFFER)
             - Size: 64 KB
             - Range: 0x000A0000 → 0x000AFFFF
             - Used for pixel graphics modes:
                 * Mode 13h (320×200)
                 * VESA modes (varies)
             - CPU writes pixel values directly here

0x000B8000 ──────────────────────────────────────────────────────────
             VGA TEXT MODE BUFFER
             - Size: 4 KB (depending on mode)
             - Range: 0x000B8000 → ~0x000B8FA0
             - Layout (80×25 mode):
                 * 2000 characters
                 * Each: 1 byte char + 1 byte attribute = 2 bytes
             - Your ‘K’ printed earlier is ultimately reflected here
             - CPU writes characters here directly

0x000C0000 ──────────────────────────────────────────────────────────
             BIOS ROM + EXPANSION ROMs
             - Size: 256 KB
             - Range: 0x000C0000 → 0x000FFFFF
             - Contains:
                 * System BIOS firmware
                 * Video BIOS
                 * Option ROMs (network PXE boot, RAID controllers)
             - Memory is READ-ONLY, mapped to actual ROM hardware
             - Firmware lives here permanently

0x00100000 ──────────────────────────────────────────────────────────
             1 MB MARK — HIGH MEMORY — KERNEL LAND
             - Size: "Everything above 1 MB" (depends on RAM)
             - Range: 0x00100000 → end of RAM
             - Clean, uncontested, flat RAM
             - Perfect for protected-mode kernels:
                 * No BIOS data
                 * No video memory
                 * No ROM shadowing
             - This is where you will load and execute your kernel
             - Common OS convention:
                 Kernel physical start = **0x00100000**
```

## Why The Map Looks Like This (History + Hardware Reasons)

Everything below 1 MB is a disaster of 1980s design, preserved eternally for compatibility:

- The **first 1 KB** (IVT) must exist so interrupts work.
- The **next 256 bytes** (BDA) hold BIOS system info before any OS loads.
- From 0x500 → 0x7BFF is old-school RW RAM available to boot sectors.
- The bootloader is forced to be at **exactly** 0x7C00 due to 1981 IBM PC BIOS design.
- The space from **0xA0000 to 0xBFFFF** is reserved by VGA hardware.
- The **top 256 KB** (0xC0000–0xFFFFF) is ROM where the BIOS literally lives.

The region above 1 MB is clean only because:

> After the A20 line is enabled and the CPU enters protected mode, memory above 1 MB is flat, linear, and not polluted by BIOS history.

This is why _every modern kernel starts at 1 MB_.

## Related

- [[docker-training]]
