# Day 01 – Introduction to RISC-V ISA

## 1. Objective

The objective of Day 01 is to understand the fundamentals of the RISC-V Instruction Set Architecture and the basic workflow used to compile, analyze, and execute RISC-V programs.

The session focuses on understanding the RISC-V ISA, registers, instructions, number representation, the RISC-V GNU toolchain, C-to-assembly conversion, compiler optimization, disassembly, and RISC-V simulation using Spike.

---

## 2. Introduction to RISC-V

RISC-V is an open standard Instruction Set Architecture based on the Reduced Instruction Set Computer (RISC) design philosophy. It defines the instructions and programmer-visible features that a processor supports.

An Instruction Set Architecture acts as an interface between software and hardware. Software generates instructions according to the ISA, and processor hardware implements those instructions.

RISC-V follows a modular architecture. It provides a basic integer instruction set and allows additional extensions to be included depending on the requirements of the processor.

The open and modular nature of RISC-V makes it suitable for processor research, education, embedded systems, custom processor development, and hardware design.

---

## 3. Instruction Set Architecture (ISA)

An Instruction Set Architecture defines the operations that a processor can understand and execute.

The ISA specifies important processor features such as:

- Instruction set
- Registers
- Instruction formats
- Data representation
- Memory operations
- Arithmetic and logical operations
- Control-flow operations

The ISA defines **what** a processor can do, while the microarchitecture defines **how** those operations are implemented internally.

Therefore, different processors can implement the same RISC-V ISA while using different internal hardware structures.

---

## 4. RISC-V Architecture

RISC-V follows the RISC design philosophy, where instructions are designed to be simple and regular.

The architecture uses a load/store approach. Arithmetic and logical operations are generally performed using registers, while memory is accessed using load and store instructions.

RISC-V provides different base architectures such as RV32 and RV64.

The number indicates the width of the integer registers.

For example:

```text
RV32 → 32-bit architecture
RV64 → 64-bit architecture
```

The base integer instruction set is represented by the letter `I`.

Additional extensions can be combined with the base ISA to provide additional functionality.

---

## 5. RISC-V Registers

RISC-V provides 32 general-purpose integer registers identified as:

```text
x0 – x31
```

These registers are used to store operands, addresses, intermediate values, and results during program execution.

Some important registers are:

| Register | ABI Name | Purpose |
|---|---|---|
| x0 | zero | Constant zero |
| x1 | ra | Return address |
| x2 | sp | Stack pointer |
| x3 | gp | Global pointer |
| x4 | tp | Thread pointer |
| x5–x7 | t0–t2 | Temporary registers |
| x8–x9 | s0–s1 | Saved registers |
| x10–x17 | a0–a7 | Arguments / return values |
| x18–x27 | s2–s11 | Saved registers |
| x28–x31 | t3–t6 | Temporary registers |

The `x0` register is special because it always contains zero.

---

## 6. RISC-V Instruction Formats

RISC-V instructions are organized into different instruction formats.

The major formats are:

```text
R-Type
I-Type
S-Type
B-Type
U-Type
J-Type
```

### R-Type

R-type instructions are mainly used for register-to-register operations.

The instruction contains source registers, a destination register, opcode, and function fields.

```text
funct7 | rs2 | rs1 | funct3 | rd | opcode
```

Here:

- `rs1` is the first source register.
- `rs2` is the second source register.
- `rd` is the destination register.
- `opcode` identifies the instruction category.
- `funct3` and `funct7` provide additional operation information.

### I-Type

I-type instructions contain an immediate value and are commonly used for immediate arithmetic and load operations.

### S-Type

S-type instructions are mainly used for storing register data into memory.

### B-Type

B-type instructions are used for conditional branch operations and change the program
