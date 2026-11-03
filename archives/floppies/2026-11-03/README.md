# BLUE and GREEN floppy file backups — November 3, 2026

Prepared at the owner's request so others can inspect the ANS testing files.
**These are clearly labeled saved file-backup archives, not raw floppy images
or verified bootable clones.** No physical disk was read or changed while the
live boot test was awaiting a result.

| Download | Payload files | Archive bytes |
|---|---:|---:|
| [BLUE file archive](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-03/BLUE-file-backup-2026-11-03.tar.xz) | 21 | 212,468 |
| [GREEN file archive](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-03/GREEN-file-backup-2026-11-03.tar.xz) | 2 | 1,728 |

## What is included

- **BLUE:** the verified pre-HALV1 file backup from October2, plus the preserved
  HALV1.DLL whose bytes match the documented subsequent BLUE transfer. It is
  an assembled, documented file set, not a fresh November sector capture.
- **GREEN:** OSLOLD.SCT and OSLNEW.SCT,512 bytes each, from the complete saved
  ANS-load captures. Their media fingerprints were rechecked in November3
  testing. OLD FNV52610671; NEW FNV5FE5BDE9.
- Each archive contains README.md, MANIFEST.json, SHA256SUMS.txt and files/.
  Mac AppleDouble and filesystem-event metadata is excluded.
- Separate [BLUE manifest](BLUE-MANIFEST.json), [GREEN manifest](GREEN-MANIFEST.json)
  and [archive checksums](SHA256SUMS.txt) are provided for inspection.

BLUE's BOOTDISK.OF reads a veneer from fixed raw floppy LBAs starting at0x15D.
Ordinary file copying does not reproduce that allocation or create a working
boot disk. These archives preserve file bytes but do not capture the FAT,
boot sector or unused/deleted sectors. Do not write the archives as disk images.
GREEN is recovery data media; its mapped HDD LBA0x1229 belongs to this specific
installation and is not a universal patch address for another machine.

## Verify and extract

Download both archives and SHA256SUMS.txt into one directory. On macOS:

```sh
shasum -a 256 -c SHA256SUMS.txt
mkdir BLUE GREEN
tar -xJf BLUE-file-backup-2026-11-03.tar.xz -C BLUE
tar -xJf GREEN-file-backup-2026-11-03.tar.xz -C GREEN
```

Archive SHA-256 values:

```text
1eed0f5617ef0a464077ae5f3da1b7ce5672f460548630378bd24a33cf0a7287  BLUE-file-backup-2026-11-03.tar.xz
5388b683d65144677f0675ba43eff0677254b9ba52a1fbec0889925ff701716f  GREEN-file-backup-2026-11-03.tar.xz
```

The manifests also record each payload's size, hash and backup source. The
archives contain historical test files; inclusion does not select an image
for deployment or make all candidate HALs current. They are testing backups,
not a new software release or a verified reproduction/build recipe.

Follow the [current hardware progress](../../../docs/ANS-HARDWARE-PROGRESS-2026-11-03.md)
and [short startup guide](../../../docs/ANS-STARTUP.md) before interpreting the
files. Complete physical floppy images can be captured separately after the
live test and stock-loader restoration, when the media is available at the Mac.
