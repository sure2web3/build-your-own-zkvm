# Overview of the `nexus-vm-prover` crate in `nexus-zkvm`

The `nexus-vm-prover` crate is the core proving engine of the Nexus zkVM project. It is responsible for generating zero-knowledge proofs that attest to the correct execution of programs within the Nexus virtual machine.

## Key responsibilities and features

| Responsibility | Description |
| --- | --- |
| **Trace Generation** | It constructs execution traces of the VM, capturing the state of registers, memory, and other components at each step. |
| **AIR Constraint System** | The crate defines and applies Algebraic Intermediate Representation (AIR) constraints to the trace, ensuring that only valid program executions can produce a satisfiable trace. |
| **Proof Construction** | Using the trace and AIR constraints, it generates cryptographic proofs (such as STARKs) that can be verified by anyone, without revealing private inputs or the full trace. | 
| **Parallelism and Performance** | The crate leverages libraries like rayon for parallel computation, making proof generation efficient even for large programs. | 
| **Integration** | It depends on other core modules such as nexus-vm (the virtual machine implementation), nexus-common (shared utilities), and nexus-vm-prover-macros (procedural macros for code generation). | 

**Summary**

`nexus-vm-prover` is the heart of the proof system in Nexus zkVM, turning VM execution into verifiable, privacy-preserving cryptographic proofs.

## Directory Structure of `prover`
The `prover` folder in `nexus-zkvm` contains the main implementation of the zero-knowledge proof system for the Nexus virtual machine. It includes code for generating execution traces, applying AIR (Algebraic Intermediate Representation) constraints, and constructing cryptographic proofs to verify correct program execution. This folder also manages performance optimizations, integrates with the VM core, and provides procedural macros to automate boilerplate code for trace columns and constraints. Overall, it is the core component responsible for turning VM execution into verifiable zero-knowledge proofs.

### Directory tree

