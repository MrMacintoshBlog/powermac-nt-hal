# ANS cold startup — short working procedure

Saved working serial profile: **38400 baud, 8 data bits, no parity, 1 stop
bit, no flow control**. Open the ANS USB serial adapter connection and start
logging before power-on.

1. Insert the disk selected by the current test. BLUE provides BOOTDISK.OF;
   GREEN holds recovery references. Keep GREEN available during diagnostics.
2. Power on and wait for OpenFirmware2.26 / `0 >`. Press Return once if quiet.
3. Enter `hex` and continue directly with the selected test.

The normal route uses the previously confirmed saved firmware setup. It does
not require a routine settings dump, repeated setenv, temporary io redirect,
memory map or extra reset. File identity and disk-write/readback checks belong
to the individual test and remain required there.

When resuming a session already used by a boot/partial script, reset-all alone
and wait for a fresh prompt before setting up that test. A clean cold-power-on
session skips that reset. BOOTDISK makes its own fixed-address claims; reset
between a file-check session that mapped3E00000/80000 and a BOOTDISK run.

## Persistent console setup — one-time repair

At an accessible OF console, only when establishing/repairing saved routing:

```forth
setenv auto-boot? false
setenv input-device ttya:38400
setenv output-device ttya:38400
reset-all
```

Keep Serial.app open at38400 and wait for the serial prompt. These saved
settings govern Open Firmware input/output across ordinary reboots and cold
starts. NT/HAL diagnostic logging uses the existing separate boot/HAL path.

## Troubleshooting snapshot — only after unexpected behavior or settings changes

```forth
hex
printenv input-device
printenv output-device
printenv auto-boot?
printenv real-base
printenv load-base
printenv real-mode?
printenv little-endian?
printenv virt-base
load-base .
loadaddr .
```

Working current values, reverified November3: input/outputttya:38400,
auto-bootfalse, real-base3F00000, load-base/loadaddr3E00000, real-modefalse,
little-endiantrue, virt-base-1. The other printenv column is firmware defaults.
Unexpected settings or load addresses require review before memory copies or
writes. Previous defaults were observed and corrected; their cause is unknown.

Current pending experiment: [one LDRRS1 native-status boot and stock restoration](ANS-HARDWARE-PROGRESS-2026-11-03.md).
Consult its current checkpoint before using archived test files. BLUE's saved
file archive is not a replacement for its known-good raw boot-media layout.
