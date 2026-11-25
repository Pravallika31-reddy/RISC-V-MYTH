
# Day 02 – Application Binary Interface (ABI) and RISC-V Assembly Programming

## 1. Introduction

Day 02 explored the Application Binary Interface (ABI) and its role in connecting software applications with the processor architecture.

The session covered RISC-V register conventions, memory organization, instruction formats and assembly language programming. Practical work included implementing a C function in RISC-V assembly, compiling and executing the program using the RISC-V toolchain and Spike simulator, and exploring PicoRV32 RTL and firmware files.

## 2. Concepts Studied

### 2.1 Application Programming Interface (API)

An Application Programming Interface (API) provides predefined functions and services that allow software applications to interact with libraries.

Examples:
- C – `stdio.h`
- Java – `java.text`

APIs simplify application development by allowing programmers to use existing functions instead of implementing every operation from scratch.

### 2.2 Application Binary Interface (ABI)

The Application Binary Interface defines the low-level rules that allow compiled programs to communicate with each other and interact with the execution environment.

ABI specifies:
- Register usage.
- Function calling conventions.
- Argument passing mechanisms.
- Return value handling.
- Stack organization.
- Binary compatibility and system-level interfaces.

The relationship between software and hardware was studied through the following flow:

**Application → API → ABI / System Interface → Operating System → ISA → RTL → Hardware**

### 2.3 RISC-V Register Architecture

RISC-V provides 32 general-purpose integer registers, identified as `x0` to `x31`.

Important register concepts:
- Each register is identified using a 5-bit register field in the instruction encoding.
- XLEN represents the width of the integer registers.
- RV32 uses 32-bit integer registers.
- RV64 uses 64-bit integer registers.

### 2.4 Register Naming Convention

RISC-V uses ABI names to describe the purpose of individual registers.

| Register | ABI Name | Description |
|---|---|---|
| x0 | zero | Constant zero |
| x1 | ra | Return address |
| x2 | sp | Stack pointer |
| x3 | gp | Global pointer |
| x4 | tp | Thread pointer |
| x5–x7 | t0–t2 | Temporary registers |
| x8 | s0/fp | Saved register / Frame pointer |
| x9 | s1 | Saved register |
| x10–x11 | a0–a1 | Function arguments / Return values |
| x12–x17 | a2–a7 | Function arguments |
| x18–x27 | s2–s11 | Saved registers |
| x28–x31 | t3–t6 | Temporary registers |

The following register conventions were also examined:

- Caller-saved registers: The calling function must preserve values it needs after a function call.
- Callee-saved registers: The called function must restore these registers if it modifies them.
- Argument registers: `a0`–`a7` carry integer function arguments.
- Return register: `a0` generally carries the integer return value.

### 2.5 Memory Organization and Endianness

Endianness defines the order in which bytes of a multi-byte value are stored in memory.

**Little-Endian**
- Least Significant Byte (LSB) is stored at the lowest memory address.
- Most Significant Byte (MSB) is stored at the highest memory address.

**Big-Endian**
- Most Significant Byte (MSB) is stored at the lowest memory address.
- Least Significant Byte (LSB) is stored at the highest memory address.

Example: Consider the 32-bit value `0x12345678`.

| Memory Address | Little-Endian | Big-Endian |
|---|---|---|
| 1000 | 78 | 12 |
| 1001 | 56 | 34 |
| 1002 | 34 | 56 |
| 1003 | 12 | 78 |

Additional observations:
- 1 Byte = 8 bits.
- 8 Bytes = 64 bits.
- Memory is byte-addressable.
- The practical environment used little-endian memory ordering.

### 2.6 RISC-V Load Instruction – LD

The `ld` instruction loads a 64-bit double word from memory into a destination register.

**Syntax:**

```asm
ld rd, offset(rs1)
```

**Example:**

```asm
ld x8, 16(x23)
```

Explanation:
- `x8` – Destination register.
- `x23` – Base address register.
- `16` – Immediate offset.
- Effective address = contents of `x23` + 16.

If `x23` contains zero, the effective address is 16.

### 2.7 RISC-V Addition Instruction – ADD

The `add` instruction performs addition between two source registers and stores the result in a destination register.

**Syntax:**

```asm
add rd, rs1, rs2
```

**Example:**

```asm
add x8, x24, x8
```

The contents of `x24` and `x8` are added, and the result is stored in `x8`.

### 2.8 RISC-V Store Instruction – SD

The `sd` instruction stores a 64-bit double word from a register into memory.

**Syntax:**

```asm
sd rs2, offset(rs1)
```

**Example:**

```asm
sd x8, 8(x23)
```

Explanation:
- `x8` – Register containing the data.
- `x23` – Base address register.
- `8` – Immediate offset.
- Effective address = contents of `x23` + 8.

