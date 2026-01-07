# Hex Counter

A hex counter program that displays values 00-FF on the two-digit hex display with a controllable delay between counts.

## Program Versions

| File | Size | Description |
|------|------|-------------|
| `hexcounter.txt` | 31 bytes | Original with fixed delay (0x5F) |
| `hexcounter_switch_delay.txt` | 31 bytes | Switch-controlled delay |
| `hexcounter_switch_delay_short.txt` | 30 bytes | Switch-controlled, optimized |
| `hexcounter_no_preamble.txt` | 29 bytes | Fixed delay, preamble removed |
| `hexcounter_switch_no_preamble.txt` | 28 bytes | Switch-controlled, no preamble |
| `hexcounter_efficient.txt` | 22 bytes | Optimized with restructured delay loop |
| `hexcounter_minimal.txt` | 20 bytes | Most compact version |

## How It Works

See [`hexcounter_explained.md`](hexcounter_explained.md) for a detailed line-by-line walkthrough, written for programmers new to assembly/machine code.

## Quick Start

The simplest version to try is `hexcounter.txt` (31 bytes):

```
64 00 90 B2 B3 A3 F8 30 A2 52 E2 30 16 13 83 52 64 22 C4 C4 30 0B F8 5F B4 32 0D 94 24 30 19
```

To control speed with the toggle switches, use `hexcounter_switch_delay.txt`:

```
64 00 90 B2 B3 A3 F8 30 A2 52 E2 30 16 13 83 52 64 22 C4 C4 30 0B 6C C4 B4 32 0D 94 24 30 19
```

Set switches to FF for slowest counting, 01 for fastest.

## Diagnostic Files

These were created during development/debugging:

- `test_switch_read.txt` - Simple loop that reads switches and displays their value
- `hexcounter_inp_diagnostic.txt` - Displays INP 4 value instead of counter
- `DEBUG_SWITCH_DELAY.md` - Notes on debugging the switch-controlled version

## Adjusting the Delay

In the fixed-delay versions, look for the byte `5F` in the hex dump - this is the delay value. Change it to:
- `FF` for slowest
- `5F` for medium (default)
- `01` for fastest
- `00` to skip delay entirely
