# NPFS file check — received results and remaining step

The following are excerpts from the user's WORK #3 serial output, published
November 5, 2026. The command echoes and checker definitions are omitted.

## Complete named file

```text
HAL allocation verified. Disk FNV 85AE99B
/bandit/53c825@12/sd@0,0:2,\WINNT\system32\drivers\npfs.sys  3E00000 12A50
80C8D029
ok
```

This covers the complete named file and matches the archived media
reference size/FNV. The installed HAL check remains HALIM2.

## First raw-check invocation

```text
0 > np-raw Partition 2 differs
NPFS raw check stopped
ok
0 >
```

This stopped before NPFS payload comparisons. The original checker used
an incorrect primary-partition guard. The archived physical MBR instead
shows extended type05 at11000, with the logical FAT16 volume at11020.

The published NPFSRAW.OF corrects that guard and validates the intervening
EBR. The corrected routine passed 58 host stack/control-flow and mocked-read
cases. Its corrected hardware output has not been received.

## Next result to collect

```text
NPFS allocation verified 3A8 3A9
NPFS full raw/named comparison 0
```

These last two lines are expected output, not an observed result. Use the
current handoff's read-only command to collect the actual output.
