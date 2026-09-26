# Real Apple Network Server 700 hardware work

This branch tracks experiments performed on a **physical Apple Network Server 700**
while bringing Windows NT 4.0 PowerPC up on real hardware.

It intentionally starts from PappaDF's `pappadf-experiments` branch. The upstream
branches `main` and `pappadf-experiments` remain untouched in this fork.

## Hardware baseline

- Apple Network Server 700
- Open Firmware 2.26
- 64 MB parity RAM
- PowerPC PVR `00090202`
- serial console: 57600 8N1
- installed NT HDD: `/bandit/53c825@12/sd@0,0`
- CD-ROM: `/bandit/53c825@11/sd@0,0`
- floppy: `/bandit/gc/swim3`
- Windows NT 4.0 Build 1381 SP1

The installed HAL occupies an existing contiguous allocation of exactly
`0xD600` bytes / `0x6B` sectors at physical LBAs `0x172A..0x1794`.
Experimental HALs must fit this allocation; they are never truncated to fit.

## Current experimental baseline

The latest verified source baseline is **SCSIRING2**, derived from the working
SWEEP1 tree while preserving the cache/DMA work and the ANS-specific GPIO change.

SCSIRING2 adds a bounded 32-entry completion ring with adapter identity. It does
not dereference live DMA data buffers and does not broadly print from completion
paths. Runtime evidence identified repeated `SRB_STATUS_BUS_RESET (0x0E)`
bulk completions on the same adapter carrying the installed NT HDD traffic.

The exact SCSIRING2 build used on hardware is kept outside this upstream-derived
branch until the WORK source tree is imported verbatim. Do not reconstruct it
from this document.

## IOMAPFIX2 candidate

A source review found that the inherited `IoMapTransfer` implementation could
derive a PFN-array index from an invalid `CurrentVa`. The first proposed
zero-progress rejection was discarded after static inspection of stock NT4 PPC
`SCSIPORT.SYS`: its scatter/gather loop can advance the output entry pointer
without advancing total progress when MapTransfer returns a zero length.

IOMAPFIX2 therefore:

- preserves the existing physical-contiguity correction;
- preserves `ByteOffset` semantics and exclusive-end bounds;
- accepts `MappedSystemVa` only when `MDL_MAPPED_TO_SYSTEM_VA` is set;
- uses widened arithmetic for the page-count calculation;
- returns zero only for a genuine caller-requested zero-length transfer;
- uses the existing non-returning `KeBugCheckEx` import for malformed nonzero
  requests instead of inventing an address or reporting fake progress.

WORK integrated this exact patch into the byte-reproducible SCSIRING2 baseline.
Only `dma.c` changed. Verification reported:

- 2,066 C sanitizer tests passed;
- 18 Python model tests passed;
- two PowerPC builds were byte-identical;
- final candidate `HALIM2.DLL` size: `0xD600`;
- PE `SizeOfImage`: `0x13000`;
- PE checksum: `0x19266`.

The NT4 DDK evidence used by WORK supports the mapped-offset convention, and the
diagnostic bugcheck paths use the existing kernel import mechanism.

## First HALIM2 hardware run

The first reported hardware run is evidence, not proof of causality.

Observed:

- no IOMAPFIX2 `E2/494D` diagnostic stop;
- the earlier Beep.sys failure did not occur on that run;
- boot stopped at `C000026C` loading `Npfs.SYS`, status `C0000185`;
- SWEEP1 reached 20/20;
- last I/O count: 121;
- SCSIRING2 count: 426;
- the retained ring contained 12 HDD bulk `BUS_RESET 0E` completions through
  caller pair `8007D62C / 80081874`;
- successful HDD READ(10) completions at LBA/count
  `2E535/40`, `2E575/40`, and `2E5B5/16` cover the previously mapped NPFS
  file region;
- those successful completion statuses do **not** prove returned data integrity.

Earlier NPFSTRACE1 work had already reached NPFS, so this run is not claimed as
a new furthest point and does not establish IOMAPFIX2 as a fix.

The immediate experiment is one identical repeat of the installed HALIM2
candidate. No new HAL or broad logger is justified before that evidence is
captured.

## Evidence discipline

This branch should keep observations and hypotheses separate.

In particular:

- successful SRB/SCSI completion does not prove payload bytes are correct;
- a bulk `BUS_RESET` completion is not equivalent to one individually logged
  failed SRB;
- movement of the visible failing driver between boots may be timing-sensitive;
- one improved boot does not prove that a source change caused the improvement;
- no source change should be described as fixing Beep.sys/C0000221 without
  hardware evidence that the changed path actually activated.

## Redistribution

Do not add Microsoft Windows NT installation media, Microsoft binaries, patched
Microsoft loaders, or floppy/disk images containing Microsoft files to this
repository. Keep the repository to redistributable source, patches, tooling,
documentation, hashes, and independently captured hardware evidence.
