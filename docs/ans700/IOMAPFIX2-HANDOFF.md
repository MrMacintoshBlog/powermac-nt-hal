# ANS 700 NT4 PPC — IOMAPFIX2 source-review handoff

## Scope
Regular Chat source review only. No HAL was built or proposed for deployment. WORK retains responsibility for applying this patch to the exact reproducible `hal-scsiring2` baseline, doing the PowerPC build, and enforcing the final `0xD600` allocation gate.

Preserve unchanged: SCSIRING2, SWEEP1/cache behavior, GPIOCMP1, CRASH1, and the in-place `D600` deployment allocation.

## Why IOMAPFIX1 was revised
The supplied static disassembly of stock NT4 PPC `SCSIPORT.SYS` shows its scatter/gather loop advances the SG output pointer even when returned mapped length is zero, while accumulated progress remains unchanged and the loop continues. Therefore returning `Length=0` for an invalid **nonzero** MapTransfer request is not a safe error path for this caller.

The MapTransfer interface itself returns only a logical address plus an in/out byte count; there is no separate status result. For the observed caller, the bounded diagnostic-safe response to malformed nonzero input is therefore a non-returning HAL diagnostic stop, not fabricated progress and not an invented DMA address.

## IOMAPFIX2 changes
Patch: `SCSIRING2-IOMAPFIX2.patch`  
SHA256: `bc3fcd17b9ff7d4d202b55de086db38310476154226f7731f0805cf83993ad6f`

Exact patched function text: `IoMapTransfer-IOMAPFIX2.c.txt`  
SHA256: `78c67d1f55118024a5c2f36ace6b1b0da9c0314c70fa8bf5cf8057587701838b`

Changes are confined to `IoMapTransfer` in `src/dma.c`:

1. A genuine caller-requested `*Length == 0` returns `{logical=0,length=0}` before PFN access.
2. Invalid MDL geometry on a nonzero request (`ByteOffset >= 4096`, zero `ByteCount`, or `ByteOffset + ByteCount` overflow) calls `KeBugCheckEx(0xE2, 0x494D0001, Mdl, ByteOffset, ByteCount)`.
3. Canonical MapTransfer indexing accepts only the exclusive range `[StartVa + ByteOffset, StartVa + ByteOffset + ByteCount)`.
4. `MappedSystemVa` is considered only when `MdlFlags & 0x0001` (`MDL_MAPPED_TO_SYSTEM_VA`) is set and the pointer is non-null. A non-null field without the flag is not trusted.
5. An invalid nonzero address after those checks calls `KeBugCheckEx(0xE2, 0x494D0002, Mdl, CurrentVa, requested)`. It never returns zero progress, fabricates an address, or reports fake success.
6. The existing physical-contiguity rule is preserved. Arithmetic is widened only where needed:
   - `npages = (((ULONGLONG)limit + 4095) >> 12)` prevents 32-bit `(limit + 4095)` wrap;
   - `(run + inpage)` is also widened before deriving the next PFN index;
   - mapping growth is capped by `maxRun = min(requested,total)` so the final partial page cannot wrap `run`.
7. `HalFlushIoBuffers` is byte-identical to the recovered baseline; no cache or logging behavior changes were made.

## MDL semantics checked
Project source defines the NT4 PPC MDL as `StartVa`, `MappedSystemVa`, `ByteCount`, `ByteOffset`, and uses bit `0x0001` as `MDL_MAPPED_TO_SYSTEM_VA` elsewhere in `dma.c`.

The standard MapTransfer DDI describes `CurrentVa` as the MDL index obtained from `MmGetMdlVirtualAddress` and advanced by completed transfer length. This supports `StartVa + ByteOffset` as the canonical path. Modern Microsoft documentation is corroborating DDI evidence, not proof of this exact NT4 PPC binary.

### Remaining `MappedSystemVa` assumption
The fallback assumes that when `MDL_MAPPED_TO_SYSTEM_VA` is set, `MappedSystemVa` denotes the first described buffer byte, so mapped-relative progress is translated back to the PFN-array offset by adding `ByteOffset`. This is consistent with classic system-address-for-MDL usage, but the exact NT4 PPC mapping implementation has not been independently recovered here. The fallback is defensive; normal SCSIPORT MapTransfer traffic should use the canonical `MmGetMdlVirtualAddress` index.

If WORK has stock NT4 PPC kernel source/disassembly proving a different offset convention for `MappedSystemVa`, adjust only that fallback before build.

## Actual C execution tests
The harness `iomapfix2_c_harness.c` was mechanically generated with the exact patched `IoMapTransfer` text embedded unchanged. It was compiled and executed on the host under AddressSanitizer and UndefinedBehaviorSanitizer. Expected results are hard-coded independently in the harness.

Result: **18/18 passed**.

