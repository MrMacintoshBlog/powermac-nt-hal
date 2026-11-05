# HALIM2 / NPFS investigation reassessment

Reconciled November 5, 2026, at the user's request.

Continue from the recovered HALIM2 build and investigate the NPFS image-load
failure. First close the complete-file identity gap with a read-only firmware
check. If that file matches its reference, the next instrumented experiment
should identify which native image-loading stage returns the error. A further
Beep-only trace or SCSI interrupt experiment would leave that question
unanswered at this checkpoint.

## Confirmed installation and boot progress

The September 18 hardware ledger
records completed text-mode Setup on the real ANS 700, including formatting,
file copy, and the successful restart message. The September 23 carryover
also records a fresh text-mode install after the DMA fix. The remaining
installation milestone is GUI Setup after the reboot.

HALIM2's saved artifact matches SHA-256
`d2fb635ae4760ffe394dafa80e41fe6e06e60b3b7b2725316160fb16a5d89c8f`,
size `D600`, FNV `085AE99B`. The current restoration has source identity,
complete raw readback, and named-file comparison `0` in the user's transcript.

The complete CRASH1 blocks in these four captures all equal the October 1
recovery block byte for byte:

- `boot-1.log`
- `boot-2.log`
- `work3-recovery-boot.log`
- `work3-recovery-repeat.log`

All reach NPFS / `C000026C` / `C0000185`, SWEEP20/20, I/O121, ring426.
The WORK #3 recovery is our farthest verified physical installed-system boot
checkpoint. The last repeat starts mid-boot and contains the complete crash
block. These are comparisons of submitted captures, with the installation
identity established separately by the restoration transcript.

## What the storage trace establishes

The retained HALIM2 ring reports these HDD READ(10) completions with
SRB status `01` and SCSI status `00`:

| Sequence | Absolute LBA | Blocks | Reported transfer length |
|---|---|---|---|
| 395 | `2E535` | `40` | `8000` |
| 398 | `2E575` | `40` | `8000` |
| 401 | `2E5B5` | `16` | `2C00` |

These cover the recorded NPFS file allocation, including the final sector.
The complete-file size from the historical directory entry is `12A50`.
The old inference from NPFSTRACE1 silence that NPFS data reads never reach
the miniport is superseded for this baseline.

The logger observes the miniport notification before forwarding it to
SCSIPORT. It does not capture the final IRP status or verify the bytes
delivered to the image loader. Bulk `0E` records likewise report calls to
complete requests with BUS_RESET, without enumerating the requests actually
affected. Successful individual completions and nearby bulk reset calls
therefore cannot alone explain the driver's final error.

The historical firmware check sampled three NPFS sectors. A current full-file
comparison is still unestablished. The stronger full-file Beep checks belong
to a different file and diagnostic build.

The archived NPFS media reference at `diagnostics/nt-media/expanded/npfs.sys`
has size `12A50`, FNV `80C8D029`, and SHA-256
`4d15a5a8ecbca1ff8a619c75605a2e9cded59715cb9559e9670a95a08b6b178b`.
Its stored PE checksum `1F852` matches an independently recomputed checksum.
The next read-only check should load the actual named installed file, check
its complete size/fingerprint, and compare that buffer against a full raw
read after revalidating its directory entry and FAT chain. FNV is repeatable
identity evidence, not a cryptographic or byte-for-byte reference comparison.
An unexpected fingerprint requires checking the installed file's version and
source before selecting any repair. A full reference byte comparison can
then resolve a suspected mismatch.

The final two HDD reads (`2D5BF/20` and `2D627/1E`) fall outside the recorded
NPFS file span. Their owning files are unestablished. Do not label every
request near the NPFS stop as an NPFS file-data request.

## Native image-load boundary checked in this reassessment

The supplied PPC kernel remains SHA-256
`e91f40de2d608ebb86f1b6e172ca0d19ee228cdb18c5d988dee00f9bb458425f`.
Instruction analysis used its preferred base `80010000` and explicit
TOC `80018340`. Addresses below are RVAs. Its `.pdata` identifies the
image-loading routine as preferred `800D802C..800D88A8`.

That routine opens the file, calls the existing validation helper, creates
an image section, loads/maps it, handles relocation and imports, and builds
the module entry. It contains the native error path that wraps an underlying
failure as `C000026C` with the driver name.

