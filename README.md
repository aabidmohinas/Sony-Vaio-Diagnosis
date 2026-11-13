# Sony VAIO SVE151J13M: Technical Diagnosis (Laptop 1)

Diagnosis of random kernel panics, failed boots and no-display starts on a Sony VAIO SVE151J13M.

- **Diagnosed:** 17-18 October 2026
- **Report prepared:** 25 October 2026

📄 **[Read the full diagnosis report (PDF)](docs/diagnosis.pdf)**

## Machine

| Item | Detail |
|------|--------|
| Model | Sony VAIO SVE151J13M |
| CPU | Intel Core i5-3230M, 2.60 GHz, 2 cores / 4 threads |
| Memory | DDR3 SODIMM, 2 slots (originally 4GB + 2GB) |
| Storage | Hitachi HTS723232A7A364, 320 GB 2.5in SATA HDD |
| OS | Ubuntu (kernels 7.0.0-34 and 7.0.0-38), GRUB 2.12 |
| Toolbox | Arch Linux live USB, MemTest86+ |

## Summary

The symptoms looked unrelated (black screen, kernel panics, out-of-memory errors on a live USB, GRUB errors, initramfs shell, emergency mode). Working down the boot chain showed one hardware fault and two knock-on software faults.

**Root cause chain:** failing RAM module → memory corruption → crashes and bad writes → damaged filesystem metadata and a half-written kernel update → failed boots.

## Findings

| # | Finding | Status |
|---|---------|--------|
| 1 | Faulty 4GB RAM module (primary cause) | Module removed |
| 2 | ext4 metadata corruption on `sda2` | Repaired with `fsck` |
| 3 | Half-installed kernel 7.0.0-38 | Purged; running 7.0.0-34 |
| 4 | Worn hard drive (SMART clean, ~1.5M load cycles) | Monitor / replace with SSD |
| 5 | Intermittent power-on (needs a knock) | Open, watch |

**Outcome:** boots Ubuntu on kernel 7.0.0-34-generic with 0 failed systemd units.

## Method

Layered diagnosis, hardware first, moving up only when a layer was proven healthy:

1. Power and firmware
2. Bootloader (GRUB shell)
3. Kernel and initramfs
4. Memory (MemTest86+)
5. Storage (`smartctl`)
6. Filesystem (`fsck -f`)
7. OS packages (`systemctl`, `journalctl`, `dpkg`)

## Recommendations

- **High:** run a full MemTest86+ pass (ideally two) on the exact RAM configuration in use.
- **High:** don't run `apt upgrade` until memory is confirmed stable, then run it on mains power.
- **Medium:** keep backups, reduce head parking or swap the HDD for a 2.5in SATA SSD, and clean the fan and vents.
- **Watch:** if it again needs a knock to start, or crashes with known-good RAM, suspect the motherboard.

## Repository structure

```
.
├── README.md
└── docs/
    └── diagnosis.pdf
```

## Limits

Based on commands and outputs captured during the troubleshooting session. The knock-to-start fault was not conclusively explained.