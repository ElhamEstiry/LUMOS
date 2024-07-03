<!-- <img src="https://github.com/IUST-Computer-Organization/.github/blob/main/images/CompOrg_orange.png" alt="Image" width="85" height="85" style="vertical-align:middle"> LUMOS RISC-V -->
Computer Organization - Spring 2024
==============================================================
## Iran Univeristy of Science and Technology
## Assignment 1: Assembly code execution on phoeniX RISC-V core

- Name: Elham Estiry
- Team Members: Yasaman Nemati - Sanaz Motie
- Student ID: 99411065 - 99413226 - 99413082
- Date: 13.04.1403



## Introduction

The primary aim of this project is to explore and implement a multi-cycle RISC-V processor with an additional fixed-point arithmetic unit. The project involves programming the processor to execute a simple RISC-V assembly code that calculates the distances between points on a map. The processor used for this project is the LUMOS RISC-V core, which stands for Light Utilization with Multicycle Operational Stages. This core is designed for educational purposes and supports a subset of the 32-bit base integer ISA of RISC-V.

### LUMOS RISC-V Core

The LUMOS core operates using a multi-cycle implementation where each instruction is broken down into several steps, each taking one clock cycle to complete. This design optimizes the use of various functional units, allowing them to be reused across different clock cycles, thus reducing hardware requirements. The processor's design is divided into two main parts:

1. **Data Path Design**: Focuses on designing the ALU and other functional units, and managing access to registers and memory.
2. **Control Path Design**: Involves creating state machines to decode instructions and generate control signals necessary for manipulating the data path.

### Data Path Implementation

The data path implementation is detailed in the LUMOS.v file, and includes components such as the program counter, memory interface, instruction register, ALU, and register file. The process of executing an instruction involves several stages:

1. **Instruction Fetch and Decode**: The instruction is fetched from memory and decoded to determine the operation.
2. **Operand Fetch and Immediate Generation**: The necessary operands are fetched from the register file, and immediate values are generated.
3. **Execution**: The ALU performs the required operation based on the fetched operands and immediate values.
4. **Write-Back**: The result of the operation is written back to the register file.

### Fixed-Point Unit (FPU)

The project extends the LUMOS RISC-V core with a Fixed-Point Unit (FPU) capable of performing fixed-point arithmetic operations like addition, subtraction, multiplication, and square root calculation. The FPU operations are less precise than single-precision floating-point operations but are suitable for the project's requirements.

1. **Fixed-Point Arithmetic**: Uses Q notation (Qi.f) to represent numbers, where `i` is the number of integer bits and `f` is the number of fractional bits. This notation allows binary operations to yield correct results for fixed-point numbers.
2. **Fixed-Point Multiplication**: The multiplication process involves simple binary multiplication with extra consideration for the number of bits in the product. The fixed-point position shifts after multiplication, requiring adjustments to maintain the correct format.
3. **Fixed-Point Square Root Calculation**: Implements an Algorithmic State Machine (ASM) to calculate the square root using subtraction and bit shifts. This method provides an approximate answer suitable for the project's needs.


## SQRT Part

Square Root Unit Breakdown

The Verilog code begins with the implementation of a square root unit. Here's a step-by-step explanation of its functionality:
Registers and Parameters

    root and root_ready: Registers to store the computed square root and indicate readiness.
    current_square_phase and next_square_phase: 2-bit state machine registers to manage the computation phases.
    sqrt_function_begin and sqrt_function_busy: Control signals to initiate and track the square root calculation process.
    x, q, ac, and test_result: Intermediate registers used in the computation.
    ITER: A local parameter that determines the number of iterations needed for the calculation based on operand width and fractional bits.
    valid: An unused register in this context.