```shell
prover  
├── Cargo.toml                  # Crate manifest and dependencies (the main manifest file for the crate, specifying dependencies and metadata)  
├── macros  
│   ├── Cargo.toml              # Manifest for macros subcrate (manifest for the procedural macros subcrate, which is used for code generation)  
│   └── src  
│       ├── column_enum.rs      # Macro for generating trace column enums (Rust procedural macro that generates enums for trace columns, reducing repetitive code and ensuring consistency)  
│       └── lib.rs              # Entry point for macros crate (the main library file for the macros subcrate, where macro exports are defined)  
├── README.md                   # Documentation for the prover crate (provides an overview, usage, and design notes for the prover crate)  
└── src  
    ├── chips  
    │   ├── cpu.rs              # CPU chip constraints and logic (defines the algebraic constraints and logic for the CPU chip, which models the core processor state transitions in the zkVM)  
    │   ├── custom.rs           # Custom chip implementations (contains implementations for custom chips, allowing extension of the VM with new features or constraints)  
    │   ├── decoding  
    │   │   ├── mod.rs          # Module for instruction decoding (the main module for instruction decoding, organizing all decoding logic for different instruction types)  
    │   │   ├── type_b.rs       # Decoding for type B instructions (handles decoding for RISC-V type B instructions, which are conditional branch instructions such as BEQ, BNE, BLT, BGE, BLTU, BGEU; in nexus-zkvm, these control program flow based on register comparisons)  
    │   │   ├── type_i.rs       # Decoding for type I instructions (handles decoding for RISC-V type I instructions, which include immediate arithmetic, loads, and some system instructions; in nexus-zkvm, these are used for operations like ADDI, LW, JALR, etc.)  
    │   │   ├── type_j.rs       # Decoding for type J instructions (handles decoding for RISC-V type J instructions, which are jump instructions such as JAL; in nexus-zkvm, these enable unconditional jumps with a 20-bit immediate)  
    │   │   ├── type_r.rs       # Decoding for type R instructions (handles decoding for RISC-V type R instructions, which are register-register arithmetic and logic instructions like ADD, SUB, SLL, etc.; in nexus-zkvm, these perform operations between two registers)  
    │   │   ├── type_s.rs       # Decoding for type S instructions (handles decoding for RISC-V type S instructions, which are store instructions such as SW, SB, SH; in nexus-zkvm, these write register values to memory)  
    │   │   ├── type_sys.rs     # Decoding for system instructions (handles decoding for system instructions, such as ECALL and EBREAK; in nexus-zkvm, these are used for system calls and environment interaction)  
    │   │   └── type_u.rs       # Decoding for type U instructions (handles decoding for RISC-V type U instructions, which are upper immediate instructions like LUI and AUIPC; in nexus-zkvm, these load a 20-bit immediate into the upper bits of a register or add it to the PC)  
    │   ├── instructions  
    │   │   ├── add.rs          # ADD instruction constraints (algebraic constraints for the ADD instruction, which performs register addition in the VM)  
    │   │   ├── auipc.rs        # AUIPC instruction constraints (constraints for AUIPC, which stands for Add Upper Immediate to PC; in RISC-V and nexus-zkvm, this adds a 20-bit immediate to the program counter and writes the result to a register)  
    │   │   ├── beq.rs          # BEQ instruction constraints (constraints for BEQ, Branch if Equal; checks if two registers are equal and conditionally branches)  
    │   │   ├── bge.rs          # BGE instruction constraints (constraints for BGE, Branch if Greater or Equal; checks if one register is greater than or equal to another and branches accordingly)  
    │   │   ├── bgeu.rs         # BGEU instruction constraints (constraints for BGEU, Branch if Greater or Equal Unsigned; similar to BGE but for unsigned values)  
    │   │   ├── bit_op.rs       # Bitwise operation constraints (constraints for bitwise operations such as AND, OR, XOR, SLL, SRL, SRA; ensures correct bitwise logic in the VM)  
    │   │   ├── blt.rs          # BLT instruction constraints (constraints for BLT, Branch if Less Than; checks if one register is less than another and branches)  
    │   │   ├── bltu.rs         # BLTU instruction constraints (constraints for BLTU, Branch if Less Than Unsigned; unsigned comparison for conditional branching)  
    │   │   ├── bne.rs          # BNE instruction constraints (constraints for BNE, Branch if Not Equal; branches if two registers are not equal)  
    │   │   ├── jal.rs          # JAL instruction constraints (constraints for JAL, Jump and Link; jumps to a target address and saves the return address in a register)  
    │   │   ├── jalr.rs         # JALR instruction constraints (constraints for JALR, Jump and Link Register; indirect jump using a register and saves the return address)  
    │   │   ├── load_store.rs   # Load/store instruction constraints (constraints for memory load and store instructions, such as LW, SW, LB, SB; ensures correct memory access and data movement)  
    │   │   ├── lui.rs          # LUI instruction constraints (constraints for LUI, Load Upper Immediate; loads a 20-bit immediate into the upper bits of a register)  
    │   │   ├── mod.rs          # Module for instruction constraints (main module for organizing all instruction constraint files)  
    │   │   ├── sll.rs          # SLL instruction constraints (constraints for SLL, Shift Left Logical; shifts a register value left by a specified amount)  
    │   │   ├── slt.rs          # SLT instruction constraints (constraints for SLT, Set Less Than; sets a register to 1 if one register is less than another, else 0)  
    │   │   ├── sltu.rs         # SLTU instruction constraints (constraints for SLTU, Set Less Than Unsigned; unsigned version of SLT)  
    │   │   ├── sra.rs          # SRA instruction constraints (constraints for SRA, Shift Right Arithmetic; arithmetic right shift, preserving sign)  
    │   │   ├── srl.rs          # SRL instruction constraints (constraints for SRL, Shift Right Logical; logical right shift, filling with zeros)  
    │   │   ├── sub.rs          # SUB instruction constraints (constraints for SUB, subtraction between two registers)  
    │   │   └── syscall.rs      # System call instruction constraints (constraints for system call instructions, such as ECALL; enables interaction with the host or environment)  
    │   ├── memory_check  
    │   │   ├── mod.rs          # Module for memory checks (main module for memory consistency and integrity checks)  
    │   │   ├── program_mem_check.rs   # Program memory consistency checks (checks to ensure program memory is accessed and modified correctly)  
    │   │   ├── register_mem_check.rs  # Register memory consistency checks (checks to ensure register and memory states are consistent)  
    │   │   └── timestamp.rs    # Timestamp checks for memory (logic for tracking the order and timing of memory operations)  
    │   ├── mod.rs              # Main module for chips (aggregates all chip modules and logic)  
    │   ├── range_check  
    │   │   ├── mod.rs          # Module for range checks (main module for range constraint logic)  
    │   │   ├── range_bool.rs   # Boolean range checks (ensures a value is either 0 or 1, used for boolean constraints)  
    │   │   ├── range128.rs     # 128-bit range checks (ensures a value fits within 128 bits)  
    │   │   ├── range16.rs      # 16-bit range checks (ensures a value fits within 16 bits)  
    │   │   ├── range256.rs     # 256-bit range checks (ensures a value fits within 256 bits)  
    │   │   ├── range32.rs      # 32-bit range checks (ensures a value fits within 32 bits)  
    │   │   └── range8.rs       # 8-bit range checks (ensures a value fits within 8 bits)  
    │   └── utils.rs            # Utility functions for chips (helper functions for chip logic, constraint evaluation, and code reuse)  
    ├── column.rs               # Trace column definitions and utilities (defines trace columns, which represent VM state variables in the execution trace, and provides utilities for accessing and manipulating them)  
    ├── components  
    │   ├── lookups.rs          # Lookup table logic for constraints (logic for lookup tables used in constraints, such as memory or register value tables)  
    │   └── mod.rs              # Module for components (main module for reusable components shared across chips and constraints)  
    ├── extensions  
    │   ├── bit_op.rs           # Bitwise operation extensions (extensions for advanced bitwise operations, supporting cryptographic and custom logic)  
    │   ├── config  
    │   │   └── mod.rs          # Configuration for extensions (configuration logic for extension modules, such as parameter settings)  
    │   ├── final_reg.rs        # Final register state extensions (logic for tracking and constraining the final state of registers at program end)  
    │   ├── keccak  
    │   │   ├── bit_rotate  
    │   │   │   └── mod.rs      # Bit rotation logic for Keccak (logic for bit rotation operations used in Keccak hash computations)  
    │   │   ├── bitwise_table  
    │   │   │   ├── constraints.rs      # Constraints for Keccak bitwise table (algebraic constraints for the Keccak bitwise operation table)  
    │   │   │   ├── mod.rs              # Module for bitwise table (main module for Keccak bitwise table logic)  
    │   │   │   ├── preprocessed_columns.rs # Preprocessed columns for bitwise table (columns precomputed for efficient bitwise operations in Keccak)  
    │   │   │   └── trace.rs            # Trace logic for bitwise table (logic for constructing and managing the bitwise operation trace for Keccak)  
    │   │   ├── memory_check  
    │   │   │   ├── constraints.rs      # Constraints for Keccak memory check (constraints for memory consistency in Keccak operations)  
    │   │   │   ├── mod.rs              # Module for Keccak memory check (main module for Keccak memory check logic)  
    │   │   │   └── trace.rs            # Trace logic for Keccak memory check (logic for tracking memory operations in Keccak)  
    │   │   ├── mod.rs          # Main module for Keccak extensions (aggregates all Keccak-related extension logic)  
    │   │   └── round  
    │   │       ├── component.rs        # Keccak round component logic (logic for each round of the Keccak hash function, including state updates)  
    │   │       ├── constants.rs        # Constants for Keccak rounds (constants used in Keccak round computations, such as round constants)  
    │   │       ├── constraints.rs      # Constraints for Keccak rounds (algebraic constraints for each Keccak round)  
    │   │       ├── eval.rs             # Evaluation logic for Keccak rounds (logic for evaluating Keccak round computations)  
    │   │       ├── interaction_trace.rs # Interaction trace for Keccak rounds (tracks data flow and interactions between Keccak rounds)  
    │   │       ├── mod.rs              # Module for Keccak round (main module for Keccak round logic)  
    │   │       └── trace.rs            # Trace logic for Keccak rounds (logic for constructing and managing the trace for Keccak rounds)  
    │   ├── mod.rs              # Main module for extensions (aggregates all extension modules and logic)  
    │   ├── multiplicity.rs     # Multiplicity extension logic (logic for multiplicity constraints, used for enforcing multiple occurrences or relationships)  
    │   ├── multiplicity8.rs    # 8-bit multiplicity extension logic (logic for 8-bit multiplicity constraints, a specialized form of multiplicity checks)  
    │   ├── ram_init_final.rs   # RAM initialization/finalization logic (logic for initializing and finalizing RAM state, ensuring memory is set up and cleaned up correctly)  
    │   └── trace.rs            # Trace logic for extensions (logic for constructing and managing traces for extension features)  
    ├── lib.rs                  # Crate entry point and main logic (the main entry point for the prover crate, organizing modules and exposing the public API)  
    ├── machine.rs              # VM machine state and orchestration (manages the overall VM state, coordinates chips, and controls the proof process)  
    ├── test_utils.rs           # Utilities for testing the prover (helper functions, mocks, and utilities for unit and integration testing of the prover)  
    ├── trace  
    │   ├── eval.rs             # Trace evaluation logic (logic for evaluating the correctness of execution traces)  
    │   ├── mod.rs              # Module for trace logic (main module for organizing all trace-related logic)  
    │   ├── preprocessed.rs     # Preprocessing for traces (logic for preprocessing trace data for efficiency and correctness)  
    │   ├── program_trace.rs    # Program-specific trace logic (logic for handling traces specific to individual programs)  
    │   ├── program.rs          # Program representation for tracing (defines how programs are represented within the trace system)  
    │   ├── regs.rs             # Register state tracing (logic for tracking register state changes throughout execution)  
    │   ├── sidenote  
    │   │   ├── keccak.rs       # Keccak-related sidenote logic (auxiliary logic for Keccak hash operations)  
    │   │   └── mod.rs          # Module for sidenote (main module for auxiliary and sidenote logic)  
    │   ├── trace_builder.rs    # Builder for constructing traces (logic for building execution traces from VM execution)  
    │   ├── utils_external.rs   # External utilities for traces (helper functions for trace processing, used externally)  
    │   └── utils.rs            # Internal utilities for traces (helper functions for internal trace manipulation and processing)  
    ├── traits.rs               # Common prover traits and interfaces (defines common traits and interfaces for chips, traces, and components, ensuring modularity and extensibility)  
    └── virtual_column.rs       # Logic for virtual (computed) trace columns (logic for virtual trace columns, which are computed from other columns and used for expressing complex constraints)  
```
### Description of each component

