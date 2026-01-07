# COSMAC ELF Program Ideas

Future program ideas for the COSMAC ELF with its minimal I/O:
- 8 toggle switches (binary input, 0x00-0xFF)
- 1 push button (INPUT)
- 2 hex digit display (00-FF)
- 1 LED (Q output)

---

## Games

### Reaction Timer
Q LED turns on after a random delay. Player hits INPUT as fast as possible. Display shows reaction time (lower = better).

**Implementation notes:**
- Use switches to set difficulty/delay range
- Random delay could use R3 low bits as pseudo-random seed
- Time measured in loop iterations, scaled to fit display
- Estimated size: 40-60 bytes

### Guess the Number
Computer picks a secret 00-FF value. Player sets switches and presses INPUT to guess.

**Implementation notes:**
- Q LED blinks for "too high", stays solid for "too low"
- Display shows number of guesses when correct
- Could reveal answer after N wrong guesses
- Estimated size: 50-70 bytes

### Simon Memory
Display flashes a sequence of hex digits. Player must reproduce the sequence on switches, pressing INPUT after each digit.

**Implementation notes:**
- Sequence starts with 1 digit, grows each successful round
- Q LED shows success/failure
- Need to store sequence in memory (limits max length)
- Pseudo-random sequence generation from seed
- Estimated size: 60-80 bytes

### Slot Machine
Display rapidly cycles random digits. Press INPUT to "stop" each digit.

**Implementation notes:**
- Two separate "reels" for each hex digit
- Match patterns (77, doubles, etc.) to win
- Q LED celebrates wins
- Estimated size: 50-70 bytes

### Code Lock
Enter the correct sequence of 3-4 switch values, pressing INPUT between each.

**Implementation notes:**
- Q LED lights when unlocked
- Display could show attempt count or hint (how many correct so far)
- Secret code stored in program memory (or set by switches in "setup mode")
- Estimated size: 30-50 bytes

---

## Educational / Utility

### Binary Trainer
Display shows a random hex value. Player sets the binary equivalent on switches and presses INPUT to check.

**Implementation notes:**
- Q LED shows right/wrong
- Could keep score (correct answers in display after round)
- Simple and great for learning hex/binary conversion
- Estimated size: 30-45 bytes

### Math Drill
Display alternates between two numbers. Player enters the result (sum, difference, XOR, AND) on switches.

**Implementation notes:**
- Switches select operation mode
- Q LED confirms correct answer
- Could increase difficulty over time
- Estimated size: 50-70 bytes

### Prime Detector
Slow counter on display. Q LED lights whenever the current value is prime.

**Implementation notes:**
- Need primality test routine (trial division)
- Display counts 02, 03, 04... with delay
- Q LED on for primes: 02, 03, 05, 07, 0B, 0D, 11, 13...
- Computationally intensive for larger numbers
- Estimated size: 50-70 bytes

### Fibonacci Sequence
Display cycles through Fibonacci numbers: 01, 01, 02, 03, 05, 08, 0D, 15, 22, 37, 59, 90, E9...

**Implementation notes:**
- Wraps at FF (or could show overflow with Q LED)
- Switches control speed
- Could also do powers of 2, triangular numbers, etc.
- Estimated size: 25-35 bytes

---

## Demos / Toys

### Random Dice
Switches set the range (lower nibble = max value, e.g., 06 for d6). Press INPUT to roll.

**Implementation notes:**
- Display shows random result 01 to max
- Q LED flashes during "roll" animation
- Useful for tabletop gaming
- Estimated size: 35-50 bytes

### Metronome
Switches set tempo. Q LED blinks at that rate.

**Implementation notes:**
- Display shows current tempo setting
- If Q connected to speaker, produces audible click
- Could add accent patterns (every 2nd, 4th beat brighter/different)
- Estimated size: 25-40 bytes

### Q LED Music
Play simple melodies through the Q output (requires speaker connection).

**Implementation notes:**
- Switches select song
- INPUT starts/stops playback
- Square wave tones via Q toggling at audio frequencies
- Song data stored as frequency/duration pairs
- Estimated size: varies by song length, 50+ bytes

### Binary Clock
Display shows time. Q LED blinks each second.

**Implementation notes:**
- Left digit = hours (0-17 for 24h, or 1-C for 12h)
- Right digit = minutes (0-3B, i.e., 0-59)
- Not accurate without crystal, but demonstrates concept
- Switches could set time
- Estimated size: 40-60 bytes

### Bouncing Ball
Simple animation: a value "bounces" between 00 and FF on the display.

**Implementation notes:**
- Counts up to FF, then back down to 00, repeat
- Switches control speed
- Q LED on during "up" phase, off during "down"
- Estimated size: 20-30 bytes

---

## Priority / Recommendations

Good starting projects (simple, educational, small):
1. **Binary Trainer** - Simple logic, teaches hex/binary
2. **Reaction Timer** - Fun, demonstrates timing/input
3. **Bouncing Ball** - Very simple demo, good for testing
4. **Fibonacci Sequence** - Mathematical, compact

More ambitious:
5. **Simon Memory** - Requires sequence storage, more complex
6. **Guess the Number** - Needs random number generation
7. **Code Lock** - Good security/puzzle demo

---

## Technical Considerations

### Random Number Generation
The 1802 has no built-in RNG. Options:
- Use low bits of a free-running counter (sampled when user presses INPUT)
- Linear feedback shift register (LFSR) in software
- Read uninitialized memory (unpredictable but not truly random)

### INPUT Button Detection
- EF4 flag reflects INPUT button state
- Use B4 (branch if EF4) or BN4 (branch if not EF4) instructions
- Opcode: 34 = B4, 3C = BN4

### Timing
- At 1.79 MHz, each machine cycle is ~0.56 microseconds
- Delay loops give rough timing control
- No interrupts in simple programs, so timing is approximate

### Memory Constraints
- Programs loaded via toggle switches, so smaller = easier to enter
- 256 bytes of RAM total on basic ELF
- Keep programs under 64 bytes if possible for easy entry
