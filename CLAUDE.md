# COSMAC ELF Assembly Project

Assembly code development for the RCA CDP1802 microprocessor running on the COSMAC ELF system.

## Reference Documentation

- `reference docs/cdp1802.pdf` - CDP1802 processor datasheet (pinouts, timing, instruction set)
- `reference docs/COSMAC ELF Manual.pdf` - COSMAC ELF system manual with architecture details and sample programs

## CDP1802 Architecture Overview

- 8-bit CMOS microprocessor
- 16 x 16-bit general purpose registers (R0-RF)
- 8-bit accumulator (D register)
- 4-bit program counter selector (P) and data pointer selector (X)
- Single-bit flags: DF (data flag/carry), Q (output), IE (interrupt enable)
- 4 external flag inputs (EF1-EF4)
- 91 instructions
- Big-endian addressing

## Key Registers

| Register | Purpose |
|----------|---------|
| D | 8-bit accumulator |
| DF | Data flag (carry/borrow) |
| P | 4-bit pointer to current program counter (selects R0-RF) |
| X | 4-bit pointer to current data register (selects R0-RF) |
| T | Holds old X,P during interrupt |
| R0-RF | 16 x 16-bit scratch-pad registers |
| Q | Single-bit output flag |

## Common Instructions

| Opcode | Mnemonic | Description |
|--------|----------|-------------|
| 00 | IDL | Idle (wait for DMA/interrupt) |
| 0N | LDN | Load D from memory at R(N) |
| 1N | INC | Increment R(N) |
| 2N | DEC | Decrement R(N) |
| 3x | BR/BZ/etc | Short branch instructions |
| 4N | LDA | Load D and advance R(N) |
| 5N | STR | Store D to memory at R(N) |
| 6N | OUT/INP | I/O instructions |
| 7x | RET/DIS/etc | Return, disable, skip instructions |
| 8N | GLO | Get low byte of R(N) into D |
| 9N | GHI | Get high byte of R(N) into D |
| AN | PLO | Put D into low byte of R(N) |
| BN | PHI | Put D into high byte of R(N) |
| Cx | Long branch instructions |
| DN | SEP | Set P = N (change program counter) |
| EN | SEX | Set X = N (change data pointer) |
| F8 | LDI | Load immediate into D |
| Fx | ALU operations (ADD, SUB, AND, OR, XOR, etc.) |

## COSMAC ELF I/O

- Q output controls LED
- EF4 input reads momentary pushbutton
- Hex display directly shows data bus during DMA cycles
- Toggle switches for data entry in LOAD mode

## Operating Modes

1. **LOAD** - Enter program via switches, DMA writes to memory
2. **RESET** - Initialize, R0=0, P=0, X=0, Q=0
3. **RUN** - Execute program
4. **PAUSE** - Halt execution, retain state

## Code Format

Assembly listings use format: `ADDRESS OPCODE [COMMENT]`
```
00 64    ; OUT 4 - Display D on hex display
01 00    ; IDL
02 90    ; GHI R0
```