### 2.9 RISC-V Instruction Formats

The practical session introduced three commonly used RISC-V instruction formats.

**I-Type Format**

Used for instructions involving a register and an immediate value.

Example:

```asm
ld x8, 16(x23)
```

Fields:
- Immediate
- rs1
- funct3
- rd
- opcode

**R-Type Format**

Used for register-to-register arithmetic and logical operations.

Example:

```asm
add x8, x24, x8
```

Fields:
- funct7
- rs2
- rs1
- funct3
- rd
- opcode

**S-Type Format**

Used for storing register data into memory.

Example:

```asm
sd x8, 8(x23)
```

Fields:
- Immediate (split across instruction fields)
- rs2
- rs1
- funct3
- opcode

These standard instruction formats use 32-bit instruction encodings.

---

## 3. Practical Implementation – C and Assembly Integration

### 3.1 Objective

To implement a function that calculates the sum of integers from 0 to 9 using RISC-V assembly language and call that function from a C program.

### 3.2 C Source Program

**File: `1to9_custom.c`**

```c
#include <stdio.h>

extern int load(int x, int y);

int main()
{
    int result = 0;
    int count = 9;

    result = load(0, count + 1);

    printf("Sum of numbers from 0 to %d is %d\n",
           count, result);

    return 0;
}
```

The C program declares an external function named `load()`. Its implementation is provided separately in the assembly source file.

### 3.3 RISC-V Assembly Implementation

**File: `load.S`**

```asm
.section .text
.global load
.type load, @function

load:
    add  a4, a0, zero
    add  a2, a0, a1
    add  a3, a0, zero

loop:
    add  a4, a3, a4
    addi a3, a3, 1
    blt  a3, a2, loop

    add  a0, a4, zero
    ret
```

### 3.4 Assembly Instruction Explanation

| Instruction | Purpose |
|---|---|
| `add` | Adds two register values |
| `addi` | Adds an immediate value to a register |
| `blt` | Branches when one signed value is less than another |
| `ret` | Returns from a function |
| `zero` | Constant-zero register |
| `a0` | First argument and return-value register |
| `a1` | Second argument register |
| `a2–a4` | Registers used by the assembly routine |

### 3.5 Working of the Assembly Function

The function receives its arguments through `a0` and `a1`.

The operations are performed as follows:

1. The initial value is copied into register `a4`.
2. The upper loop limit is calculated and stored in `a2`.
3. Register `a3` is initialized as the loop counter.
4. The loop adds the counter value to the accumulated sum.
5. `addi` increments the counter by one.
6. `blt` checks whether the counter is less than the upper limit.
7. After the loop finishes, the accumulated result is copied into `a0`.
8. `ret` transfers control back to the calling C program.

### 3.6 Compilation Using RISC-V GCC

The C and assembly source files are compiled together using the RISC-V cross-compiler.

```bash
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o 1to9_custom.o 1to9_custom.c load.S
```

This command:
- Uses the RISC-V GCC cross-compiler.
- Applies the optimization level `-Ofast`.
- Selects the LP64 ABI.
- Targets the RV64I instruction set.
- Combines the C and assembly source files into an executable.

### 3.7 Execution Using Spike Simulator

The compiled program is executed using the Spike RISC-V ISA simulator and proxy kernel.

```bash
spike pk 1to9_custom.o
```

**Expected output:**

```text
Sum of numbers from 1 to 9 is 45
```

The output confirms that the assembly function calculates the required sum and returns the result to the C program.

### 3.8 Disassembly Analysis

The generated executable was examined using the RISC-V GNU Objdump utility.

```bash
riscv64-unknown-elf-objdump -d 1to9_custom.o | less
```

Disassembly analysis helps identify:
- RISC-V machine instructions.
- Function entry and exit operations.
- Register usage.
- Arithmetic and branch instructions.
- The relationship between assembly code and machine code.


---
<img width="1920" height="923" alt="VirtualBox_Risc-v_24_11_2026_11_54_51" src="https://github.com/user-attachments/assets/a039f87e-58ff-4578-9e8b-c645c573dd00" />

## 4. PicoRV32 Processor and Firmware Exploration

The workshop also introduced the PicoRV32 processor implementation and its associated firmware files.

### 4.1 Files Examined

| File | Description |
|---|---|
| `picorv32.v` | Verilog RTL implementation of the PicoRV32 processor |
| `firmware.hex` | Firmware represented in hexadecimal format |
| `firmware32.hex` | 32-bit firmware hexadecimal representation |
<img width="955" height="910" alt="VirtualBox_Risc-v_24_11_2026_12_27_50" src="https://github.com/user-attachments/assets/b5dc564e-cd24-48df-bc5c-74d58577cf60" />


### 4.2 Activities

