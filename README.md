# duckling-hackathon

A CHIP-8 and SUPER-CHIP emulator that plays games right in your terminal, written in
[Duckling](https://ducktype.org). It runs ready-made `.ch8` ROMs as well as Octo
(`.8o`) source files, which it assembles itself, so you can write a game and play it with
one command.

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│    ▄▄▄▄  ▄      ▄▄   ▄▄▄   ▄▄▄   ▄   ▄           ▄▄▄▄▄▄        │
│    █▄▄   █     █▄▄█  █▄▄▀  █▄▄▀   ▀▄▀          ▄▄██▀▀██▄▄      │
│    █     █▄▄▄  █  █  █     █       █           ████▄▄██▀▀      │
│                                          ▄▄    ██████▀▀        │
│    ▄▄▄   ▄  ▄   ▄▄▄  ▄  ▄                ██▄▄▄▄██████▄▄        │
│    █  █  █  █  █     █▄▀                 ██████▀▀▀▀████        │
│    █▄▄▀  ▀▄▄▀  ▀▄▄▄  █ ▀▄                ▀▀████▄▄▄▄██▀▀        │
│                                            ▀▀▀▀▀▀▀▀▀▀          │
│                                                                │
│                                                                │
│                                                                │
│           ▄▄▄   ▄▄▄   ▄▄▄▄   ▄▄▄   ▄▄▄        ▄   ▄            │
│           █▄▄▀  █▄▄▀  █▄▄   ▀▄▄   ▀▄▄         █ ▄ █            │
│           █     █ ▀▄  █▄▄▄  ▄▄▄▀  ▄▄▄▀        █▀ ▀█            │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

*FlappyDuck, the game included in `games/`, on its title screen.*

## What it can do

- **Two machines.** CHIP-8 with the quirks of the original COSMAC VIP interpreter, and
  SUPER-CHIP with the "modern" quirks Octo uses: 128x64 hires mode, 16x16 sprites,
  scrolling, the large font, flag registers and `exit`. SUPER-CHIP is the default.
- **Play in the terminal.** Two pixels per character cell using half-block characters,
  centred in the window, redrawn only when the program changes the screen. The frame
  grows when a program switches to hires. The sound timer rings the terminal bell.
- **Run Octo sources directly.** `.8o` files go through a built-in port of the Octo
  assembler that produces the same bytes as Octo, including labels, `:macro`, `:calc`,
  `:alias`, `:const`, `loop`/`while` and `if`/`begin`/`else`/`end`. Assembly errors
  point at the line.
- **Headless mode.** `--dump N` runs N frames without a terminal and prints the screen
  as `#` and `.`, which is handy for scripts and quick checks.
- **Built-in tests.** `-t` runs the unit tests for the CPU, the SUPER-CHIP instructions,
  the display, the Octo assembler and the string helpers.

## Requirements

- Linux. The emulator reads its arguments from `/proc/self/cmdline` and talks to the
  terminal through termios.
- The Duckling toolchain: `duck` and `duckc` on your `PATH`. `duck` uses the first
  `duckc` it finds there. Built and tested with these versions:

  | Tool | Version | Commit | Date |
  |---|---|---|---|
  | `duckc` | 0.3-alpha | `f49326c51` | 2026-11-20 |
  | DVM (`VM`) | 0.3-alpha | `f49326c51` | 2026-11-20 |
  | `duck` | 0.3.0 | `203a4c60a` | 2026-11-13 |
- A terminal with UTF-8. The screen needs 66x19 cells for CHIP-8 programs and 130x35 for
  SUPER-CHIP hires ones (the picture plus a border and a status line). If the window is
  too small you get a message saying how much room is needed.

## Quick start

```sh
duck build
duck run games/flappyduck.8o
```

Press **W** to flap and **Esc** to quit.

## Usage

```
duckling-hackathon <rom.ch8 | program.8o> [--chip8 | --schip] [--dump <frames>]
duckling-hackathon -t [filter]
```

| Option | What it does |
|---|---|
| `<file>` | A `.ch8` ROM, or an Octo `.8o` source (assembled before running). |
| `--schip` | SUPER-CHIP: hires, scrolling, big font, modern quirks. The default. |
| `--chip8` | Original CHIP-8 (COSMAC VIP quirks). SUPER-CHIP instructions stop the program with an error. |
| `--dump <frames>` | Run `<frames>` frames (60 per second) with no input, print the screen and exit. |
| `-t [filter]` | Run the tests whose names contain `filter`, or all of them. |

Through `duck`, the arguments go after the package:

```sh
duck run games/flappyduck.8o
duck run path/to/game.ch8 --chip8
duck run -- -t cpu
```

The built binary can also be run directly from `.duck_build/dev/duckling-hackathon`.

### Keys

The 16-key CHIP-8 keypad is mapped onto the left side of the keyboard:

```
CHIP-8 keypad     keyboard
 1 2 3 C          1 2 3 4
 4 5 6 D          q w e r
 7 8 9 E          a s d f
 A 0 B F          z x c v
```

**Esc** or **Ctrl-C** quits. Terminals don't report key releases, so a key counts as held
for 300 ms after its last keystroke; holding a key down keeps it held through keyboard
auto-repeat.

### Running on the Duckling VM

The project also builds to DVM bytecode through the `dvm` profile in `quackconfig.yaml`:

```sh
duck build --profile dvm
duck run --profile dvm games/flappyduck.8o
duck run --profile dvm -- -t
```

`duck run` needs the `VM` binary on your `PATH`. Everything works the same as the
native build, with three differences:

- Starting takes about 7 seconds while the VM loads the bytecode.
- Interactive play is slower: the VM redraws a hires screen about 5 times a second, so
  SUPER-CHIP games run in slow motion. Headless `--dump` runs at full speed.
- The VM passes arguments to the program as a comma-separated list, so a single argument
  can't contain a comma.

## Examples

Write a program in Octo and run it straight away:

```sh
cat > hello.8o <<'EOF'
: main
	v0 := 0xA   v1 := 4   v2 := 4
	i := hex v0
	sprite v1 v2 5
	loop again
EOF
duck run hello.8o
```

Check what a program draws without opening the terminal UI. This prints the screen after
30 frames (half a second):

```sh
$ duck run games/flappyduck.8o --dump 30
................................................................
....####..#......##...###...###...#...#...........######........
....#.....#.....#..#..#..#..#..#...#.#............######........
....###...#.....####..###...###.....#...........####..####......
....#.....#.....#..#..#.....#.......#...........####..####......
....#.....####..#..#..#.....#.......#...........########........
...
```

Run a classic CHIP-8 ROM (not included) with the COSMAC VIP quirks:

```sh
duck run path/to/pong.ch8 --chip8
```

A program that hits an invalid instruction stops with a message instead of hanging:

```sh
$ duck run broken.8o --dump 10
Error: CHIP-8 halted: unknown opcode 0x0123 at 0x0200
```

Run a subset of the tests:

```sh
$ duck run -- -t display
PASS display.flip_collision
PASS display.set_hires
PASS display.shift
...
```

## Games

| File | Machine | Description |
|---|---|---|
| `games/flappyduck.8o` | SUPER-CHIP | Keep the duck in the air and fly through the gaps between the pipes. W flaps. |

FlappyDuck is a good example of what SUPER-CHIP makes possible: the pipes are never
redrawn, the whole screen scrolls left 4 pixels per tick and only a thin strip of the
next pipe is drawn at the edge. The source explains the timing tricks in its header.

## How it is built

The code is organised so that the emulator core knows nothing about the terminal, and the
terminal code knows nothing about CHIP-8 instructions. The main loop connects them: once
per frame it passes key states in and draws the screen out.

| Path | Contents |
|---|---|
| `src/chip8/chip8.dk` | The machine: memory, registers, timers, keypad, and one method per instruction. |
| `src/chip8/isa/` | The instruction set as one table, turned into code at compile time. |
| `src/chip8/platform/` | The quirks of each machine (COSMAC VIP, SUPER-CHIP). |
| `src/chip8/octo/` | The Octo assembler. |
| `src/display/` | The monochrome screen (64x32 or 128x64) shared by the core and the renderer. |
| `src/emulator/` | The 60 Hz loop, keypad handling and headless mode. |
| `src/tui/` | libc bindings (termios, `read`, `write`, `ioctl`, clock), raw-mode terminal, half-block renderer. |
| `src/strings/`, `src/utils/` | String helpers, file reading, command-line arguments. |
| `src/testing/`, `src/tests/` | A small test framework and the tests. |
| `games/` | Games written in Octo. |

A few Duckling features doing the heavy lifting:

- **Compile-time code generation.** `src/chip8/isa/isa.dk` lists every opcode as a
  pattern and a name, such as `"DXYN", "Draw"`. At compile time, `expand` turns that table
  into one class per instruction, the `Instruction` variant, the `decode` function and the
  `match` that dispatches each instruction to its handler. Adding an instruction means
  adding a table row and an `exec` method.
- **Variants instead of flags.** The machine's state is one value:
  `Running | WaitingForFrame | WaitingForKey | Halted | Exited`. Errors are
  `ReadFailed | AsmFailed | RomTooBig | TerminalError | TooSmall | Halted`, and the command
  line parses into `RunTests | RunFile`.
- **Optionals and `Result`.** Lookups that can fail return `?T`, operations that can fail
  return `Result[T, E]` (`Ok[T] | Err[E]`).
- **Tests assembled at compile time.** Test programs are written in Octo inside the tests
  and assembled during compilation with `expand octo(...)`.
- **A destructor restores the terminal.** `Terminal.open()` switches to raw mode and the
  alternate screen; the destructor puts everything back however the program ends.
- **libc through `extern("C")`.** termios, `read`/`write`, `ioctl` and the clock are
  called directly, with no C code in the project.

## Tests

```sh
duck run -- -t
```

The assembler tests can also compare against the
[Timendus CHIP-8 test suite](https://github.com/Timendus/chip8-test-suite). Put it in
`roms/chip8-test-suite/`; the tests look for `<name>.8o` sources and their official
`<name>.ch8` builds in its `bin/` directory and check that the assembler produces the
same bytes. Without the suite those checks are skipped.

## Known issues

Duckling is young, and building this project turned up a number of compiler bugs and
missing features. They are listed with minimal examples and workarounds in
[BUGS.md](BUGS.md). The most visible one for this code: a `bool[N]` field breaks a class's
memory layout, which is why the screen and the keypad store pixels and keys as `u8`.
