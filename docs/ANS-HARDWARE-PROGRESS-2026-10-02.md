# Apple Network Server / Windows NT 4.0 hardware progress

Checkpoint: 2026-10-02. Physical testing and serial captures by Mr. Macintosh
([MrMacintoshBlog](https://github.com/MrMacintoshBlog)).

**The current investigation has reached physical installed-system boot testing,
but a working NT boot has not been established.** A temporary loader diagnostic
is installed and verified. Its first boot-script evaluation stopped in Open
Firmware with `CLAIM failed`, before any captured NT startup or diagnostic
status. Stock-loader restoration remains pending as part of this experiment.

## Observed hardware results

| Stage | Observation | What it establishes |
|---|---|---|
| HALRS1 installed-system boot | STOP `C000026C`, `Beep.SYS`, underlying `C0000221` | The original image-validation failure persists. |
| HALRS1 SCSI diagnostics | Active requests include Beep's disk range; retained recovery observations include cached SIST0 `90` and DSPS low byte `08` | Native recovery activity overlaps requests covering Beep. Its cause and effect on the final image remain unproved. |
| Firmware Beep reference check | Read at LBA `0x4C135`, count `0xD`, returned `0xD`; all `0x1970` bytes compared equal to the reference | The bytes obtained through this firmware read match the reference. It does not establish the bytes returned by NT's own read path. |
| HALV1 physical boot | NT loader reported missing/corrupt `'osloader'\hal.dll`; no HAL initialization or VAL1 output captured | The validation probe did not execute. The native HAL image-load failure status is still unknown. |
| HALV1 follow-up checks | Fresh raw disk and named FAT-file loads compared equal over `0xD600` bytes; the stock PPC loader checksum model accepts the correct file | Neither the generic loader message nor these checks identifies the failing native image-load stage. |
| LDRRS1 deployment | One sector written, fresh sector read back, complete 512-byte comparison equal; named loader size `0x6BE00`, full-file FNV-1a `08DC483A` | The deployed sector matches the reviewed diagnostic, and the complete named loader has its expected size/fingerprint. FNV is noncryptographic evidence. |
| LDRRS1 boot-script evaluation | `CLAIM failed`, then firmware prompt; no NT startup output | No native HAL status or NT diagnostic boot outcome has been obtained. |

The HALRS1 capture retained 11 completed, origin-valid, status-valid bulk
records without logger fault or overflow. Seven records at native origin
`0x3ACC` contain cached SIST0 `0x90`; four at `0x281C` contain cached DSPS low
byte `0x08`. These are recovery-path observations, not final IRP completion
results or proof that a reset caused `C0000221`.

## Why image validation remains the focus

The exact PPC image-validation path can produce `C0000221` following a failed
checksum-validation result or a caught exception. That status alone therefore
does not prove a checksum mismatch. Incorrect installed bytes, changed loaded
bytes, failed/incomplete reads, and image-handling faults remain hypotheses.

HALV1 was prepared to observe the actual validation boundary, but its loader
rejection occurs earlier. In the stock PPC `OSLOADER.EXE`, the displayed HAL
message is the generic error path for a nonzero HAL `BlLoadImage` return,
before import binding. The next useful observation is that native return
value. Status `4`, if obtained, would identify broad image-load rejection;
it would not uniquely identify a checksum failure.

HALDS1, a diagnostic of preceding DIP/SIP overlap, remains deferred: either
overlap result would still leave the actual image-validation failure unresolved.
No interrupt-priority change or reset suppression is justified by these results.

## Minimal loader-status diagnostic

LDRRS1 changes one instruction on the existing HAL-load failure path:

| Item | Value |
|---|---|
| Stock loader file offset | `0xBCC` |
| Preferred virtual address | `0x806007CC` |
| Original instruction | `li r4,0x233C` |
| Replacement | `ori r4,r31,0` |
| Actual little-endian bytes | `3C 23 80 38` → `00 00 E4 63` |

`r31` holds the native `BlLoadImage` return. Passing it as the existing second
error-display argument makes the missing-message fallback print the status in
hexadecimal. A status that matches a message-resource ID can resolve to text
instead. Successful image loading bypasses the changed instruction. Native
reads, validation decisions, relocation and final returned status are unchanged.

The current FAT16 allocation was read rather than assumed contiguous:
live loader start cluster `5`, observed chain `5 → 6 → 7 → 8 → 9 → A`, data
start `0x1221`, one sector per cluster. File-sector index 5 therefore maps to
**LBA `0x1229`**, and the instruction lies at sector offset `0x1CC`.

The complete original sector was captured and saved. Only its four instruction
bytes differ in the diagnostic sector; the other 508 bytes are identical.
The immediate pre-write read matched the original. Deployment then returned
write `1`, fresh read `1`, comparison `0`, with the diagnostic bytes present.

## Recovery and remaining work

GREEN is dedicated FAT12 recovery media containing `OSLOLD.SCT` and
`OSLNEW.SCT`, each 512 bytes. Mac readback after physical reinsertion passed.
Both independently loaded ANS files were captured as complete 512-byte dumps
and compared byte-for-byte with their saved sources. The original recovery
load followed a fresh firmware session; restoration does not depend on RAM
surviving the diagnostic boot. BLUE retains the established boot script.

The latest firmware failure is recorded in
[`../traces/2026-10-02-ldrrs1-claim-failed.log`](../traces/2026-10-02-ldrrs1-claim-failed.log).
Its exact failing memory claim is unknown. The next action is a demonstrated
fresh firmware reset and script reload before another evaluation. If the
claim failure persists in a fresh session, stop the boot attempt and restore
the stock loader through GREEN.

The experiment permits exactly one NT diagnostic boot, capturing all serial
output. Afterward, a fresh firmware session must reload the recovery files and
read the target sector **before** any restoration write:

- Diagnostic sector matches: restore the verified original and verify readback.
- Original already matches: omit the write and verify the complete stock loader.
- Neither matches: stop and preserve the complete sector for analysis.

Stock verification requires original-sector equality, loader size `0x6BE00`
and full-file FNV `BB6AEFD2`. No second diagnostic boot precedes restoration.
HALV1 remains installed at `0xD600` bytes; HALRS1, HALDS1 and the stock loader
artifacts remain frozen. This checkpoint establishes no repair and no causal
connection between SCSI recovery and Beep's image-validation failure.

## Artifact fingerprints

These identify locally preserved artifacts; this progress update contains
documentation and an observed serial trace.

| Artifact | SHA-256 |
|---|---|
| Stock PPC OSLOADER | `a1bf1defbafa02f949c8a876856f10d6af25c73001a02081fae4e35c9120573a` |
| LDRRS1 | `c04e0767d349451372fd3d302d8b31257e04d1bddc7a0e2fc674a9e6f2b8841c` |
| Physical original sector | `5a8545691c158cadbaa26f8613124050cec0f938c8402855feb13d7a5fc2c052` |
| Diagnostic sector | `efde44a89be228b2eb2b646d27eab19cdc60e7f937f5cf5fb54a55cbb7bf8d14` |
| HALRS1 | `92948e031626252109f73f82192d4650529975f1ad35326379a855bfd831f499` |
| HALV1 | `c7ad0c254faf580563fc3c672c4afb045b3aa49ae234346574a7721351b2d076` |
| Deferred HALDS1 | `980f1245983b007f45e726ccdfebe641761b98e7f898dd4ec411d33f1d695d31` |
