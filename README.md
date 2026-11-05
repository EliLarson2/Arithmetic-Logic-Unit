# 8-Bit FPGA Arithmetic Logic Unit

An 8-bit Arithmetic Logic Unit written in **VHDL** and implemented on an **Intel/Altera DE10-Lite FPGA development board**.

I built this project to move beyond isolated combinational-logic exercises and create a more complete digital system with a datapath, control logic, registered inputs, status flags, and physical user interfaces.

One of the main design constraints was the limited number of switches available on the board. There were not enough physical inputs to enter two 8-bit operands and an opcode at the same time, so I designed a finite state machine that reuses the switches. The user enters Operand A, advances the system, enters Operand B, and then selects the operation. This turned a simple ALU into a larger sequential system that coordinated inputs, internal state, computation, and display output.

The complete synthesizable implementation is available in [`src/alu.vhd`](src/alu.vhd).

---

## System Architecture

The design is organized around two operand registers, an 8-bit ALU, control logic, status-flag generation, and output logic for the DE10-Lite LEDs and seven-segment displays.

```text
       DE10-Lite Switches
               |
               v
      +------------------+
      | Input and FSM    |
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
       Result      Status Flags
          |
          v
      +------------------+
      | Display Logic    |
      +------------------+
          |          |
          v          v
        LEDs     7-Segment
                   Displays
```

The ALU performs the selected operation on the stored operands. The result and status flags are then passed to the board-level output logic, where they can be observed through the LEDs and seven-segment displays.

---

## Reusing the Physical Inputs

The DE10-Lite did not have enough switches to dedicate separate physical inputs to both operands and the opcode. Rather than reducing the operand width, I used an FSM to capture the inputs sequentially.

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
Select Opcode / Execute
  |
  v
Idle
```

A pushbutton advances the system through these states. Button edge detection prevents one physical press from being interpreted as multiple state transitions.

This was one of my first projects where an FSM controlled a larger digital system instead of a single isolated behavior. The controller had to determine when each operand should be captured, when the opcode should be interpreted, and when the result should be presented to the user.

---

## ALU Operations

The ALU accepts two 8-bit operands and supports arithmetic, bitwise, shift, pass-through, and comparison operations.

| Category | Operations |
| --- | --- |
| Arithmetic | Addition, subtraction, increment, decrement |
| Bitwise logic | AND, OR, XOR, NOT |
| Shifts | Logical shift left, logical shift right, arithmetic shift right |
| Data movement | Pass Operand A |
| Comparison | Compare Operand A and Operand B |

For arithmetic operations, the design uses a 9-bit intermediate value where needed so information such as carry is available before the final result is reduced to eight bits.

The design also generates processor-style status information, including carry, parity, overflow, zero, negative, and positive or comparison-related conditions. Implementing these flags made the distinction between signed and unsigned interpretation much more concrete. The same eight physical bits can represent different values and produce different conditions depending on how the operation interprets them.

---

## Signed Results and Display Logic

The ALU result is interpreted as an 8-bit two's-complement signed value. To make that value readable on the physical board, the output logic converts the binary result into decimal digits for the seven-segment displays.

The display path determines whether the result is negative, obtains its magnitude, separates that magnitude into decimal digits, and maps each digit to the appropriate seven-segment pattern. The sign and decimal value are then shown on the DE10-Lite.

This part of the project was more involved than simply connecting the ALU result directly to LEDs. It required translating an internal binary representation into an output that was useful to a person, and it created a clear boundary between the computational logic and the presentation logic.

---

## Verification and Debugging

The most valuable part of this project was not implementing another arithmetic operation. It was learning how to isolate a problem in a digital system.

During development, the seven-segment displays initially showed incorrect output. Because the display was the visible result of the system, my first suspicion was that the ALU itself was producing the wrong value. Manually entering operands and opcodes on the board would have been a slow way to test that assumption, and it would not have made the internal signals easy to observe.

Instead, I used a VHDL testbench and simulation during development to apply inputs and opcodes while observing the ALU outputs independently of the physical display path. The simulation showed that the ALU computation was behaving correctly, which narrowed the problem to logic farther downstream in the display and output path.

That experience changed how I approached debugging:

> A visible failure does not necessarily identify the subsystem that caused it. Break the system into functional blocks, define the expected behavior of each block, and verify them independently.

The testbench used during development is not currently included in this repository, so the repository should not be treated as containing a complete automated verification environment. The synthesizable design itself is available in `src/alu.vhd`.

---

## Implementation

The design was developed in **VHDL**, synthesized with **Intel Quartus**, and deployed to the **MAX 10 FPGA** on the DE10-Lite.

The implementation in `src/alu.vhd` contains the FSM, operand registers, ALU operations, status-flag generation, button edge detection, signed-result handling, decimal digit extraction, seven-segment encoding, and board-level output interfaces.

Keeping these functions together made sense while I was learning how the complete system worked, but reviewing the project now also makes the value of clearer module boundaries much more obvious.

---

## What I Would Improve Today

This project represents an earlier stage of my FPGA experience, and there are several changes I would make now. I see those changes less as problems with the original assignment and more as evidence of how my approach to digital design and verification has developed.

### Use Standard Numeric Types

The original implementation uses the older arithmetic packages:

```vhdl
STD_LOGIC_UNSIGNED
STD_LOGIC_ARITH
```

I would now use:

```vhdl
use IEEE.NUMERIC_STD.ALL;
```

with explicit `signed` and `unsigned` types.

That would make the intended numerical interpretation of each signal clearer, reduce ambiguity around conversions, and make the arithmetic more portable between tools. This is particularly important in an ALU because the design intentionally works with both signed and unsigned behavior.

### Separate the Functional Blocks

Much of the original system is contained within a single VHDL architecture. If I redesigned it, I would separate the major responsibilities into clearer modules:

```text
Input Interface
FSM Controller
Operand Registers
ALU Core
Flag Generator
Display Controller
```

That structure would make the datapath and control logic easier to understand. It would also allow the ALU core, flag logic, and display subsystem to be verified independently rather than requiring the entire design to be exercised at once.

The debugging experience already showed why those boundaries matter: the ALU and the display path were separate functional problems even though they were implemented as part of the same system.

### Build a Self-Checking Verification Environment

Simulation helped me isolate the display issue, but today I would go farther and create a self-checking testbench from the beginning.

Instead of relying primarily on waveform inspection, the testbench would apply representative and edge-case inputs, calculate the expected result, compare it against the design output, and report failures automatically. I would verify the status flags as carefully as the result itself, especially around signed overflow, unsigned carry, zero, negative values, and comparison boundaries.

I would also keep that verification code in the repository so the design could be evaluated and reproduced without relying only on the physical FPGA board.

### Make the Datapath Easier to Extend

The original design is fixed at eight bits. A future version could define the datapath width with a VHDL generic, allowing the same ALU structure to be synthesized at different widths.

I would treat parameterization as a secondary improvement rather than the main goal. The first priority would be making the eight-bit implementation modular, readable, and thoroughly verified. Once those foundations were in place, extending the width would be much less risky.

---

## What I Took Away From the Project

This project was my first substantial experience combining a datapath, finite state machine, registered inputs, arithmetic behavior, status flags, and physical FPGA interfaces into one system.

The ALU itself was important, but the larger lesson came from everything around it: working within hardware constraints, coordinating sequential behavior, presenting an internal result through a physical interface, and using simulation to determine which subsystem was actually failing.

It helped shift my mindset from writing logic that *should* work toward designing systems that can be **understood, verified, and debugged when they do not behave as expected**.
