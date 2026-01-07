# Bouncing Ball

A simple animation demo that counts from 00 to FF, then reverses direction and counts back down to 00, creating a "bouncing" effect on the hex display.

## Features

- Display shows values 00 → FF → 00 → FF → ... in a continuous bounce
- Q LED indicates direction: ON = counting up, OFF = counting down
- Adjustable speed via delay value (or switches in switch-controlled version)

## Program Versions

| File | Size | Description |
|------|------|-------------|
| `bouncing_ball.txt` | 42 bytes | Fixed delay (0x5F) |
| `bouncing_ball_switch.txt` | 41 bytes | Switch-controlled delay |

## How It Works

The program uses the Q output as a direction flag:
- Q = 1: Counting up (INC R3)
- Q = 0: Counting down (DEC R3)

At the boundaries:
- When counter reaches FF, switch direction to down (REQ)
- When counter reaches 00, switch direction to up (SEQ)

The FF detection uses XRI (XOR immediate): `D XOR FF = 0` only when D = FF.

## Quick Start

```
90 B2 B3 A3 F8 30 A2 E2 7B 83 52 64 22 F8 5F B4 24 94 3A 10 31 1F 23 83 32 1C 30 09 7B 30 09 13 83 FB FF 32 27 30 09 7A 30 09
```

## Adjusting Speed

In `bouncing_ball.txt`, change the byte at address 0E:
- `FF` = slowest
- `5F` = medium (default)
- `01` = fastest

Or use `bouncing_ball_switch.txt` to control speed with the toggle switches in real-time.
