# COSMAC ELF Playground

Machine code programs for the [COSMAC ELF](https://en.wikipedia.org/wiki/COSMAC_ELF), a simple single-board computer from 1976 based on the RCA CDP1802 processor.

This repository is an experiment in using [Claude](https://claude.ai) to write and analyze code for the CDP1802. The 1802 is a less well-known processor compared to its contemporaries like the Z80 and 6502, which makes it an interesting test case for AI assistance - there's far less training data available for it, so it's a good way to explore how AI handles working with more obscure architectures.

## Programs

### [Hex Counter](hexcounter/)

A hex counter that displays 00-FF on the display with adjustable delay. Includes multiple versions from 20-31 bytes, with both fixed and switch-controlled delay options. See the [detailed explanation](hexcounter/hexcounter_explained.md) for a line-by-line walkthrough.

### [Bouncing Ball](bouncing-ball/)

A simple animation that counts 00 → FF → 00 in a continuous "bounce". The Q LED indicates direction (ON = up, OFF = down). 42 bytes with fixed or switch-controlled delay.

### Future Ideas

See [`plans/program-ideas.md`](plans/program-ideas.md) for planned programs including reaction timer, binary trainer, Simon memory game, and more.

## Hardware

The COSMAC ELF has minimal I/O:

- 8 toggle switches (binary input, readable as 0x00-0xFF)
- INPUT pushbutton
- 2-digit hex display
- Q LED

## Reference Documentation

The `reference docs/` folder contains:

- **CDP1802 datasheet** - Complete processor documentation
- **COSMAC ELF Manual** - Original construction and programming guide (By Paul Schmidt)
- **ELF Schematic** - Circuit diagram (By Paul Schmidt)

## Credits

Special thanks to **Paul Schmidt** for documenting a modern build of the COSMAC ELF:

- [Build details and documentation (PDF)](http://cosmacelf.com/publications/books/cosmac_elf_build_details.zip)
- [YouTube: COSMAC ELF Build Part 1](https://www.youtube.com/watch?v=MHYn_NBKsfU)
- [YouTube: COSMAC ELF Build Part 2](https://www.youtube.com/watch?v=MXX7FeYH1N8)

## Resources

- [cosmacelf.com](http://cosmacelf.com/) - COSMAC ELF resource site with documentation, programs, and more
- [COSMAC ELF Group](https://groups.io/g/cosmacelf) - Active mailing list for ELF enthusiasts
- [COSMAC ELF on Wikipedia](https://en.wikipedia.org/wiki/COSMAC_ELF)
- [CDP1802 on Wikipedia](https://en.wikipedia.org/wiki/RCA_1802)
