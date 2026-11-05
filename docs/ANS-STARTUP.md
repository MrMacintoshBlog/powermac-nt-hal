# Real ANS 700 startup — recovered HALIM2 checkpoint

Updated November 5, 2026. The [handoff](ANS-HANDOFF-2026-11-05.md) records
the current binary, source, disk downloads, outstanding check and captures.

## Routine boot

1. Open the saved ANS serial connection at **38400 baud, 8N1, no flow control**.
2. Insert BLUE, power on, and wait for Open Firmware `0 >`.
3. After a previous boot or scratch session, send `reset-all` separately
   and wait for the fresh banner and prompt.
4. Paste the whole boot line once.

```forth
hex " load fd:,\bootdisk.of" eval load-base loadsize eval
```

Expected HALIM2 result is NPFS / `C000026C` / `C0000185`, SWEEP20/20,
I/O121 and SCSI ring426. Capture through `CRASH1 END`.

The quoted eval on one physical line lets the console receive the full
line before disk activity. Earlier multiline pastes after `load` repeatedly
lost characters. Keep resets separate from subsequent commands.

## Saved configuration

| Variable | Verified current setting |
|---|---|
| `input-device` | `ttya:38400` |
| `output-device` | `ttya:38400` |
| `auto-boot?` | `false` |
| `real-base` | `3F00000` |
| `load-base` / `loadaddr` | `3E00000` |
| `real-mode?` | `false` |
| `little-endian?` | `true` |
| `virt-base` | `-1` |

These settings were measured and repaired during the recorded sessions.
Use `printenv` to inspect actual current values if defaults have returned
or a fresh boot shows configuration problems. Defaults such as load-base
`4000` caused earlier invalid transfers and claim failures.

If the prompt appears on the ANS monitor, the recorded temporary serial
selection is `" ttya:38400" io`. The persistent setup uses the input/output
values above and a separate reset. The boot script supplies the ARC
environment for the HDD boot through `/options` on each run.

## Installation geometry and recovery

The HDD is `/bandit/53c825@12/sd@0,0`. The primary system partition starts
at1000. The extended partition starts at11000 and contains the NT logical
volume at11020. The verified HAL allocation is172A/count6B, sizeD600.
The recorded one-disk hardware configuration is the baseline.

The current installed HALIM2 FNV is085AE99B. The handoff provides recovery
source identity, prior readback evidence, and the guarded helper. The
outstanding NPFS measurement is read-only and is described there.