State Machine Logic

    The state machine is updated with each clock cycle when the square root operation is triggered (operation == FPU_SQRT`).
    The phases are managed in the always @(posedge clk) block:
        If the square root operation is active, the state transitions to the next phase.
        Otherwise, the state resets, and root_ready is cleared.

Phase Transition Logic

    In the always @(*) block, the next phase is determined based on the current phase:
        Phase 0: Initialization (sets sqrt_function_begin to 0).
        Phase 1: Begins the square root function (sets sqrt_function_begin to 1).
        Phase 2: Continues the calculation.
    This phase logic controls the progression of the square root computation.

Iterative Square Root Calculation

    The Newton-Raphson method is used for the iterative calculation:
        x: The radicand.
        q: The result of the square root calculation.
        ac: Accumulator for intermediate values.
        test_result: Used to determine the next value of ac and x.
    Depending on the value of test_result[WIDTH + 1], the next values of ac and x are adjusted, and q is either incremented or shifted.

Clock-Driven Updates

    The always @(posedge clk) block manages the iterative computation:
        On starting (sqrt_function_begin), variables are initialized, and the calculation begins.
        During the busy state (sqrt_busy), intermediate values are updated in each iteration until the computation is complete.
        When the final iteration is reached (i == ITER-1), sqrt_busy is cleared, and root_ready is set, storing the result in root.


 ## Multiplication Part

  Detailed Explanation of the Multiplication Module

The Verilog code implements a sophisticated Multiplier Circuit designed to perform 32-bit multiplication using smaller 16-bit operations. Here’s a breakdown of the key components and processes involved:
Initial Setup and Registers

    product: A 64-bit register that holds the final multiplication result.
    product_ready: A signal that indicates when the multiplication process is complete.
    multiplierCircuitInput1 and multiplierCircuitInput2: These 16-bit registers are used to provide inputs to the internal multiplier module.
    multiplierCircuitResult: A 32-bit wire that captures the output of the multiplication.

Multiplier Module

An instance of the Multiplier module is used to handle 16-bit multiplications. This module multiplies two 16-bit numbers and outputs a 32-bit result:

verilog

Multiplier multiplier_circuit (
    .operand_1(multiplierCircuitInput1),
    .operand_2(multiplierCircuitInput2),
    .product(multiplierCircuitResult)
);

Phased Multiplication Process

To multiply two 32-bit numbers, the process is divided into several phases, each handling a portion of the calculation. The results of these phases are then combined to form the final product.
Phases of Operation

    Initialization Phase:
        All control signals and intermediate registers are reset.
        The next_mul_phase is set to start the first phase of multiplication.

    Phase 1: Lower 16-bits Multiplication:
        Multiplies the lower 16 bits of both operands.
        Stores the result in partialProduct1.

    Phase 2: Upper-Lower Multiplication:
        Multiplies the upper 16 bits of operand_1 with the lower 16 bits of operand_2.
        Stores the result in partialProduct2.

    Phase 3: Lower-Upper Multiplication:
        Multiplies the lower 16 bits of operand_1 with the upper 16 bits of operand_2.
        Stores the result in partialProduct3.

    Phase 4: Upper 16-bits Multiplication:
        Multiplies the upper 16 bits of both operands.
        Stores the result in partialProduct4.

    Phase 5: Final Product Calculation:
        Combines the partial products to compute the final 64-bit result:

        verilog

        product <= partialProduct1 + (partialProduct2 << 16) + (partialProduct3 << 16) + (partialProduct4 << 32);

        Sets product_ready to indicate that the final product is available.

State Machine

A state machine is used to control the phases of the multiplication process. The current and next phases are managed using current_mul_phase and next_mul_phase registers.

    State Machine Logic:
        The state transitions occur on the rising edge of the clock (posedge clk).
        The operation signal determines if the multiplication process should start or reset.



Summary

Each phase computes a partial product, which is then combined in the final phase to produce the full 64-bit result. The state machine ensures that each phase is executed in the correct order, and the product_ready signal indicates when the computation is complete. This modular and phased approach makes the design efficient and scalable.

## Wave Form

### SQRT:
![alt text](<Screenshot 2024-07-04 000638.png>)

### Multiplication
![alt text](<Screenshot 2024-07-04 000650.png>)

### Final
![alt text](<Screenshot 2024-07-04 000704.png>)

We should be able to see 1126.2958 in F0