Covered:
- canonical first and subsequent mappings;
- first-page byte offset handling;
- requested-length clipping;
- non-contiguous PFN stopping;
- mapped-VA fallback with the required flag;
- rejection when mapped pointer exists without the flag;
- exclusive-end, before-range, beyond-range cases;
- true zero requested length;
- zero byte count, invalid byte offset, and limit overflow diagnostic stops;
- current VA below both bases diagnostic stop;
- final valid byte;
- `limit == UINT32_MAX` / final-PFN case, directly exercising the old `limit+4095` overflow boundary.

C test results: `IOMAPFIX2-C-test-results.txt`  
SHA256: `a3505e4bea8bb0cf853174589393ba2bd123a01aadb35f76cfac7221d049b484`

No ASan/UBSan runtime failure occurred. Host compilation emits expected pointer-to-32-bit-cast warnings because the real target is 32-bit PPC while the execution harness is a 64-bit host. Separately, the patched project `src/dma.c` passes PowerPC-targeted `clang -fsyntax-only` with the project flags and no diagnostics. This is syntax/front-end validation only, not a HAL build.

## Python model tests
A separate Python arithmetic model was updated for the new fail-stop semantics and flag gating. It is not treated as execution of the C.

Result: **18/18 passed** with independently specified expected results.

Python results: `IOMAPFIX2-Python-test-results.txt`  
SHA256: `87de5e109ca25e1f80ebd913be1f900c8342a767fce842d6ff614043d657629b`

## Preservation evidence
Recovered SWEEP1 source audit: only `src/dma.c` was edited for this review; one diff hunk. `HalFlushIoBuffers` is byte-identical. SCSI trace/thunk, CRASH1 and init files remain unchanged in the recovered tree.

Preservation report: `IOMAPFIX2-preservation.txt`  
SHA256: `8597861541feea2cd085e14995b879a14ce1de9ad873489d8924110499c0b1fc`

WORK must repeat this preservation check against the **actual byte-reproducible SCSIRING2 tree**, because the conversation archive does not contain the exact three SCSIRING2-modified source files.

## Caller handling conclusion
Given the supplied stock `SCSIPORT.SYS` loop, zero progress for a nonzero request is unsafe. No genuine error-return channel was identified in the MapTransfer interface or that loop. IOMAPFIX2 therefore uses the HAL's already-existing non-returning `KeBugCheckEx` facility for impossible/malformed nonzero inputs.

This is intentionally diagnostic fail-stop behavior. It is not a claim that malformed input is occurring on the ANS today.

## WORK integration gate
WORK should:
1. Start from the byte-reproducible `hal-scsiring2` tree that regenerates known `HALSR2.DLL`.
2. Apply only `SCSIRING2-IOMAPFIX2.patch` to `src/dma.c`.
3. Confirm no other source change and unchanged SCSIRING2/SWEEP1-cache/GPIOCMP1/CRASH1 fingerprints.
4. Build with the original reproducible PowerPC toolchain.
5. Require the raw HAL to fit the established allocation; never truncate.
6. Produce/restamp the final transfer image exactly `0xD600`, verify PE checksum, and record SHA256.
7. Do not deploy if the candidate exceeds `D600` or source preservation fails.

## Next hardware test
If and only if the integrated candidate passes that gate, run **one** boot of `SCSIRING2 + IOMAPFIX2` with no other changes.

Capture the normal SCSIRING2/CRASH1 output plus any intentional IOMAP diagnostic stop.

Interpretation:
- `E2 / 494D0001`: malformed MDL geometry actually reached IoMapTransfer; record the supplied MDL/offset/count and stop broad diagnosis.
- `E2 / 494D0002`: a nonzero CurrentVa outside the canonical range and valid flagged mapping actually reached IoMapTransfer; record MDL/current/request and inspect that caller.
- No IOMAP stop + same Beep/C0000221: the patch did not demonstrate activation; proceed to CHECK1 rather than claiming a DMA fix.
- No IOMAP stop + improved boot: record as a timing/layout-correlated result only. **One improved boot does not establish causality.** Repeatability and evidence of changed path would still be required.
- IOMAP stop absent but reset pattern changes: treat as comparative evidence, not proof that this source correction caused the previous resets.

## Unresolved assumptions / limits
- No evidence currently proves that the invalid-address path activates in the real ANS failure.
- The supplied SCSIPORT disassembly establishes the zero-progress hazard statically, not that this loop encountered malformed input on hardware.
- Exact NT4 PPC `MappedSystemVa` byte-offset convention remains unverified here; the flag gate is verified, the fallback convention is an explicitly documented assumption.
- Host C tests validate source behavior and memory safety in the harness; they do not validate PPC ABI, generated machine code, timing, or final HAL layout.
- No deployable HAL or candidate HAL hash exists from this Chat review by design.
- Nothing here establishes that IOMAPFIX2 explains `Beep.SYS / C0000221`.