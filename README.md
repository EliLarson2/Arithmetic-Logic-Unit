# 8-Bit FPGA Arithmetic Logic Unit

An **8-bit Arithmetic Logic Unit (ALU)** implemented in **VHDL** and deployed on an **Intel/Altera DE10-Lite FPGA development board**.

## About the Project

The goal of this project was to move beyond basic combinational logic and build a more complete digital system resembling a simplified processor datapath.

I wanted to gain hands-on experience with:

- Designing arithmetic and logic functions in VHDL
- Developing a finite state machine to control system behavior
- Working with signed and unsigned binary arithmetic
- Implementing processor-style status flags
- Interfacing digital logic with physical FPGA inputs and outputs
- Developing a repeatable verification and debugging process
- Understanding how a design moves from VHDL into physical FPGA hardware

A major challenge was working around the limited number of physical switches on the DE10-Lite. Rather than requiring separate switches for both operands and the opcode, I used an FSM to sequentially load each input into internal registers before executing the selected operation.

---

## System Architecture

The system uses the DE10-Lite switches to enter two 8-bit operands and a 4-bit opcode.

A pushbutton advances the finite state machine through the input and execution process.

```text
       DE10-Lite Switches
               |
               v
      +------------------+
      | Input / FSM      |
      | Control Logic    |
      +------------------+
               |
        +------+------+
        |             |
        v             v
   Register A     Register B
        |             |
        +------+------+
               |
               v
      +------------------+
      |    8-Bit ALU     |
      +------------------+
          |          |
          v          v
       Result      Flags
          |
          v
      +------------------+
      | Display Logic    |
      +------------------+
          |          |
          v          v
        LEDs    7-Segment
                 Display
```

---

## Finite State Machine

Because the DE10-Lite does not provide enough switches to enter both 8-bit operands and the opcode simultaneously, the system reuses the physical inputs.

The FSM sequences the user through:

```text
Idle
  |
  v
Load Operand A
  |
  v
Load Operand B
  |
  v
Select Opcode / Operate
  |
  v
Idle
```

A button edge-detection mechanism prevents a single physical button press from unintentionally advancing through multiple states.

This was one of the first projects where I used an FSM to coordinate multiple parts of a larger digital system instead of implementing isolated logic functions.

---

## ALU Operations

The ALU accepts two 8-bit operands and supports arithmetic, logical, shift, and comparison functions.

### Arithmetic

- Addition
- Subtraction
- Increment
- Decrement

### Bitwise Logic

- AND
- OR
- XOR
- NOT

### Shift Operations

- Logical shift left
- Logical shift right
- Arithmetic shift right

### Additional Operations

- Pass Operand A
- Compare Operand A and Operand B

For arithmetic operations, a 9-bit intermediate value is used where appropriate so that information such as carry can be retained before the final 8-bit result is stored.

---

## Processor-Style Status Flags

The design generates several status flags based on the result of an operation:

- **Carry**
- **Parity**
- **Overflow**
- **Zero**
- **Negative**
- **Positive / Greater-than conditions**

Implementing these flags helped reinforce the differences between signed and unsigned arithmetic and how processors retain information about the result of an operation beyond the result value itself.

---

## Signed Output and Display Logic

The ALU result is interpreted as an 8-bit signed value using two's-complement representation.

For the physical DE10-Lite display, the result is converted into a human-readable decimal representation.

The display subsystem:

1. Determines whether the result is negative
2. Converts the result to its absolute magnitude
3. Separates the value into decimal digits
4. Converts each digit into the appropriate seven-segment pattern
5. Displays the sign and result on the DE10-Lite

This portion of the project became particularly useful during debugging because it demonstrated the difference between **correct internal computation** and **correct presentation of that computation to the user**.

---

## Verification and Debugging

One of the most valuable parts of this project was learning to debug the system systematically.

During development, the seven-segment displays initially produced incorrect output.