| RVA | Native observation point | Relevant instruction |
|---|---|---|
| `C80CC` | File-open status | `ori r29,r3,0` |
| `C80E0` | Validation-helper status | `ori r29,r3,0` |
| `C8254` | Image-section creation status | `ori r29,r3,0` |
| `C8284` | Section-object reference status | `ori r29,r3,0` |
| `C8354` | Image loading helper status | `ori r29,r3,0` |
| `C83B4` | Relocation helper status | `ori r29,r3,0` |
| `C8400` | Import-resolution helper status | `ori r29,r3,0` |
| `C87B8` | Save underlying failure as hard-error parameter 2 | `stw r29,4(r30)` |
| `C87C4..C87C8` | Set outer hard-error status | `lis r29,C000` / `ori r29,r29,026C` |

Thirteen selected instruction words and eight direct-call destinations were
checked against the exact kernel file. The system-call stub at `37B74`
selects service `4F`, matching the exported ZwOpenFile descriptor. The stub
at `37A04` selects service `21`, matching ZwCreateSection.

The file-open negative branch at `C80D4` goes directly to cleanup `C8880`.
Later failures can reach `C872C` and the hard-error wrapper. The validation
helper's return is explicitly checked for `C0000221` and `C0000249` before
the loader continues, so its result must be logged separately from the
subsequent image-section status. The NPFS error name identifies the image
being loaded. This code inspection points to image loading before NPFS's
own initialization. The exact failing stage still needs a runtime observation.

The kernel file and static sites are available, but these new windows have
not been fingerprinted on the running machine. Runtime identity checks and
PPC ABI validation are prerequisites for any probe using them.

## Experiment selection

The complete-file read-only check comes first. If identity and raw/named
comparisons agree, the next candidate should record the image name, stage, returned NTSTATUS,
and final underlying loader error using bounded RAM records printed at STOP.
Build it from the recovered HALIM2 source and preserve the verified DMA,
cache, GPIO and driver fixes. Record results for all load attempts so an
earlier stopping point still supplies evidence. A file-name or Beep-LBA-only
filter could miss the new failure again.

| Measured outcome | Investigation it selects |
|---|---|
| Named file has an unexpected size/fingerprint, or differs from revalidated raw data | Resolve file identity, source version, and filesystem mapping before a diagnostic boot. |
| Image-section creation or image-copy/loading returns `C0000185` | Follow the associated file/paging IRP through filesystem, CLASS2 and SCSIPORT completion. |
| Validation helper returns an error | Separate its own section/map/query failure from checksum mismatch or a caught paging exception. |
| Relocation or import resolution fails | Inspect that image and the native loader's corresponding inputs and exception path. |
| The loader succeeds and a later driver-initialization failure is observed | Move the observation to NPFS initialization using the measured transition. |
| Boot stops earlier than NPFS | Use that attempt's stage records and compare with the preserved HALIM2 baseline. |

This is an experiment specification, not a built or deployed candidate.
Before transfer, require a reproducible PPC build, exact native fingerprints,
preserved register/control state, the `D600` size gate, and a tested guarded
rollback from its own fingerprint to HALIM2. Add only the observation needed
to distinguish these outcomes.

## Upstream and process review

The known [upstream draft PR 1](https://github.com/pappadf/powermac-nt-hal/pull/1)
was re-read through GitHub. It remains a draft describing emulated ANS500
GUI Setup/desktop results, with a reported update date of September 18.
The connector returned no discussion comments. This bounded check did not
identify a published real-ANS700 NPFS fix to apply.

Two historical reporting mistakes contributed to the confusion. Completed
text-mode Setup was omitted from WORK #3 summaries, and diagnostic findings
were discussed without consistently separating them from the best boot
milestone. Both are corrected at the top of the current progress file.
Passing Beep in one build did not establish a permanent fix, and installing
another logger did not establish forward boot progress.

The recovered baseline and latest captures already answer reproducibility.
Current hardware state is sufficient for the offline probe design. First
complete the read-only NPFS file check, then select the instrumented boot
from the identity and stage questions above.

## Handoff update after this reassessment

The named-file check returned 3E00000 / 12A50 / 80C8D029 and passed. The
first raw-check guard incorrectly assumed a primary FAT16 volume at11020.
The saved physical MBR has extended type05 at11000, containing the logical
NT volume at11020. The corrected MBR/EBR-aware routine is published with
58 host stack/control-flow and mocked-read cases. It uses the captured
MBR bytes and a synthetic EBR fixture. Its corrected hardware result is
still pending. The current handoff supersedes this report's earlier plan
status where that read-only check has already advanced.
