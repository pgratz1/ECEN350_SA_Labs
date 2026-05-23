# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the ECEN 350 (Computer Organization) lab repository for the study abroad program. Labs progress through two major domains:
- **Labs 1–3**: ARMv8/LEGv8 assembly programming, called from C via `gcc`
- **Labs 4–6**: Verilog hardware design building toward a single-cycle LEGv8 processor

## Build Commands

### C/Assembly Labs (Lab01–Lab03)
Each lab has its own `Makefile`. From within a lab directory:
```sh
make          # build the target binary
make clean    # remove object files and binary
./LabXX       # run the compiled program
```
All C/Assembly labs compile with `gcc -g -O0` for maximum debuggability. Lab01 has three parts (part1, part2, part3), each with its own Makefile.

### Verilog Labs (Lab04–Lab06)
No Makefile is provided; use Icarus Verilog directly:
```sh
# Compile and simulate a module + its testbench
iverilog -o sim Lab04/GatesTest.v Lab04/Gates.v && vvp sim

# For Lab06 (full processor):
iverilog -o sim Lab06/SingleCycleProcTest.v Lab06/SingleCycleProc.v Lab06/SingleCycleControl.v \
         Lab06/InstructionMemory.v Lab06/DataMemory.v Lab04/RegisterFile.v Lab05/ALU.v \
         Lab05/NextPClogic.v Lab05/SignExtender.v Lab04/mux2to1_8.v && vvp sim

# View waveforms (VCD output from $dumpfile in testbenches)
gtkwave GatesTest.vcd
```

## Architecture

### C/Assembly Labs Pattern
Each lab pairs a `main.c` driver with an assembly file (`.S`). The C file declares an `extern` function implemented in assembly; `gcc` compiles and links them. The ARMv8 calling convention places arguments in X0–X7 and returns in X0. The stack must be kept 16-byte aligned.

### Verilog Single-Cycle Processor (Labs 4–6)
The processor implements a LEGv8 (textbook ARMv8 subset) instruction set. Components are built across labs and assembled in Lab06:

| Module | File | Role |
|---|---|---|
| `Gates` | Lab04/Gates.v | AND/OR/XOR primitives |
| `mux2to1_8` | Lab04/mux2to1_8.v | 8-bit 2-to-1 mux |
| `RegisterFile` | Lab04/RegisterFile.v | 32×64-bit register file |
| `ALU` | Lab05/ALU.v | 64-bit ALU (AND/OR/ADD/SUB/PassB) |
| `SignExtender` | Lab05/SignExtender.v | Sign-extends I/D/CB/B-type immediates |
| `NextPClogic` | Lab05/NextPClogic.v | Computes next PC (branch/unconditional/+4) |
| `SC_Control` | Lab06/SingleCycleControl.v | Decodes 11-bit opcode into control signals |
| `InstructionMemory` | Lab06/InstructionMemory.v | ROM encoded as a case statement |
| `DataMemory` | Lab06/DataMemory.v | 1024-byte RAM, big-endian, clock-edge R/W |
| `SingleCycleProc` | Lab06/SingleCycleProc.v | Top-level datapath; students wire components |

`Lab06/SingleCycleProcTest.v` is the testbench. It drives `reset`/`startpc`, runs the clock, and checks `MemtoRegOut` after programs complete. A watchdog timer terminates runaway simulations after 255 clock cycles.

**Supported opcodes** (defined in `Lab06/SingleCycleControl.v`): `LDUR`, `STUR`, `ADD`, `SUB`, `AND`, `ORR`, `CBZ`, `B`, `MOVZ`, `ADDI`, `SUBI`.

**ALU control codes** (defined in `Lab05/ALU.v`): `AND=0000`, `OR=0001`, `ADD=0010`, `SUB=0110`, `PassB=0111`.

**SignExtender** selects encoding via `SignOp[1:0]`: `I-type=00`, `D-type=01`, `CB-type=10`, `B-type=11`.
