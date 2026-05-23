# ECEN 350 Lab Files — Study Abroad Program

Starting-point source files for the ECEN 350 (Computer Organization) lab sequence,
study abroad edition. Labs 1–3 cover ARMv8 assembly programming; Labs 4–6 build a
single-cycle processor in Verilog.

## Prerequisites

| Tool | Used by |
|------|---------|
| `gcc`, `make` | Labs 1–3 (C/assembly) |
| `iverilog`, `vvp` | Labs 4–6 (Verilog simulation) |
| `gtkwave` *(optional)* | Viewing VCD waveform output from testbenches |

## Building and Running

**C/Assembly labs (Lab01–Lab03)** — each part has its own Makefile:

```sh
cd Lab02          # or Lab01/part1, Lab01/part2, etc.
make              # compiles and links
./Lab02           # run the binary
make clean        # remove build artifacts
```

**Verilog labs (Lab04–Lab06)** — use Icarus Verilog directly:

```sh
# Lab04 example
iverilog -o sim Lab04/GatesTest.v Lab04/Gates.v && vvp sim

# Lab06 full processor
iverilog -o sim Lab06/SingleCycleProcTest.v Lab06/SingleCycleProc.v \
         Lab06/SingleCycleControl.v Lab06/InstructionMemory.v \
         Lab06/DataMemory.v Lab04/RegisterFile.v Lab05/ALU.v \
         Lab05/NextPClogic.v Lab05/SignExtender.v Lab04/mux2to1_8.v \
         && vvp sim

# Optional: open the generated waveform
gtkwave GatesTest.vcd
```

## Labs

### Lab 1 — Introduction to ARMv8 Assembly (three parts)

A three-part introduction to writing ARMv8 assembly functions that are called from C.

**Part 1** — Introduces the function call interface and the ARMv8 calling convention:
how arguments are passed to a function, where the return value is placed, and how to
return control to the caller.

**Part 2** — Exercises the ARMv8 shift instructions (`LSR`, `LSL`). Students
customize the shift amount using a digit from their student ID, giving each person a
unique program to analyze and run.

**Part 3** — Covers memory addressing and the `.data` section. Students work with
the `ADR` instruction and the family of load instructions (`LDURB`, `LDURSH`,
`LDURSW`, `LDUR`) that read values of different widths (byte through quad-word) from
memory.

### Lab 2 — Branches and Loops

Introduces the ARMv8 branch instructions — conditional (`CBZ`) and unconditional
(`B`) — and how to use them to build loops. Students read and analyze an existing
assembly program to understand its behavior before writing their own.

### Lab 3 — Functions and the Stack

Covers the stack and callee-saved registers. Students implement a function that
requires saving and restoring a register across a loop, using `STUR`/`LDUR` with the
stack pointer and observing the 16-byte alignment requirement.

### Lab 4 — Introduction to Verilog

First exposure to hardware description with Verilog:

- **`Gates.v` / `GatesTest.v`** — a combinational logic module with an accompanying
  testbench; introduces the simulate-and-inspect workflow and VCD waveform output.
- **`mux2to1_8.v`** — an 8-bit 2-to-1 multiplexer.
- **`RegisterFile.v`** — a 32 × 64-bit register file with two read ports and one
  clocked write port.

### Lab 5 — ALU and Datapath Components

Students implement the core arithmetic and control-flow building blocks of a
processor:

- **`ALU.v`** — a 64-bit arithmetic/logic unit with a `Zero` status output.
- **`SignExtender.v`** — immediate sign extension for the four ARMv8 instruction
  encoding formats (I, D, CB, B-type).
- **`NextPClogic.v`** — next program counter selection logic supporting sequential
  execution, conditional branches, and unconditional branches.

### Lab 6 — Single-Cycle Processor

Students connect all previously-built components into a functioning single-cycle
processor by completing the datapath wiring in `SingleCycleProc.v`.

Supporting modules (fully provided):

| Module | File | Role |
|---|---|---|
| `SC_Control` | `SingleCycleControl.v` | Decodes 11-bit opcode → control signals |
| `InstructionMemory` | `InstructionMemory.v` | ROM with a pre-loaded test program |
| `DataMemory` | `DataMemory.v` | 1 KB byte-addressable RAM, big-endian |

Supported instructions: `LDUR`, `STUR`, `ADD`, `SUB`, `AND`, `ORR`, `CBZ`, `B`, `MOVZ`.

`SingleCycleProcTest.v` runs the processor against a test program and verifies the
result; a watchdog timer halts simulation if the program does not terminate.
