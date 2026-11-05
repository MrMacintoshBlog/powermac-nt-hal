# Real ANS 700 continuation handoff — November 5, 2026

**Start here. The recovered HALIM2 configuration repeatedly reaches the
NPFS driver-load failure after completing text-mode Windows NT Setup.**
GUI Setup is the next installation milestone. The owner is handing the
investigation over from this documented checkpoint.

## Downloads

- [BLUE installed-system boot disk image](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/BLUE-handoff-2026-11-05.img.xz)
- [GREEN recovery/data disk image](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/GREEN-handoff-2026-11-05.img.xz)
- [Disk manifests, file archives and checksums](../archives/floppies/2026-11-05/README.md)
- [Recovered HALIM2 source and baseline DLL](https://github.com/MrMacintoshBlog/powermac-nt-hal/raw/refs/heads/main/archives/floppies/2026-11-05/HALIM2-source-2026-11-05.tar.xz)
- [Detailed current hardware progress](ANS-HARDWARE-PROGRESS-2026-11-05.md)
- [Investigation reassessment](ANS-REASSESSMENT-2026-11-05.md)

The downloadable images are complete 1,474,560-byte FAT12 images after
decompression. They were reconstructed from verified saved payloads and
the recorded transfers. Filesystem checks and full payload comparisons
passed offline. These newly reconstructed images await hardware testing.
They are distinct from fresh captures of the two physical floppies.

BLUE preserves the original CD veneer at raw floppy LBAs `21..15C` and
the installed-system veneer at `15D..298`, including the bytes used by
`BOOTDISK.OF`. GREEN supplies recovery data. The included NPFSDEFS helper
and GREEN's additional HALIM2 copy were added for this handoff.

The disk images provide the floppy boot/recovery files. Continuing the
current installation also requires the existing NT hard disk in the ANS.
An HDD image is outside this package. The hard-disk sector addresses below
belong to that recorded installation.

## Milestones already established

1. Text-mode Setup completed on the physical ANS 700 on September 18.
   VGA and ADB input worked, the installation files copied, and Setup
   displayed its successful restart message.
2. The installed-system boot subsequently reached kernel phase 1, input
   class connections and video-driver resource setup.
3. HALIM2 reached NPFS in September and October. Its November recovery and
   repeat have CRASH1 blocks identical to the October 1 recovery.
4. Several diagnostic builds stopped earlier at Beep or Sfloppy. Their
   diagnostic findings remain useful, but they did not advance the best
   physical boot milestone.
5. The current installed HAL is HALIM2. The whole named NPFS file has the
   expected size and reference fingerprint. The corrected raw comparison
   is the outstanding read-only measurement.

## Machine and current binary

| Item | Recorded value |
|---|---|
| Machine | Physical Apple Network Server 700, 64 MB parity RAM |
| Firmware | Open Firmware 2.26 NT-oriented ROM |
| Serial | 38400 baud, 8N1, no flow control |
| HDD | `/bandit/53c825@12/sd@0,0` |
| Floppy | `/bandit/gc/swim3` |
| Installed HAL | `\OS\WINNT40\HAL.DLL` on partition 1 |
| HALIM2 size / FNV | `D600` / `085AE99B` |
| HALIM2 SHA-256 | `d2fb635ae4760ffe394dafa80e41fe6e06e60b3b7b2725316160fb16a5d89c8f` |
| HAL allocation | Absolute HDD LBA `172A`, count `6B`, ending `1794` |
| Stock OSLOADER size / FNV | `6BE00` / `BB6AEFD2` |
| Primary system partition | FAT16 type `06`, start `1000`, length `10000` |
| Extended partition | MBR type `05`, start `11000` |
| Logical NT volume | FAT16 type `06`, start `11020` |

HALIM2 restoration passed complete raw readback and named-file byte
comparison `0`. A clean build of the handoff source with LLVM 23.1.1
reproduced the baseline DLL byte for byte during packaging. Source and
build instructions are inside the source archive. The root repository
source remains the original project snapshot. Use the frozen HALIM2
archive for this recovered hardware checkpoint.

## Reproduce the recovered boot

Use BLUE and the saved serial connection. After an earlier boot or scratch
session, send the reset separately and wait for the fresh firmware prompt.

```forth
reset-all
```

Then paste the entire boot line at `0 >`.

```forth
hex " load fd:,\bootdisk.of" eval load-base loadsize eval
```

The recovered checkpoint is `Npfs.SYS`, STOP `C000026C`, underlying
`C0000185`, SWEEP entered/completed `20/20`, last I/O `121`, and
SCSIRING2 count `426`. Capture through `CRASH1 END`.
The [recovery](../traces/2026-11-05-halim2-recovery.log) and
[repeat](../traces/2026-11-05-halim2-repeat.log) captures are provided.
Reproducibility is already established, so an additional identical boot
serves only a deliberate setup/recovery check.

## Resume the outstanding read-only check

The current named-file load returned `3E00000 12A50` and FNV `80C8D029`,
matching the archived NPFS media reference. The physical disk HAL check
still returned `85AE99B`.

The first raw check stopped at `Partition 2 differs`. That was a guard
implementation error. It expected a primary FAT16 entry at `11020` instead
of the documented extended partition at `11000`. The corrected checker
validates MBR, EBR and BPB in order. It has 58 host stack/control-flow and
mocked-read cases, using the archived physical MBR. The EBR test fixture
is synthetic. Its corrected hardware run has not been received.

On either newly reconstructed disk, NPFSDEFS.OF contains the corrected
definitions without an automatic invocation. At a fresh firmware prompt,
paste this entire physical line once.

```forth
hex " load fd:,\CENDCHK.OF" eval load-base loadsize eval " load fd:,\NPFSDEFS.OF" eval load-base loadsize eval " load /bandit/53c825@12/sd@0,0:2,\WINNT\system32\drivers\npfs.sys" eval np-raw
```

Expected final messages are `NPFS allocation verified 3A8 3A9` and
`NPFS full raw/named comparison 0`. Preserve the complete result. The
original physical disks contain CENDCHK, but NPFSDEFS was added only to
the new handoff sets. It can be copied from the file archive to original
media, or the [paste-ready checker](../tools/hardware/NPFSRAW.OF) can be
defined in the same session after the named NPFS load.

All disk operations in this check are reads. It traverses the live
WINNT/SYSTEM32/DRIVERS path, validates NPFS size/cluster and both FAT copies,
then compares all `12A50` bytes with the named RAM copy in five bounded
reads. A failed check should be investigated from its actual output.

## Investigation after the file check

The retained trace reports successful NPFS-range READ(10) notifications,
but lacks the final NT IRP status and byte verification at image loading.
Nearby BUS_RESET bulk-completion calls do not identify the requests affected.

If the whole-file and raw/named checks agree, the proposed next diagnostic
records the image name, native loading stage and returned NTSTATUS at STOP.
Candidate return sites for validation, image-section creation/loading and
import resolution are documented in the reassessment. That experiment still
requires implementation, runtime fingerprints, PPC ABI checks, a reproducible
build, and the `D600` allocation gate. Build from the recovered HALIM2 source.

The preserved Beep-focused HALV1/CLASSEND and deferred HALDS1 builds represent
earlier investigation branches. Their presence on media does not select
them as the current baseline.

## Recovery tools and history

CENDCHK evaluates a read-only HAL allocation check. Its generic `ce-replace`
word validates the loaded source base, size and fingerprint and the current
disk fingerprint before writing the confirmed HAL sectors and reading them
back. The separate `ce-verify` loads the named HAL and compares it with the
raw buffer. The successful CLASSEND-to-HALIM2 restore used source FNV
`085AE99B` and current-disk FNV `4A202505`.

The current machine already has HALIM2. Select a recovery write only from
freshly measured source/current identities. `ce-install` and `ce-restore`
specifically select the older CLASSEND/HALV1 pair. Their names do not mean
generic HALIM2 restoration.

GREEN also carries the saved OSLOADER sector references and the LDRECHK and
HDRCHK helpers. They relate to completed loader diagnostics, including the
AppleDouble-corrupted HAL repair. Stock OSLOADER has been restored. Any
future use of those write paths requires their own live identity guards.

For console and firmware settings, use the [startup guide](ANS-STARTUP.md).