Below is a detailed overview of its structure, including the purpose and principles behind each component.

| Name/Path         | Type      | Description & Principle                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|-------------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| chips/            | Directory | Contains modular "chips" implementations, each representing a specific part of the VM (such as CPU, instruction decoding, memory checks, and range checks). Each chip encodes the algebraic constraints for its component, enabling the prover to enforce correct behavior for that part of the VM. Chips are composed to form the full constraint system, supporting modularity, extensibility, and efficient proof generation for complex instruction sets.                        |
| components/       | Directory | Provides reusable components and lookup tables that are shared across chips and constraint systems. These components abstract common logic (like lookups for memory or registers) and help organize the constraint system, making it easier to maintain and extend. Lookup tables are crucial for efficient constraint evaluation and for supporting advanced features like custom instructions or cryptographic primitives.                                         |
| extensions/       | Directory | Implements advanced features and cryptographic extensions, such as bitwise operations, Keccak hashing, and multiplicity checks. Extensions allow the prover to support more complex computations and cryptographic protocols by adding new constraints and trace columns. They are designed to be composable and configurable, so the proof system can be tailored to different use cases or security requirements.                                             |
| trace/            | Directory | Responsible for constructing, evaluating, and managing execution traces. The trace records the state of the VM at each step (registers, memory, etc.), forming the main witness for the zero-knowledge proof. This directory includes logic for building traces, preprocessing them, evaluating constraints, and providing utilities for trace manipulation. The trace structure is central to the AIR (Algebraic Intermediate Representation) used in proof generation.         |
| macros/           | Directory | Contains procedural macros that automate code generation for trace columns, enums, and constraint definitions. These macros reduce boilerplate, enforce consistency, and help prevent errors in the definition of trace-related structures. By generating repetitive code automatically, macros make the prover codebase more maintainable and less error-prone, especially as the number of instructions and constraints grows.                                    |
| column.rs         | File      | Defines the structures and utilities for trace columns, which represent individual state variables (like registers or memory cells) in the execution trace. This file maps VM state to trace columns and provides methods for accessing, indexing, and manipulating columns. It is essential for organizing the trace data and applying constraints to the correct parts of the trace during proof generation.                                               |
| lib.rs            | File      | The main entry point for the prover crate. It sets up the module structure, imports key components, and exposes the primary API for proof generation and verification. This file orchestrates the interaction between chips, components, extensions, and trace logic, serving as the glue that binds the prover system together.                                                                                                                        |
| machine.rs        | File      | Manages the overall VM machine state and coordinates the interaction between chips, trace construction, and proof logic. It defines the main proving and verification routines, handles the composition of chips and extensions, and manages the flow of data through the proof system. This file is responsible for turning a VM execution trace into a zero-knowledge proof and for verifying such proofs against public inputs.                        |
| test_utils.rs     | File      | Provides helper functions, mock data, and utilities for testing the prover’s logic and constraints. This file enables robust unit and integration testing, ensuring that the prover produces correct proofs and that constraints are enforced as expected. It is essential for maintaining code quality and reliability as the system evolves.                                                                                                 |
| traits.rs         | File      | Defines common traits and interfaces for chips, trace columns, components, and other prover modules. These traits ensure modularity, extensibility, and type safety, allowing different parts of the prover to interact in a generic and composable way. They also enable easy integration of new chips or extensions without modifying core logic.                                                                                      |
| virtual_column.rs | File      | Implements logic for virtual (computed) trace columns, which are not directly stored in the trace but are derived from other columns. Virtual columns are used to express complex constraints or relationships between state variables, making the constraint system more expressive and efficient. This file provides the infrastructure for defining, computing, and using virtual columns in the proof system.                                 |

