# Binary Clock

A real-time clock for the COSMAC ELF that displays hours and minutes on the hex display.

## Features

- Display alternates between hours and minutes every 2 seconds
- Q LED indicates what's being displayed: ON = hours, OFF = minutes
- Available in 12-hour (1-12) and 24-hour (0-23) formats
- Optional time-setting mode using switches and INPUT button
- Calibrated for 1 MHz crystal oscillator

## Program Versions

| File | Size | Description |
|------|------|-------------|
| `clock_12h.txt` | 78 bytes | 12-hour, starts at 12:00 |
| `clock_12h_set.txt` | 104 bytes | 12-hour with time setting |
| `clock_24h.txt` | 76 bytes | 24-hour, starts at 00:00 |
| `clock_24h_set.txt` | 102 bytes | 24-hour with time setting |

## Display Format

The ELF's 2-digit hex display shows one value at a time:

**12-Hour Mode:**
- Hours: `01` - `0C` (1-12 in hex)
- Minutes: `00` - `3B` (0-59 in hex)

**24-Hour Mode:**
- Hours: `00` - `17` (0-23 in hex)
- Minutes: `00` - `3B` (0-59 in hex)

**Example:** 3:45 PM displays as:
- 12h: alternates between `03` (Q=ON) and `2D` (Q=OFF)
- 24h: alternates between `0F` (Q=ON) and `2D` (Q=OFF)

## Setting the Time

For versions with `_set` suffix:

1. **Set Hours:** Q LED is ON. Set switches to desired hour, press INPUT.
2. **Set Minutes:** Q LED is OFF. Set switches to desired minutes, press INPUT.
3. **Clock Starts:** Begins running from the set time.

## Timing Calibration

The delay value at address 0x19 (or 0x33 in _set versions) controls clock speed:

```
At 1 MHz crystal:
- 0x51 (81) ≈ 1 second (default)
- Increase if clock runs fast
- Decrease if clock runs slow
```

**Timing calculation:**
- 1 MHz clock ÷ 8 = 125,000 machine cycles/sec
- Delay loop: ~6 cycles × 256 = 1,536 cycles per unit
- 125,000 ÷ 1,536 ≈ 81 (0x51) units per second

## Quick Reference

**Hex to Time conversion:**
| Display | 12h | 24h |
|---------|-----|-----|
| `01` | 1 AM/PM | 1 AM |
| `06` | 6 AM/PM | 6 AM |
| `0C` | 12 | 12 PM |
| `0F` | - | 3 PM (15) |
| `17` | - | 11 PM (23) |
| `1E` | 30 min | 30 min |
| `3B` | 59 min | 59 min |
