# Physical ANS 700 progress — November 5, 2026

**Recovered checkpoint. HALIM2 repeatedly reaches the NPFS load failure.
Text-mode NT Setup completed earlier on this real ANS 700. GUI Setup is
the remaining installation stage.** The investigation is being handed over
with source, captures, manifests and reconstructed floppy images.

Start with the [continuation handoff](ANS-HANDOFF-2026-11-05.md) and
[BLUE/GREEN downloads](../archives/floppies/2026-11-05/README.md).

## Progress since the November 3 snapshot

| Stage | Observed result |
|---|---|
| OSLOADER recovery | Stock loader restored and verified at size `6BE00`, FNV `BB6AEFD2`. |
| HAL image rejection investigation | The named and raw HAL contained an AppleDouble header. The clean HALV1 data fork was restored with full comparisons and accepted by NT. |
| HALV1 runtime validation | Beep's checksum loop raised `C0000006` with underlying I/O status `C0000185`, before its comparison. |
| CLASSEND diagnostic | Its hooks installed, but the supplied boot stopped earlier at Sfloppy with zero matching Beep records. |
| HALIM2 restoration | Loaded at `3E00000 D600`, FNV `085AE99B`, full raw readback matched, named-file comparison `0`. |
| HALIM2 recovery and repeat | Both reach NPFS / `C0000185`; each CRASH1 block matches the October 1 recovery exactly. |
| Whole named NPFS read | Firmware loaded `3E00000 12A50`; complete-file FNV `80C8D029` matches the media reference. |
| Raw NPFS comparison | First attempt stopped at an incorrectly specified partition guard. Corrected MBR/EBR-aware routine is prepared; its live result is pending. |

The latest boot position is the earlier best checkpoint, with SWEEP20/20,
last I/O121 and ring426. Input classes connect and video-driver resources
are processed before the NPFS stop. No GUI Setup or desktop success has
been captured on this physical machine.

## What the current evidence means

The NPFS range has three reported successful reads covering its recorded
file allocation. Those notifications precede final SCSIPORT/CLASS2/IRP
completion. They do not establish final NT read status or delivered bytes.
The old NPFSTRACE1 silence should not be generalized to this later capture.

The original firmware check sampled three NPFS sectors. The new named-file
read covers the whole file, but a fresh raw/named byte comparison remains
pending. The recorded layout has extended type05 at11000 and logical
FAT16 at11020. A guard that expects a primary FAT16 entry at11020 rejects
this valid layout before reaching the payload check.

## Packaged baseline

HALIM2 is exactly `D600` bytes, FNV `085AE99B`, SHA-256
`d2fb635ae4760ffe394dafa80e41fe6e06e60b3b7b2725316160fb16a5d89c8f`.
The packaged source rebuilt byte-identically with LLVM clang/lld 23.1.1.
The root repository source is an older snapshot. The handoff source archive
is the recovered hardware baseline.

The new BLUE and GREEN images are reconstructed 1.44 MB FAT12 disk images.
Both pass filesystem and complete file-payload checks. BLUE's two fixed
raw veneer ranges match the saved boot image exactly. The new images have
not been tested on hardware. Per-file provenance and image hashes are in
their manifests.

## Next work

1. Run the corrected read-only whole-file NPFS comparison from the handoff.
2. Reconcile an unexpected fingerprint or mapping with the actual file
   version before selecting any repair.
3. If the file checks agree, implement the bounded native image-loader
   stage/status observation described in the
   [reassessment](ANS-REASSESSMENT-2026-11-05.md).

The named-file result has passed. The corrected raw check and proposed
loader-stage diagnostic remain outstanding. This handoff makes no new
claim of a storage fix or forward boot milestone.