My first suspicion was that the ALU itself was calculating the wrong results. Manually entering every possible opcode and comparing outputs, however, would have been slow and would not have efficiently isolated the problem.

Instead, I developed a simulation-based verification approach using a **VHDL testbench**.

The testbench allowed different inputs and opcodes to be applied systematically while observing the internal ALU outputs independently from the physical display hardware.

This showed that the ALU computation was behaving correctly and allowed the problem to be isolated farther downstream in the display/output logic.

### Debugging lesson

This project taught me an engineering principle that I now apply to hardware and embedded-system debugging:

> **A visible failure does not necessarily identify the subsystem causing the failure. Break the system into functional blocks, determine the expected behavior of each block, and verify them independently.**

That approach later became useful when troubleshooting embedded hardware, communication interfaces, and custom PCBs.

---

## Source Code

The complete synthesizable VHDL implementation is available in:

```text
src/alu.vhd
```

The design contains:

- FSM state control
- Operand registers
- ALU operations
- Status-flag generation
- Button edge detection
- Signed-result handling
- Decimal digit extraction
- Seven-segment encoding
- DE10-Lite output interfaces

---

## What I Learned

This project gave me practical experience with:

- **VHDL**
- **FPGA development**
- **Finite state machines**
- **Digital datapath design**
- **Registers and sequential logic**
- **Combinational logic**
- **Two's-complement arithmetic**
- **Carry and overflow behavior**
- **Status flags**
- **Edge detection**
- **Seven-segment displays**
- **Simulation and testbenches**
- **Waveform-based debugging**
- **System-level decomposition**
- **Physical FPGA implementation**

More importantly, the project helped shift my approach from simply writing logic that should work toward thinking about **how a digital system can be structured, tested, and debugged when it does not work as expected**.

---

## What I Would Improve Today

This was an academic project and there are several improvements I would make if I redesigned it with my current hardware and verification experience.

### 1. Modernize the VHDL arithmetic

The original implementation uses:

```vhdl
STD_LOGIC_UNSIGNED
STD_LOGIC_ARITH
```

A future revision would use the IEEE-standard:

```vhdl
use IEEE.NUMERIC_STD.ALL;
```

with explicit `signed` and `unsigned` types.

This would make arithmetic behavior more explicit and improve portability and readability.

### 2. Break the design into clearer modules

Instead of keeping much of the system in a single architecture, I would separate the design into functional modules such as:

```text
ALU Core
FSM Controller
Flag Generator
Display Controller
Input Interface
```

This would make each subsystem easier to independently verify and reuse.

### 3. Develop a self-checking testbench

Rather than relying primarily on waveform inspection, I would create a testbench that automatically:

- Applies test vectors
- Calculates the expected result
- Compares expected and actual outputs
- Tests arithmetic edge cases
- Verifies status flags
- Reports pass/fail results

### 4. Parameterize the datapath

The original design is fixed at 8 bits.

A future version could use a generic data width so the same ALU could be synthesized as an 8-bit, 16-bit, or wider architecture.

### 5. Improve the input interface

The FSM was necessary because the physical switch count limited how operands could be entered.

A future version could use **UART** to send operands and opcodes from a computer, allowing automated testing and eliminating the physical-switch limitation.

---

## Future Direction

This project gave me my first substantial experience building and debugging a larger FPGA-based digital system.

Future FPGA projects will build on this foundation by moving toward:

- Streaming digital data
- FIR filtering
- FIFO buffering
- Digital signal processing
- Hardware/software interfaces
- Automated verification
- FPGA-based signal-processing architectures

The long-term goal is to apply the digital-design and verification techniques learned here to more advanced **embedded, signal-processing, and RF-oriented systems**.

---

## Platform

- **FPGA:** Intel/Altera MAX 10
- **Development Board:** DE10-Lite
- **HDL:** VHDL
- **Development Environment:** Intel Quartus
- **Datapath Width:** 8 bits
