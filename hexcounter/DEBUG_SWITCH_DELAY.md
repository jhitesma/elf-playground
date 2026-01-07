# Debugging the Switch-Controlled Delay

## Resolution

**RESOLVED:** The switch-controlled delay works correctly on real hardware. The issue was with the emulator, not the code.

The emulator appears to have a bug with INP 4 (switch reading) that causes it to return incorrect values. On actual COSMAC ELF hardware, `hexcounter_switch_delay.txt` works as expected - switches control the counting speed.

## Working Versions

All of these work on real hardware:

| File | Size | Description |
|------|------|-------------|
| hexcounter.txt | 31 bytes | Original with fixed delay (0x5F) |
| hexcounter_switch_delay.txt | 31 bytes | Switch-controlled delay |
| hexcounter_switch_delay_short.txt | 30 bytes | Switch-controlled, optimized |
| hexcounter_no_preamble.txt | 29 bytes | Fixed delay, no preamble |
| hexcounter_switch_no_preamble.txt | 28 bytes | Switch-controlled, no preamble |
| hexcounter_efficient.txt | 22 bytes | Optimized fixed delay |
| hexcounter_minimal.txt | 20 bytes | Most compact version |

## Diagnostic Files (No Longer Needed)

These were created for debugging but can be kept for reference:

- `test_switch_read.txt` - Simple INP 4 test loop
- `hexcounter_inp_diagnostic.txt` - Displays INP value instead of counter
- `hexcounter_switch_with_nops.txt` - Version with timing NOPs (unnecessary)

## Emulator Bug Notes

If testing in an emulator and INP 4 doesn't work:
- The code is correct - test on real hardware if possible
- The emulator may not properly simulate the N2 line / switch multiplexing
- Some emulators may require specific configuration for I/O ports
