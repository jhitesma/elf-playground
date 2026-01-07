# Understanding the Hex Counter Program

A line-by-line explanation for programmers new to assembly/machine code.

## Introduction: Machine Code vs High-Level Languages

If you're coming from Python, JavaScript, or C, machine code will feel alien at first. Here's why:

**No variables** - Instead of named variables, you have a fixed set of *registers* (small storage locations built into the CPU) and *memory addresses* (numbered locations in RAM).

**No functions with return addresses** - The CPU just executes the next instruction in sequence. To "call" code elsewhere, you manually jump to it. To "return," you jump back.

**No data types** - Everything is just bytes. The number `65`, the letter `A`, and the instruction `OUT 4` are all just the byte `0x41` - context determines meaning.

**Everything is explicit** - Want to add two numbers? You must first load one into a register, then add the other to it, then store the result somewhere. Nothing is automatic.

## The CDP1802 Architecture (Quick Overview)

The 1802 has:
- **D register**: An 8-bit "accumulator" - most operations go through this
- **R0-RF**: Sixteen 16-bit general-purpose registers (that's R0 through R15 in hex)
- **P register**: 4-bit pointer that says which R register is the Program Counter
- **X register**: 4-bit pointer that says which R register is the Data Pointer
- **Flags**: DF (carry flag), Q (output line), etc.

Think of P and X as "which register should I use for this purpose?" selectors.

## The Program's Purpose

This program counts from 00 to FF on the hex display, with a delay between each count that's controlled by the input switches.

In pseudocode:
```
counter = 0
while true:
    delay = read_switches()
    wait(delay)
    counter++
    display(counter)
```

Now let's see how that translates to 30 bytes of machine code.

---

## Reading the Code Listing

If you've never seen assembly or machine code before, the format of these listings might be confusing. Here's how to read them:

```
ADDR  CODE  INSTRUCTION     DESCRIPTION
----  ----  -----------     -----------
02    90    GHI R0          D = R0.1 (high byte of R0)
03    B2    PHI R2          R2.1 = D (= 0x00)
```

Each line has four columns:

| Column | Name | What It Is |
|--------|------|------------|
| `02` | Address | The memory location where this instruction lives (in hexadecimal) |
| `90` | Opcode | The actual byte stored in memory - this is what the CPU sees |
| `GHI R0` | Mnemonic | A human-readable name for the instruction |
| `D = R0.1...` | Comment | An explanation of what the instruction does |

### Machine Code vs Assembly Language

**Machine code** is what the CPU actually executes - raw bytes like `90`, `B2`, `F8`. Every CPU has its own set of opcodes, and there's a fixed mapping between each byte value and the operation it performs.

**Assembly language** uses mnemonics (short names) to represent those same operations. Instead of memorizing that `90` means "copy the high byte of R0 into D," you write `GHI R0` (Get High byte of R0). An *assembler* program converts these mnemonics back into the numeric opcodes.

For the COSMAC ELF, there's no assembler - you enter the raw bytes directly using toggle switches. The mnemonics in our listing are just for human readability. What actually gets loaded into memory is:

```
Address:  00  01  02  03  04  05  06  07  08  09  0A  0B  0C  ...
Contents: 64  00  90  B2  B3  A3  F8  30  A2  52  E2  30  16  ...
```

### Two-Byte Instructions

Most 1802 instructions are one byte, but some need additional data. For example:

```
06    F8    LDI 30          Load Immediate: next byte goes into D
07    30                    ...the value 0x30
```

`LDI` (Load Immediate) is opcode `F8`, but it needs to know *what* value to load. That value (`30`) comes in the next byte. So `LDI 0x30` occupies two memory locations: `F8` at address 06 and `30` at address 07.

Branch instructions work similarly:

```
0B    30    BR 16           Branch to address 0x16
0C    16                    ...the target address
```

The opcode `30` means "branch unconditionally," and the next byte (`16`) specifies where to jump.

### Hexadecimal Notation

All values in these listings are in hexadecimal (base 16). You might see them written as:
- `30` - just the hex digits
- `0x30` - with the "0x" prefix (common in programming)
- `$30` - with a dollar sign (common in some assemblers)

In hex, the digits go 0-9, then A-F for 10-15. So `0x30` = 3×16 + 0 = 48 in decimal.

---

## Line-by-Line Walkthrough

### The Opening Sequence (Addresses 00-01)

```
00    64    OUT 4           Output D to hex display
01    00    IDL             Idle
```

**Purpose unclear.** These two bytes exist in the original program, but their purpose is not obvious:

- `OUT 4` outputs whatever value happens to be in D (essentially garbage after the LOAD process)
- `IDL` waits for a DMA or interrupt cycle

On the COSMAC ELF, DMA cycles occur continuously to refresh the hex display, so `IDL` doesn't actually pause for user input - it just waits for one DMA cycle (microseconds) and execution continues immediately at address 02.

The program works fine with these bytes, but they don't seem to serve a clear purpose. Possibilities include:
- Copied from a program template or convention
- Leftover from debugging/development

**Update:** Testing confirmed these two bytes can be safely removed. See `hexcounter_no_preamble.txt` for a 29-byte version without them (all branch targets adjusted accordingly).

The "real" program logic begins at address 02 with initialization.

### Initialization (Addresses 02-0A)

**Goal:** We need to set up two things before the main loop:
1. **R3 = 0x0000** - This register will be our counter
2. **Data pointer to scratch memory** - The `OUT` instruction outputs from a memory location, not directly from D. So we need a register (R2) to point at a "scratch" memory address (0x0030) where we can temporarily store values for output.

After setup, we'll have:
- R3 = 0x0000 (counter)
- R2 = 0x0030 (data pointer, pointing to scratch memory)
- X = 2 (tells CPU to use R2 as the data pointer)
- M[0x0030] initialized (scratch memory ready for use)

**Now let's see how we build this:**

```
02    90    GHI R0          D = R0.1 (high byte of R0)
```

After RESET, R0 = 0x0000. We exploit this by loading R0's high byte (which is 0x00) into D. Now D = 0x00, which we'll use to zero out other registers.

*Why this roundabout approach?* There's no "set register to zero" instruction. We have to move values through the D register. Since R0.1 is already 0, we use it as our source of zeros.

```
03    B2    PHI R2          R2.1 = D (= 0x00)
04    B3    PHI R3          R3.1 = D (= 0x00)
05    A3    PLO R3          R3.0 = D (= 0x00)
```

These three instructions use our zero in D to:
- Set R2's high byte to 0x00
- Set R3's high byte to 0x00
- Set R3's low byte to 0x00

Now R3 = 0x0000 (our counter is initialized).

```
06    F8    LDI 30          Load Immediate: next byte goes into D
07    30                    ...the value 0x30
08    A2    PLO R2          R2.0 = D
```

`LDI` is a two-byte instruction: the opcode (F8) followed by the value to load. This puts 0x30 into D, then we store it into R2's low byte. Since R2's high byte is already 0x00, now R2 = 0x0030.

```
09    52    STR R2          Store D at memory[R2]
```

Store D (still 0x30) into memory at address 0x0030. This initializes our scratch memory location. (The value stored doesn't matter much - it will be overwritten before we display it.)

```
0A    E2    SEX R2          Set X = 2
```

This tells the CPU "use R2 as the data pointer." The X register is a 4-bit selector (0-F) that tells certain instructions which register contains the memory address to use. Setting X=2 means "when instructions like OUT or INP need a memory address, use the value stored in R2."

**Setup complete.** Now when we execute `OUT 4`, it will output from memory[R2], which is memory[0x0030].

*High-level equivalent of 02-0A:*
```python
counter = 0           # R3 = 0x0000
data_ptr = 0x0030     # R2 holds address 0x0030
memory[data_ptr] = 0x30  # Initialize scratch memory location
```

### The Main Loop (Addresses 0B-15)

**Goal:** The main loop does three things on each iteration:
1. Call the delay subroutine (to control counting speed)
2. Increment the counter
3. Display the counter value on the hex display

The loop structure is a bit unusual - it starts by jumping to the delay code, which then jumps back into the middle of this section. This "jump there, jump back" pattern is how simple 1802 programs handle subroutines without a proper call stack.

**Now let's walk through it:**

```
0B    30    BR 16           Branch (jump) to address 0x16
0C    16
```

`BR` is an unconditional short branch - a 2-byte instruction where the second byte is the target address. This jumps to the delay subroutine at 0x16.

```
0D    13    INC R3          Increment R3
```

This increments our 16-bit counter. The low byte (R3.0) increases by 1; if it overflows from FF to 00, the high byte (R3.1) also increases.

```
0E    83    GLO R3          D = R3.0 (low byte of counter)
```

We load the counter's low byte into D so we can display it.

```
0F    52    STR R2          memory[R2] = D
```

Store the counter value to memory at address 0x0030 (where R2 points).

```
10    64    OUT 4           Output to port 4 (hex display)
```

Here's a quirk of the 1802: `OUT` doesn't output from D directly. It outputs from memory at address R(X), then increments R(X). Since X=2, it outputs memory[R2] (our counter value) to the display, then R2 becomes 0x0031.

```
11    22    DEC R2          Decrement R2
```

We decrement R2 back to 0x0030 to undo the auto-increment from OUT. We need R2 to stay pointing at our scratch byte.

```
12    C4    NOP             No operation
13    C4    NOP             No operation
```

These do nothing. They might be timing padding, or leftover space from an earlier version. In a 1.79 MHz CPU, two NOPs add about 4 microseconds - negligible.

```
14    30    BR 0B           Branch back to 0x0B
15    0B
```

Jump back to the delay call, creating our infinite loop.

*High-level equivalent of 0B-15:*
```python
while True:
    delay()              # 0B-0C: BR 16
    counter += 1         # 0D: INC R3
    display(counter)     # 0E-10: GLO, STR, OUT
```

### The Delay Subroutine (Addresses 16-1E)

**Goal:** Create a pause before each counter increment so the display doesn't change too fast to read. This version uses a fixed delay value of 0x5F (95 decimal).

The approach:
1. Load a fixed delay value into the high byte of R4
2. If the value is zero, skip the delay (won't happen with 0x5F, but the code handles it)
3. Loop ~256 times for each unit of the delay value

The trick is using a 16-bit register (R4) as a countdown timer, but only setting its high byte. This means a delay value of 0x5F results in roughly 95 × 256 = 24,320 loop iterations.

**Now let's walk through it:**

```
16    F8    LDI 5F          Load Immediate: next byte goes into D
17    5F                    ...the value 0x5F
```

Load the fixed value 0x5F into D. This is our delay amount - larger values mean longer delays.

```
18    B4    PHI R4          R4.1 = D (delay amount)
```

Store the delay value into the high byte of R4. R4 becomes our countdown timer, with the high byte controlling how long we wait.

```
19    32    BZ 0D           Branch if D = 0 to address 0x0D
1A    0D
```

If the delay value is 0, skip the delay entirely - branch to address 0x0D (the counter increment). With our fixed value of 0x5F this never triggers, but the code structure allows for it.

```
1B    94    GHI R4          D = R4.1
1C    24    DEC R4          Decrement R4 (full 16-bit)
```

Load R4's high byte into D, then decrement the full 16-bit R4.

**Here's the clever part:** DEC R4 decrements the entire 16-bit register. If R4 = 0x5F00:
- After 1 DEC: R4 = 0x5EFF
- After 256 DECs: R4 = 0x5E00
- After 257 DECs: R4 = 0x5DFF

The high byte only decreases by 1 for every 256 decrements of the full register.

```
1D    30    BR 19           Branch to 0x19
1E    19
```

Loop back to the BZ check. We keep looping until GHI R4 returns 0 (meaning R4.1 has counted down to zero).

**Total delay iterations:** With delay = 0x5F (95), we loop approximately 95 × 256 = 24,320 times before R4.1 reaches zero.

*High-level equivalent of 16-1E:*
```python
def delay():
    amount = 0x5F             # 16-17: LDI 5F
    if amount == 0:           # 19-1A: BZ 0D
        return
    for i in range(amount * 256):  # Approximate
        pass
```

### Why No "Return" Instruction?

Notice that the delay "subroutine" doesn't return conventionally. Instead:
- Main loop branches TO address 0x16 (delay start)
- Delay loop branches TO address 0x0D (back in main loop)

This is a common pattern in small 1802 programs. True subroutine calls (using `SEP` to switch program counters) exist but require more setup. For a simple program like this, direct branches are shorter and simpler.

### Modification: Switch-Controlled Delay

The fixed delay of 0x5F works fine, but what if you want to control the speed in real-time? The COSMAC ELF's toggle switches can be read using `INP 4`, allowing us to replace the fixed delay with a user-controlled one.

**The change:** Replace `LDI 5F` with `INP 4`:

```
; Original (fixed delay):
16    F8    LDI 5F          Load fixed value 0x5F
17    5F

; Modified (switch-controlled):
16    6C    INP 4           Read switches into D
17    C4    NOP             Padding to keep addresses aligned
```

The `INP 4` instruction reads the current state of all 8 toggle switches as a single byte (0x00-0xFF) and stores it in both D and memory at R(X). Since we immediately use D for the delay value, we get real-time speed control:

- Switches at `FF` = slowest (longest delay)
- Switches at `01` = fastest (barely visible)
- Switches at `00` = no delay (skips via BZ, full speed)

The NOP at address 17 is padding to keep the rest of the program at the same addresses. Alternatively, you can remove it and adjust all subsequent branch targets (see `hexcounter_switch_delay_short.txt` for a 30-byte version).

*High-level equivalent with switches:*
```python
def delay():
    amount = read_switches()  # 16: INP 4
    if amount == 0:           # 19-1A: BZ 0D
        return
    for i in range(amount * 256):
        pass
```

See `hexcounter_switch_delay.txt` for the complete modified program.

---

## Memory Map

```
0x00-0x1D  Program code (30 bytes)
0x1E-0x2F  Unused
0x30       Scratch byte (used by OUT instruction)
0x31+      Available for expansion
```

## Register Usage

| Register | Purpose |
|----------|---------|
| R0 | Program Counter (P=0 at startup) |
| R2 | Data pointer for OUT instruction |
| R3 | Counter (0x0000 to 0xFFFF) |
| R4 | Delay countdown timer |

## Instruction Encoding Patterns

Understanding the opcode structure helps you read 1802 code:

| Pattern | Meaning | Example |
|---------|---------|---------|
| 0N | LDN - Load D from M[RN] | 02 = LDN R2 |
| 1N | INC RN | 13 = INC R3 |
| 2N | DEC RN | 24 = DEC R4 |
| 3x | Short branch | 30 = BR, 32 = BZ |
| 5N | STR RN - Store D to M[RN] | 52 = STR R2 |
| 6N | I/O (1-7 = OUT, 9-F = INP) | 64 = OUT 4, 6C = INP 4 |
| 8N | GLO RN - Get low byte | 83 = GLO R3 |
| 9N | GHI RN - Get high byte | 94 = GHI R4 |
| AN | PLO RN - Put D in low byte | A2 = PLO R2 |
| BN | PHI RN - Put D in high byte | B4 = PHI R4 |
| EN | SEX RN - Set X register | E2 = SEX R2 |
| F8 | LDI - Load immediate | F8 30 = LDI 0x30 |

---

## Key Takeaways

1. **Everything flows through D**: The accumulator is the hub of all data movement.

2. **Registers are versatile**: The same register (like R2) can be a pointer, a counter, or temporary storage depending on context.

3. **Branches are your control flow**: No if/else, no for loops - just conditional and unconditional jumps.

4. **Instructions have side effects**: OUT auto-increments its pointer. You must manually undo this if you need the pointer unchanged.

5. **Memory is unstructured**: There's no stack, no heap - just 64KB of bytes. You decide what goes where.

6. **Code density matters**: At 30 bytes, this program does something useful. Every byte counts when you're entering code via toggle switches!
