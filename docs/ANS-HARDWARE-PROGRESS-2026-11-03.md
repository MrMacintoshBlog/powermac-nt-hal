# ANS Windows NT hardware progress — November 3, 2026

Physical testing and serial captures by Mr. Macintosh
([MrMacintoshBlog](https://github.com/MrMacintoshBlog)).

**Current checkpoint:** the corrected firmware configuration now reaches NT's
loader. The stock loader still rejects HALV1 with its generic missing/corrupt
HAL message. The reviewed LDRRS1 status-only diagnostic is temporarily installed,
has passed sector readback and complete-file fingerprint checks, and its next
boot output is awaited. No native HAL-load return code or VAL1 result has yet
been received. Stock restoration is required after that one diagnostic boot.

## What changed since October

1. **Firmware settings were measured instead of inferred.** Serial input/output
   were already `ttya:38400` and auto-boot was `false`, but NT-specific values
   had reverted to `real-base=-1`, `load-base=4000`, real mode enabled and
   little-endian disabled. The time/cause of that change is unknown.
2. **The previously working NT configuration was restored and read back after
   reset:** real-base `3F00000`, load-base/loadaddr `3E00000`, real-mode `false`,
   little-endian `true`, virt-base `-1`. Serial routing remained at38400.
3. **Stock OSLOADER was restored and verified.** The target sector returned
   write1/read1/full512-byte original comparison0, with native instruction
   bytes `3C 23 80 38`. The actual named HDD loader loaded with size `6BE00`
   and complete-file FNV-1a `BB6AEFD2`.
4. **One stock-loader boot then reached NT.** It printed the kernel and HAL
   paths and returned the same generic `'osloader'\hal.dll` failure. No
   `CLAIM failed`, HAL initialization, VAL1 output or Beep STOP appears in
   that capture. The exact earlier failed firmware claim is not isolated.
5. **The outstanding LDRRS1 measurement was prepared again.** GREEN references
   passed their expected size/fingerprints and unchanged-byte comparisons.
   A fresh target read matched stock exactly before writing. Diagnostic
   deployment then returned write1/read1/full candidate comparison0 and
   bytes `00 00 E4 63`. The actual named installed loader loaded at3E00000,
   size6BE00, full-file FNV `08DC483A` (OF prints `8DC483A`).

See the observed [stock-loader HAL rejection](../traces/2026-11-03-stock-loader-hal-rejection.log)
and [corrected firmware readback](../traces/2026-11-03-nt-settings-corrected.log).

## What the diagnostic measures

Stock PPC OSLOADER emits the displayed HAL error for a nonzero HAL
`BlLoadImage` return, before import binding. LDRRS1 changes one instruction
on that existing failure path, at file offset0xBCC / preferred VA0x806007CC:
`li r4,0x233C` becomes `ori r4,r31,0`. Actual little-endian bytes change
`3C 23 80 38` to `00 00 E4 63`. `r31` holds the native return value.

The existing missing-message fallback prints that value in hexadecimal.
A return equal to a message-resource ID can display text instead, so the
complete message must be preserved. The success branch bypasses the patch.
Native reads, image checks, relocation, acceptance decisions and final return
remain unchanged. No new HAL build or HAL write accompanies this test.

**A status of4 would be broad image-load rejection, not proof of checksum
failure.** No status value is claimed at this checkpoint.

## What remains unresolved

The older HALRS1 physical boot failed at `Beep.SYS`, STOP `C000026C` with
underlying `C0000221`. SCSI recovery observations included requests covering
Beep's disk range, but final NT read completion, bytes delivered to validation,
and a causal connection to the image failure remain unproved. A firmware Beep
read matched its reference; this does not establish NT's own runtime read bytes.

HALV1 was prepared to observe the actual validation boundary, but its earlier
loader rejection prevents that probe from running. HALDS1 remains deferred:
measuring preceding DIP/SIP overlap would still leave the validation failure
itself unresolved. No root-cause fix or completed working NT boot is claimed.

## One boot, then restore

Capture the complete LDRRS1 boot output. After that one boot, use a fresh OF
session and reload GREEN recovery references. Fresh-read the target before
any restoration write:

- Matches diagnostic: restore verified original and verify complete readback.
- Already matches original: omit the write and verify the complete stock file.
- Matches neither: stop and preserve all512 bytes for analysis.

Restoration requires original-sector comparison0, named-loader size6BE00 and
stock FNV `BB6AEFD2`. No second diagnostic boot precedes restoration. The last
verified installed HAL remains HALV1/D600; HALRS1 and HALDS1 stay frozen.
FNV values are repeatable, noncryptographic identity evidence.

The observed FAT chain5→6→7→8→9→A maps the changed file sector to HDD LBA0x1229
on this specific system. That is not a universal patch address. Only four
bytes differ; all other508 bytes of the original/diagnostic sectors match.

## Follow the work

- [Short cold-start procedure](ANS-STARTUP.md): saved38400/8N1/no-flow-control
  connection, current disk, power on, prompt, hex, then the selected test.
- [BLUE and GREEN file-backup downloads](../archives/floppies/2026-11-03/README.md),
  with source dates, manifests and SHA-256 hashes.
- [October2 investigation checkpoint](ANS-HARDWARE-PROGRESS-2026-10-02.md)
  for the earlier validation-path reasoning and artifact fingerprints.

The downloads are saved file archives, not captured floppy images or verified
bootable clones. In particular, BLUE's boot script relies on raw-sector layout
which ordinary file copying does not reproduce. The source tree's original
emulator milestone and these experimental file backups are separate snapshots.
