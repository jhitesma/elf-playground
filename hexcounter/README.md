# Hex Counter

A hex counter program that displays values 00-FF on the two-digit hex display with a controllable delay between counts.

## Program Versions

| File | Size | Description |
|------|------|-------------|
| `hexcounter.txt` | 31 bytes | Original with fixed delay (0x5F) |
| `hexcounter_switch_delay.txt` | 31 bytes | Switch-controlled delay (PE ELF) |
| `hexcounter_switch_delay_short.txt` | 30 bytes | Switch-controlled, optimized |
| `hexcounter_keypad_delay.txt` | 54 bytes | Keypad-controlled delay (AVI ELF II) |
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

## AVI ELF II Keypad Version

For the AVI ELF II with hex keypad, use `hexcounter_keypad_delay.txt`:

```
64 00 90 B2 B3 A3 B6 F8 30 A2 52 E2 F8 80 B4 3C 23 94 B5 32 1B 95 25 32 1B 30 15 13 83 52 64 22 C4 30 0F 6C 22 B7 96 52 97 F3 32 31 97 B6 30 11 C4 97 B4 B6 30 11
```

**How it works:**
The program detects the IN button by comparing keypad values:
- Hex keypress changes the buffer → stored as "pending" (not applied yet)
- IN button doesn't change buffer → value is applied to delay

**How to use:**
1. Load and run - counter starts at medium speed (0x80 delay)
2. Press two hex keys to set new delay (e.g., "F" then "F" for 0xFF)
3. Press **IN button** to apply the new delay
4. Repeat steps 2-3 anytime to change speed

**Delay examples:**
| Keys | Value | Speed |
|------|-------|-------|
| F, F | 0xFF | Slowest |
| 8, 0 | 0x80 | Medium |
| 1, 0 | 0x10 | Fast |
| 0, 1 | 0x01 | Fastest |

The keypad uses a rolling buffer (74C923 + 74C173): each keypress shifts the previous key to the high nibble and puts the new key in the low nibble.

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
