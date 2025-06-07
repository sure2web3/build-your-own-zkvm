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
├── Cargo.toml                  # Crate manifest and dependencies  
├── macros  
│   ├── Cargo.toml              # Manifest for macros subcrate  
│   └── src  
│       ├── column_enum.rs      # Macro for generating trace column enums  
│       └── lib.rs              # Entry point for macros crate  
├── README.md                   # Documentation for the prover crate  
└── src  
    ├── chips  
    │   ├── cpu.rs              # CPU chip constraints and logic  
    │   ├── custom.rs           # Custom chip implementations  
    │   ├── decoding  
    │   │   ├── mod.rs          # Module for instruction decoding  
    │   │   ├── type_b.rs       # Decoding for type B instructions  
    │   │   ├── type_i.rs       # Decoding for type I instructions  
    │   │   ├── type_j.rs       # Decoding for type J instructions  
    │   │   ├── type_r.rs       # Decoding for type R instructions  
    │   │   ├── type_s.rs       # Decoding for type S instructions  
    │   │   ├── type_sys.rs     # Decoding for system instructions  
    │   │   └── type_u.rs       # Decoding for type U instructions  
    │   ├── instructions  
    │   │   ├── add.rs          # ADD instruction constraints  
    │   │   ├── auipc.rs        # AUIPC instruction constraints  
    │   │   ├── beq.rs          # BEQ instruction constraints  
    │   │   ├── bge.rs          # BGE instruction constraints  
    │   │   ├── bgeu.rs         # BGEU instruction constraints  
    │   │   ├── bit_op.rs       # Bitwise operation constraints  
    │   │   ├── blt.rs          # BLT instruction constraints  
    │   │   ├── bltu.rs         # BLTU instruction constraints  
    │   │   ├── bne.rs          # BNE instruction constraints  
    │   │   ├── jal.rs          # JAL instruction constraints  
    │   │   ├── jalr.rs         # JALR instruction constraints  
    │   │   ├── load_store.rs   # Load/store instruction constraints  
    │   │   ├── lui.rs          # LUI instruction constraints  
    │   │   ├── mod.rs          # Module for instruction constraints  
    │   │   ├── sll.rs          # SLL instruction constraints  
    │   │   ├── slt.rs          # SLT instruction constraints  
    │   │   ├── sltu.rs         # SLTU instruction constraints  
    │   │   ├── sra.rs          # SRA instruction constraints  
    │   │   ├── srl.rs          # SRL instruction constraints  
    │   │   ├── sub.rs          # SUB instruction constraints  
    │   │   └── syscall.rs      # System call instruction constraints  
    │   ├── memory_check  
    │   │   ├── mod.rs          # Module for memory checks  
    │   │   ├── program_mem_check.rs   # Program memory consistency checks  
    │   │   ├── register_mem_check.rs  # Register memory consistency checks  
    │   │   └── timestamp.rs    # Timestamp checks for memory  
    │   ├── mod.rs              # Main module for chips  
    │   ├── range_check  
    │   │   ├── mod.rs          # Module for range checks  
    │   │   ├── range_bool.rs   # Boolean range checks  
    │   │   ├── range128.rs     # 128-bit range checks  
    │   │   ├── range16.rs      # 16-bit range checks  
    │   │   ├── range256.rs     # 256-bit range checks  
    │   │   ├── range32.rs      # 32-bit range checks  
    │   │   └── range8.rs       # 8-bit range checks  
    │   └── utils.rs            # Utility functions for chips  
    ├── column.rs               # Trace column definitions and utilities  
    ├── components  
    │   ├── lookups.rs          # Lookup table logic for constraints  
    │   └── mod.rs              # Module for components  
    ├── extensions  
    │   ├── bit_op.rs           # Bitwise operation extensions  
    │   ├── config  
    │   │   └── mod.rs          # Configuration for extensions  
    │   ├── final_reg.rs        # Final register state extensions  
    │   ├── keccak  
    │   │   ├── bit_rotate  
    │   │   │   └── mod.rs      # Bit rotation logic for Keccak  
    │   │   ├── bitwise_table  
    │   │   │   ├── constraints.rs      # Constraints for Keccak bitwise table  
    │   │   │   ├── mod.rs              # Module for bitwise table  
    │   │   │   ├── preprocessed_columns.rs # Preprocessed columns for bitwise table  
    │   │   │   └── trace.rs            # Trace logic for bitwise table  
    │   │   ├── memory_check  
    │   │   │   ├── constraints.rs      # Constraints for Keccak memory check  
    │   │   │   ├── mod.rs              # Module for Keccak memory check  
    │   │   │   └── trace.rs            # Trace logic for Keccak memory check  
    │   │   ├── mod.rs          # Main module for Keccak extensions  
    │   │   └── round  
    │   │       ├── component.rs        # Keccak round component logic  
    │   │       ├── constants.rs        # Constants for Keccak rounds  
    │   │       ├── constraints.rs      # Constraints for Keccak rounds  
    │   │       ├── eval.rs             # Evaluation logic for Keccak rounds  
    │   │       ├── interaction_trace.rs # Interaction trace for Keccak rounds  
    │   │       ├── mod.rs              # Module for Keccak round  
    │   │       └── trace.rs            # Trace logic for Keccak rounds  
    │   ├── mod.rs              # Main module for extensions  
    │   ├── multiplicity.rs     # Multiplicity extension logic  
    │   ├── multiplicity8.rs    # 8-bit multiplicity extension logic  
    │   ├── ram_init_final.rs   # RAM initialization/finalization logic  
    │   └── trace.rs            # Trace logic for extensions  
    ├── lib.rs                  # Crate entry point and main logic  
    ├── machine.rs              # VM machine state and orchestration  
    ├── test_utils.rs           # Utilities for testing the prover  
    ├── trace  
    │   ├── eval.rs             # Trace evaluation logic  
    │   ├── mod.rs              # Module for trace logic  
    │   ├── preprocessed.rs     # Preprocessing for traces  
    │   ├── program_trace.rs    # Program-specific trace logic  
    │   ├── program.rs          # Program representation for tracing  
    │   ├── regs.rs             # Register state tracing  
    │   ├── sidenote  
    │   │   ├── keccak.rs       # Keccak-related sidenote logic  
    │   │   └── mod.rs          # Module for sidenote  
    │   ├── trace_builder.rs    # Builder for constructing traces  
    │   ├── utils_external.rs   # External utilities for traces  
    │   └── utils.rs            # Internal utilities for traces  
    ├── traits.rs               # Common prover traits and interfaces  
    └── virtual_column.rs       # Logic for virtual (computed) trace columns  
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