# Terminology
## trace
In Nexus-zkVM, a trace is a structured table (or matrix) that records the step-by-step execution of the virtual machine (VM) during program execution. Each row in the trace corresponds to a single VM cycle (or instruction), and each column represents a specific register, memory cell, or internal state variable of the VM.

**Purpose of the trace** 

The trace is the primary witness data for the zero-knowledge proof system. It contains all the information needed to verify that the VM executed the program correctly, without revealing the program’s private inputs.
The trace is used as input to the Algebraic Intermediate Representation (AIR) and is committed to during the proof process.
Constraints are applied to the trace to ensure that every step follows the VM’s rules and the program logic.

**Summary**

A trace in Nexus-zkVM is a detailed log of the VM’s state at each step, structured for efficient and secure zero-knowledge proof generation and verification.

## Algebraic Intermediate Representation (AIR)

AIR is a mathematical framework used in zero-knowledge proof systems, especially STARKs (Scalable Transparent ARguments of Knowledge), to describe and verify the correct execution of computations.

In the context of Nexus-zkVM, AIR serves the following purposes:

- **Trace Constraints**: AIR defines a set of algebraic constraints (usually polynomial equations) that the execution trace of the virtual machine must satisfy. The trace records the state of the VM at each step (such as registers, memory, and program counter).
- **Soundness**: By expressing the VM’s transition rules and program logic as algebraic constraints, AIR ensures that only valid traces (i.e., traces corresponding to correct program execution) can satisfy all the constraints.
- **Proof Generation**: The prover commits to the trace and demonstrates, using the AIR constraints, that the trace is valid—without revealing private data or the full trace itself.
- **Abstraction**: AIR abstracts away low-level implementation details and provides a uniform, algebraic way to describe computation, making it easier to reason about and verify.

**Summary**

AIR in Nexus-zkVM is a set of algebraic rules that the VM’s execution trace must satisfy, enabling efficient and secure zero-knowledge proofs of correct computation.