- Opened and examined the PicoRV32 RTL source file.
- Observed processor configuration parameters.
- Explored the Verilog modules and processor implementation.
- Inspected firmware hexadecimal data.
- Examined the relationship between instruction representations and firmware contents.

This exercise provided an introduction to how RISC-V software and processor RTL are related.

---

## 5. Development Environment and Tools

The following tools and technologies were used during the practical session:

| Tool / Technology | Application |
|---|---|
| Ubuntu Linux | Development environment |
| RISC-V GCC | Cross-compilation |
| Spike | RISC-V instruction set simulation |
| RISC-V GNU Objdump | Disassembly analysis |
| C Programming | Application source code |
| RISC-V Assembly | Low-level function implementation |
| Verilog HDL | Processor RTL exploration |
| PicoRV32 | RISC-V processor implementation |

## 6. Practical Execution and Results
### 6.0 RISC-V Assembly Execution – Custom 1 to 9 Program

A custom program was implemented by rewriting the C logic using RISC-V assembly language.

The program demonstrates the interaction between C code and assembly routines and the execution of arithmetic operations using RISC-V instructions.

**Program:** `1to9_custom.c`

<img width="800" height="600" alt="VirtualBox_RISCV_25_11_2026_11_42_05" src="https://github.com/user-attachments/assets/d4574434-3428-4a97-ae99-0f8913575410" />



### 6.1 RISC-V Assembly Program Execution

The custom C program was rewritten using RISC-V assembly language. The program was compiled using the RISC-V GCC toolchain.

### 6.2 RISC-V Assembly Disassembly

Explored the generated assembly instructions and understood how C code is translated into RISC-V machine instructions.
<img width="800" height="600" alt="VirtualBox_RISCV_25_11_2026_11_03_26" src="https://github.com/user-attachments/assets/7772c18e-2047-41f4-b712-8f8cda8c3dcb" />


### 6.3 PicoRV32 RTL Source Code

Explored the PicoRV32 Verilog RTL source code to understand the processor implementation.

### 6.4 Firmware Hexadecimal File

Generated and examined the firmware hexadecimal file.
<img width="800" height="600" alt="VirtualBox_RISCV_25_11_2026_11_43_32" src="https://github.com/user-attachments/assets/b6889074-5e7b-45c7-a712-3c0c1deb532e" />


### 6.5 32-bit Firmware Hexadecimal File

Converted and examined the firmware in 32-bit hexadecimal format.

<img width="800" height="600" alt="VirtualBox_RISCV_25_11_2026_11_53_40" src="https://github.com/user-attachments/assets/84a6f322-ba46-499a-be07-ec5d27aa10dc" />

## 7. Key Learnings

- Understood Application Binary Interface (ABI).
- Learned RISC-V register architecture and ABI register names.
- Studied caller-saved and callee-saved registers.
- Understood little-endian and big-endian memory organization.
- Learned RISC-V load and store instructions.
- Studied RISC-V instruction formats.
- Practiced writing RISC-V assembly programs.
- Explored the compilation process using the RISC-V GCC toolchain.
- Explored PicoRV32 RTL and firmware hexadecimal files.

## 8. Conclusion

Day 2 provided practical exposure to RISC-V assembly programming, ABI concepts, register architecture, memory organization and instruction formats.

The session helped in understanding the relationship between C programming, assembly language, machine instructions and processor hardware through practical compilation and simulation exercises.




## 6. Key Takeaways

- Studied the difference between API and ABI.
- Understood the connection between software interfaces and processor architecture.
- Explored RISC-V general-purpose registers and ABI naming conventions.
- Learned caller-saved and callee-saved register behavior.
- Studied little-endian and big-endian memory organization.
- Practiced RISC-V load, store and arithmetic instructions.
- Identified I-Type, R-Type and S-Type instruction formats.
- Implemented an assembly function called from a C program.
- Compiled C and assembly source files using RISC-V GCC.
- Executed the compiled program using Spike.
- Examined generated machine instructions using Objdump.
- Explored PicoRV32 RTL and firmware hexadecimal files.

## 7. Conclusion

Day 02 provided practical exposure to the RISC-V Application Binary Interface, register conventions, memory organization and instruction encoding.

The main implementation involved translating a numerical summation routine into RISC-V assembly and integrating it with a C program. The program was compiled using the RISC-V GCC toolchain, executed through the Spike simulator and inspected using Objdump.

The session also introduced PicoRV32 RTL and firmware exploration, helping establish a connection between high-level software, assembly programming and processor hardware.

---

**References**

- [RISC-V Specifications](https://riscv.org/technical/specifications/)
- [RISC-V GNU Toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain)
- [Spike RISC-V ISA Simulator](https://github.com/riscv-software-src/riscv-isa-sim)
- [PicoRV32 Repository](https://github.com/YosysHQ/picorv32)

