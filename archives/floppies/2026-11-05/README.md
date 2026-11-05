# BLUE and GREEN handoff disks — November 5, 2026

Start with the [current handoff](../../../docs/ANS-HANDOFF-2026-11-05.md).
The installed ANS hard disk has the recovered HALIM2 baseline. Both BLUE
and GREEN downloads below include HALIM2.DLL, CENDCHK.OF and NPFSDEFS.OF.

## Downloads

| Artifact | Compressed bytes |
|---|---:|
| [BLUE-handoff-2026-11-05.img.xz](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/BLUE-handoff-2026-11-05.img.xz) | 215,908 |
| [BLUE-files-2026-11-05.tar.xz](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/BLUE-files-2026-11-05.tar.xz) | 214,180 |
| [GREEN-handoff-2026-11-05.img.xz](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/GREEN-handoff-2026-11-05.img.xz) | 43,048 |
| [GREEN-files-2026-11-05.tar.xz](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/GREEN-files-2026-11-05.tar.xz) | 43,928 |
| [HALIM2-source-2026-11-05.tar.xz](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/HALIM2-source-2026-11-05.tar.xz) | 130,176 |

[SHA256SUMS.txt](SHA256SUMS.txt) covers all five downloads.
[BLUE manifest](BLUE-MANIFEST.json) and [GREEN manifest](GREEN-MANIFEST.json)
record every payload's size, SHA-256 and provenance, plus the uncompressed
image SHA-256. Each disk image decompresses to exactly 1,474,560 bytes.

## What the downloads contain

BLUE is the documented October backup updated with WORK #3's byte-verified
HALV1, HALCEND and CENDCHK files, with the archived HALCR1/HALSW1 floppy
copies removed as recorded. Its preserved raw veneers occupy 21..15C and
15D..298. The installed-system BOOTDISK.OF route is the intended use here.
The historical BOOT.OF/SETUP.OF and PPC/SETUPLDR payloads remain archival.
In particular, the saved PPC/SETUPLDR was identified as Alpha machine0184,
so it is not selected for this installed-system boot route.

GREEN contains the documented loader-sector references and helpers,
HALV1/HALCEND/CENDCHK, and an added HALIM2 recovery copy for this handoff.
NPFSDEFS.OF was added to both handoff sets. It defines the corrected
read-only raw comparison; its corrected live result is pending.

The disk images are reconstructed from these documented payloads. They
pass FAT12 filesystem checks, complete payload extraction comparisons,
and BLUE fixed-sector veneer comparisons. Their new physical-media boot
has not been tested. They are not fresh raw captures of the original
physical floppies. The file archives contain the same payloads under files/,
README.md, MANIFEST.json, and per-file SHA256SUMS.txt.

The images provide floppy media. The current installed NT hard disk and
its machine-specific layout remain required to reproduce the recovered
boot. The current root repository source is older. Use the included
HALIM2 source archive for this recovered hardware baseline.

## Verify and unpack

Place the five downloads and SHA256SUMS.txt in one directory. Verify with

```sh
shasum -a 256 -c SHA256SUMS.txt
```

Decompress the images with Python's standard library

```sh
python3 - <<'PYDISKS'
from pathlib import Path
import lzma
for name in ['BLUE', 'GREEN']:
    packed = Path(name + '-handoff-2026-11-05.img.xz')
    image = lzma.decompress(packed.read_bytes())
    assert len(image) == 1474560
    packed.with_suffix('').write_bytes(image)
PYDISKS
```

The resulting .img files are full-disk images for a suitable floppy image
writer. Ordinary file copying cannot recreate BLUE's fixed-sector boot
layout. File archives and HALIM2 source can be extracted with tar -xJf.

The manifests preserve the payload origins. Third-party historical payloads
retain their original provenance. This handoff is a testing snapshot.
