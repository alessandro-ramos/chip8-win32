# CHIP-8 Interpreter, Assembler and Disassembler

This package contains a [CHIP-8][wikipedia] SDK (System Development Kit) for Windows, composed by:
* CHIP-8 **Emulator**
* CHIP-8 **Assembler**
* CHIP-8 **Disassembler**.

It also supports the SuperChip instructions.

This work is a fork of [Werner Stoop CHIP-8 Tools](https://github.com/wernsey/chip8)

The motivation for this for is remove the SDL / Linux specific files and make instructions. So, Windows users will have less confusion when building it.

The syntax of the assembler and disassembler is based on the syntax described
in [Cowgod's Chip-8 Technical Reference v1.0][cowgod], by Thomas P. Greene

Frédéric Devernay's [SVision-8 website](http://devernay.free.fr/hacks/chip8/)
has a wealth of information. He also has a collection of CHIP-8 games and
programs in his [GAMES.zip](http://devernay.free.fr/hacks/chip8/GAMES.zip).

## Compilation and Usage

* Windows: The system was built and tested with the
  [MinGW](https://sourceforge.net/projects/mingw/) tools. To compile it type `mingw32-make` from a *Git Bash* shell.

To use the emulator:

* Under Windows: Type `$ ./chip8 [path-to-your-game.ch8]`.

The assembler and disassemblers are simple command line applications and
platform independent.

To use the assembler, type

    $ ./c8asm -o file.c8h file.asm

This will assemble `file.asm` into a binary `file.c8h`. If the `-o` is not
specified it will default to `a.c8h`.

To use the disassembler, run the command

    $ ./c8dasm a.ch8 > outfile.asm

where `a.ch8` is the file you want to disassemble.

## Interpreter Implementations

The core of the emulator is in `chip8.c`. The idea is that this core be
platform independent and then hooks are provided for platform specific
implementations.

The API is described in `chip8.h`. The `docs` target in the Makefile generates
HTML documentation from it.

There is only one implementation provided in this repository:

Native Windows implementation which is intended for small size and
  requires no third party dependencies.

In both versions

* `bmp.h` and `bmp.c` (together with the `fonts/` directory) is used to draw
  and manipulate the bitmap graphics. See also
  https://github.com/wernsey/bitmap
* `render.c` implements the `init_game()`, `deinit_game()` and `render()`
  functions that forms the core of both implementations and demonstrates how
  the interpreter's API works.

The `render()` function checks the keyboard and executes the interpreter a
couple of times by calling `c8_step()` and redraws the screen if it changed.
The SDL and Win32 frameworks were written in such a way that the `render()`
function works with both with only a couple of minor modifications.

The implementations feature a rudimentary debugger: Press F5 to pause a running
game. The program counter and the current instruction will be displayed at the
bottom of the screen, along with the values of the 16 Vx registers. Press F6 to
step through the program to the next instruction and F8 to resume the program.

The `Makefile` will build this GDI version by default.

### Win32/GDI Implementation

The native Windows version uses a simple hook around the Win32 GDI and requires
no third party dependencies.

`gdi.h` and `gdi.c` implements the native Windows code. It implements a
`WinMain` function with the main Win32 events processing loop. It binds the
window's GDI context to a `Bitmap` object so that a render function can draw
onto it and fires off periodic `WM_PAINT` messages which calls the `render()`
function to draw the screen.

## Implementation Notes

The original author consulted several sources for his implementation (see references below),
and there were some discrepancies. This is how I handled them:

* Regarding `2nnn`, [cowgod][] says the stack pointer is incremented first (i.e.
  `stack[++SP]`), but that skips `stack[0]`. My implementation does it the
  other way round.
* ~~The `Fx55` and `Fx65` instructions doesn't change `I` in my implementation:~~
  * This is a known [quirk][langhoff].
  * The interpreter now provides `QUIRKS_MEM_CHIP8` to control this
* I've read [David Winter's emulator][winter]'s documentation when I started, but I
  implemented things differently:
  * His emulator scrolls only 2 pixels if it is in low-res mode, but 4 pixels
    is consistent with [Octo][].
  * His emulator's `Dxy0` instruction apparently also works differently in
    lo-res mode.
* ~~[instruction-draw][] says that images aren't generally wrapped, but
  [muller][] and [Octo][] seems to think differently.~~
  * This is alsp known [quirk][langhoff].
  * The interpreter now provides `QUIRKS_CLIPPING` to control this
* According to [chip8-wiki][], the upper 256 bytes of RAM is used for the display, but it
  seems that modern interpreters don't do that. Besides, you'd need 1024 bytes
  to store the SCHIP's hi-res mode.
* `hp48_flags` is not cleared between runs (See [octo-superchip]); I don't make any effort
  to persist them, though.
* Apparently there are CHIP-8 interpreters out there that don't use the
  standard 64x32 and 128x64 resolutions, but I don't support those.
* As far as I can tell, there is not much in terms of standard timings on
  CHIP-8 implementations. My implementation allows you to specify the speed as
  the number of instructions to execute per second (through the global variable
  `speed` in `render.c`). The value of 1200 instructions per second seems like
  a good value to start with.

## References and Links
* [Werner Stoop Original CHIP-8 Tools](https://github.com/wernsey/chip8)
* [Wikipedia entry][wikipedia]
* [Cowgod's Chip-8 Technical Reference v1.0][cowgod], by Thomas P. Greene,
* [How to write an emulator (CHIP-8 interpreter)][muller] by Laurence Muller (archived)
* [CHIP8 A CHIP8/SCHIP emulator Version 2.2.0][winter], by David Winter
* [Chip 8 instruction set][chip8def], author unknown(?)
* [Byte Magazine Volume 03 Number 12 - Life pp. 108-122. "An Easy
  Programming System,"][byte] by Joseph Weisbecker
* [chip8.wikia.com][chip8-wiki]
  * Their page on the [Draw instruction][instruction-draw]
* [Mastering CHIP-8][mikolay] by Matthew Mikolay
* [Octo][], John Earnest
* The [Octo SuperChip document][octo-superchip], by John Earnest
* [codeslinger.co.uk](http://www.codeslinger.co.uk/pages/projects/chip8/primitive.html)
* [CHIP‐8 Technical Reference](https://github.com/mattmikolay/chip-8/wiki/CHIP%E2%80%908-Technical-Reference), by Matthew Mikolay
* [corax89' chip8-test-rom](https://github.com/corax89/chip8-test-rom)
* [Timendus' chip8-test-suite][Timendus] was extremely useful to help clarify and fix the quirks.
  * Timendus' [Silicon8](https://github.com/Timendus/silicon8/) CHIP8 implementation
* Tobias V. Langhoff's [Guide to making a CHIP-8 emulator][langhoff]
  * This one is very useful for explaining the various quirks
* [Chip-8 on the COSMAC VIP: Drawing Sprites](https://web.archive.org/web/20200925222127if_/https://laurencescotford.com/chip-8-on-the-cosmac-vip-drawing-sprites/), by Laurence Scotford (archive link)
* [CHIP-8 extensions and compatibility](https://chip-8.github.io/extensions/) -
explains several of the variants out there
* <https://github.com/zaymat/super-chip8>
  * The [load_quirk and shift_quirk](https://github.com/zaymat/super-chip8#load_quirk-and-shift_quirk)
    section of that README has another explaination of some of the
    quirks, along with a list of known games that need them.

* <https://github.com/dario-santos/Super-Chip-Emulator> has a collection of ROMs I used for testing
* <https://github.com/JohnEarnest/chip8Archive> - Archive of CHIP8 programs.
* <https://github.com/tobiasvl/awesome-chip-8>

[wikipedia]: https://en.wikipedia.org/wiki/CHIP-8
[cowgod]: http://devernay.free.fr/hacks/chip8/C8TECH10.HTM
[muller]: https://web.archive.org/web/20110426134039if_/http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter/
[winter]: http://devernay.free.fr/hacks/chip8/CHIP8.DOC
[chip8def]: http://devernay.free.fr/hacks/chip8/chip8def.htm
[byte]: https://archive.org/details/byte-magazine-1978-12
[chip8-wiki]: <http://chip8.wikia.com/wiki/Chip8>
[instruction-draw]: http://chip8.wikia.com/wiki/Instruction_Draw
[mikolay]: <http://mattmik.com/chip8.html>
[octo]: https://github.com/JohnEarnest/Octo
[octo-superchip]: https://github.com/JohnEarnest/Octo/blob/gh-pages/docs/SuperChip.md
[langhoff]: https://tobiasvl.github.io/blog/write-a-chip-8-emulator/
[Timendus]: https://github.com/Timendus/chip8-test-suite

## License

This code is licensed under the [Apache license version 2](http://www.apache.org/licenses/LICENSE-2.0):

```
    Copyright 2015-2016 Werner Stoop

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
```

## TODO/Roadmap/Ideas

* [ ] I really need to fix the "Display wait" quirk. See [Timendus][]'s `5-quirks.ch8` test.
* [x] The quirks need to be in a flags variable so that they can be controlled at runtime
* [x] The runtime should have a `-q` command line option to control the quirks
* [x] The assembler needs an `include "file.asm"` directive.
  * [x] You need a way to specify how to do the include, because the assembler must be
    usable even if you're not loading the source from files. I suggest a function pointer
    that points to `c8_load_txt()` by default, but can be made to point elsewhere (or set to
    `NULL` and disable includes completely)
* [x] I should consider a `text "hello"` directive in the assembler, that places a null
      terminated string in the bytecode. Users might be able to display the text at some point
      if you have the right sprites; [Octo][] does it.
* [x] Allow for some hooks in the library to let the `SYS nnn` (`0nnn`) instructions break
      out into the environment outside.
      * It's meant as a bit of a joke, might be neat if you embed a CHIP-8 interpreter
        in another program and call out to it as a sort of scripting language.
* [x] Command line option, like `-m addr=val`, that will set the byte at `addr` to `val` in the
      RAM before running the interpreter.
      * A immediate use case is for, example, [Timendus][]'s `5-quirks.ch8` test that allows you
        to write a value between 1 and 3 to `0x1FF` and then the program will bypass the initial
        menu and skip directly to the corresponding test. I imagine that while developing and
        debugging CHIP-8 programs it might be useful to have such a mechanism.
* [x] Fix the assembler that doesn't do any bounds checks on `stepper->token`
* [ ] Breakpoints in the debugger
* [ ] ~~A `.map` file output by the assembler...~~

Porting to the Amiga 500 might be an interesting challenge to get it truly portable:
The Amiga's bus is word aligned, so if the program counter is ever an odd number then
the system might crash when it tries to retrieve an instruction. Also, the Amiga is big
endian, so that might reveal some problems as well.

[XO-Chip compatibility](http://johnearnest.github.io/Octo/docs/XO-ChipSpecification.html) seems
like something worth striving for. [Here](https://chip-8.github.io/extensions/#xo-chip)'s a
short checklist of the changes. Also look at how [Octo](https://chip-8.github.io/extensions/#octo)
modifies some instructions.
