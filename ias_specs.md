# IAS Computer Architecture Specification

## 1. OVERVIEW

- **Machine Name**: IAS (Institute for Advanced Study) Computer
- **Alternative Name**: von Neumann Machine
- **Operational Period**: June 10, 1952 – July 15, 1958
- **Developers**: John von Neumann, Herman Goldstine, Julian Bigelow, and team
- **Location**: Princeton, New Jersey
- **Architecture Type**: Von Neumann Architecture (Stored-Program Computer)

---

## 2. MEMORY

- **Total Capacity**: 4,096 words (163,840 bits)
- **Word Width**: 40 bits
- **Addressable Locations**: 0–4,095
- **Data Format**: Binary, two's complement for negative numbers
- **Access Type**: Random-access
- **Memory Organization**: 
  - Each word contains either: 2 instructions OR 1 data value (40-bit)
  - Memory address bus width: 12 bits (for 4,096 locations)
  - Data bus width: 40 bits
  - Storage mechanism: 40 Selectron tubes (CRT-based), each storing 4,096 40-bit words in parallel
- **Operations**:
  - **Read**: Address (via Memory Address Register) → Data retrieval (via Selectron Register)
  - **Write**: Address (via Memory Address Register) + Data (via Selectron Register) → Storage to memory

## 3. REGISTERS

### Primary General-Purpose Registers (Programmer-Visible)

| Register | Name | Bit Width | Purpose |
|----------|------|-----------|---------|
| **AC** | Accumulator | 40 bits | Primary arithmetic register; holds operands and results |
| **AR** | Arithmetic Register | 40 bits | Holds multiplier during multiplication; holds quotient during division |

### Control & Internal Registers (Not Directly Accessible)

| Register | Name | Bit Width | Purpose |
|----------|------|-----------|---------|
| **CC** | Control Counter | 12 bits | Holds address of next instruction pair (Program Counter) |
| **CR** | Control Register | 20 bits | Holds current instruction being executed |
| **FR** | Function Table Register | 8 bits | Holds decoded opcode from current instruction |
| **MAR** | Memory Address Register | 12 bits | Holds memory address for read/write operations (0-4095) |
| **SR** | Selectron Register | 40 bits | Holds data to/from memory (interface to Selectron memory) |

### Register Operations

- **AC**: Arithmetic operations (ADD, SUB, MUL, DIV), logical operations (AND, OR, XOR), shifts, load/store
- **AR**: Stores multiplier during multiplication; stores quotient during division
- **CC**: Automatically incremented after each instruction pair; modified by jump/branch instructions
- **CR**: Loaded from memory via SR; decoded by FR
- **FR**: Decoded opcode enables appropriate control circuits (8-bit opcode allows up to 256 operations)
- **MAR**: Loaded from CC or instruction address field to select memory location
- **SR**: Interface between AC/AR and main memory; transfers data between memory and registers

---

## 4. CONTROL UNIT

### Purpose

- Manages instruction fetch-decode-execute cycle
- Generates control signals for all machine operations
- Coordinates timing and sequencing of operations

### Key Characteristics

- **Timing**: Asynchronous (no central clock; operation completion driven)
- **Operation Times**:
  - Addition: 62 microseconds
  - Multiplication: 713 microseconds
  - Division: ~700–800 microseconds
- **Instruction Cycle**: Fetch → Decode → Execute → Store Result
- **Instruction Fetch**: CC → MAR → Memory read → CR (load into control register)

---

## 5. ARITHMETIC & LOGIC UNIT (ALU)

### Core Arithmetic Operations

The ALU supports the following operations used by the instruction set:

| Instruction | Opcode | Operation | Result |
|-------------|--------|-----------|--------|
| **S(x)->Ac+** | 1 | Load and clear | AC ← Memory[x] |
| **S(x)->Ac-** | 2 | Load negation | AC ← −Memory[x] |
| **S(x)->AcM** | 3 | Load absolute value | AC ← \|Memory[x]\| |
| **S(x)->Ac-M** | 4 | Load negative absolute | AC ← −\|Memory[x]\| |
| **S(x)->Ah+** | 5 | Addition | AC ← AC + Memory[x] |
| **S(x)->Ah-** | 6 | Subtraction | AC ← AC − Memory[x] |
| **S(x)->AhM** | 7 | Add absolute value | AC ← AC + \|Memory[x]\| |
| **S(x)->Ah-M** | 8 | Subtract absolute value | AC ← AC − \|Memory[x]\| |

### Multiplication & Division

| Instruction | Opcode | Operation | Result |
|-------------|--------|-----------|--------|
| **S(x)×R->A** | 11 | Multiply | AC ← (Memory[x] × AR) high 40 bits; AR ← low 40 bits |
| **A÷S(x)->R** | 12 | Divide | AR ← quotient; AC ← remainder |

The multiplication produces an 80-bit result split between AC (high 40 bits) and AR (low 40 bits). Division performs non-restoring division without sign restrictions.

### Shift Operations

| Instruction | Opcode | Operation | Effect |
|-------------|--------|-----------|--------|
| **L** | 20 | Shift left | AC ← AC × 2 (arithmetic left shift by 1) |
| **R** | 21 | Shift right | AC ← AC ÷ 2 (arithmetic right shift by 1) |

Shifts are used for scaling and are often employed with multiplication/division or to implement floating-point operations through programming.

### Sign Testing

The IAS ALU provides no explicit status flags. Instead, the sign of AC is tested by conditional jump instructions:
- **Cc->S(x)**: Jump if AC ≥ 0 (sign bit is 0)
- **Cc'->S(x)**: Jump if AC ≥ 0 (right instruction variant)

Note: The IAS machine does not have built-in logical operations (AND, OR, XOR) or bitwise complement operations.

---

## 6. INSTRUCTION SET ARCHITECTURE

### Instruction Word Format

Each 20-bit instruction contains:

- **Opcode Field**: 8 bits (bits 19–12) → 256 possible opcodes
- **Index Field**: 2 bits (bits 11–10) → Register selection (if applicable)
- **Address Field**: 10 bits (bits 9–0) → Memory address (0–1023)

**Bit Layout**:

| Bit Range | Field | Width | Purpose |
|-----------|-------|-------|----------|
| 19–12 | OPCODE | 8 bits | Operation code (selects instruction type) |
| 11–0 | OPERAND | 12 bits | Memory address or operand reference (0–4095) |

---

## 7. INSTRUCTION SET

| Opcode | Mnemonic | Operation | Notes |
|--------|----------|-----------|-------|
| 1 | S(x)->Ac+ | AC ← S(x) | Clear AC and load memory word at address x |
| 2 | S(x)->Ac- | AC ← −S(x) | Clear AC and load negation of S(x) |
| 3 | S(x)->AcM | AC ← \|S(x)\| | Clear AC and load absolute value of S(x) |
| 4 | S(x)->Ac-M | AC ← −\|S(x)\| | Clear AC and subtract absolute value |
| 5 | S(x)->Ah+ | AC ← AC + S(x) | Add memory word to AC |
| 6 | S(x)->Ah- | AC ← AC − S(x) | Subtract memory word from AC |
| 7 | S(x)->AhM | AC ← AC + \|S(x)\| | Add absolute value to AC |
| 8 | S(x)->Ah-M | AC ← AC − \|S(x)\| | Subtract absolute value from AC |
| 9 | S(x)->R | AR ← S(x) | Load memory word into Arithmetic Register |
| 10 | R->A | AC ← AR | Transfer AR to AC |
| 11 | S(x)×R->A | AC ← S(x) × AR (high 40 bits); AR ← (low 40 bits) | Multiply with result split between AC and AR |
| 12 | A÷S(x)->R | AR ← quotient; AC ← remainder | Divide AC by S(x) |
| 13 | Cu->S(x) | CC ← x (left instruction) | Unconditional jump to left instruction at x |
| 14 | Cu'->S(x) | CC ← x (right instruction) | Unconditional jump to right instruction at x |
| 15 | Cc->S(x) | If AC ≥ 0: CC ← x (left) | Conditional jump on sign |
| 16 | Cc'->S(x) | If AC ≥ 0: CC ← x (right) | Conditional jump on sign |
| 17 | At->S(x) | S(x) ← AC | Store AC to memory |
| 18 | Ap->S(x) | S(x) left addr ← AC left 12 bits | Partial substitution (left instruction) |
| 19 | Ap'->S(x) | S(x) right addr ← AC left 12 bits | Partial substitution (right instruction) |
| 20 | L | AC ← AC × 2 | Shift AC left 1 bit |
| 21 | R | AC ← AC ÷ 2 | Shift AC right 1 bit (arithmetic) |
| 0 | halt | Stop execution | Machine halt |

### Instruction Categories by Function

The 21 standard operations are organized by function:

- **Arithmetic** (Opcodes 1-8): ADD, SUB, ABS, conditional variants
- **Register Transfer** (Opcodes 9-10): Load/store between AC, AR, and memory
- **Multiplication** (Opcode 11): Multiply AC by AR with 80-bit result
- **Division** (Opcode 12): Divide AC by memory operand
- **Control Flow** (Opcodes 13-16): Unconditional and conditional jumps
- **Partial Substitution** (Opcodes 18-19): Modify instruction addresses (indexing)
- **Shifts** (Opcodes 20-21): Arithmetic shifts for AC
- **Halt** (Opcode 0): Stop machine execution

---

## 8. ADDRESSING MODES

### Direct Addressing
- **Description**: Instruction operand specifies memory address directly
- **Format**: 12-bit address field references 0–4095 directly
- **Example**: `S(x)->Ac+` with x=256 loads memory location 256 into AC
- **Range**: 0–4095 (all memory addresses)

### Self-Modifying Code (Indexed Addressing)
- **Description**: Use partial substitution (Ap->S(x), Ap'->S(x)) to modify stored instruction addresses
- **Purpose**: Implement loops and array indexing without dedicated index registers
- **Method**: 
  1. Store instruction with placeholder address at location y
  2. Use Ap->S(y) to replace address field with computed value from AC
  3. Execute modified instruction
  4. Repeat or modify address again for next iteration

---

## 9. INSTRUCTION EXECUTION DETAILS

### Primary Data Paths

1. **Memory → AC (Load Path)**
   - Sequence: Memory[MAR] → SR → AC
   - Used for: S(x)->Ac+, S(x)->Ac-, S(x)->AcM, S(x)->Ac-M instructions

2. **AC → Memory (Store Path)**
   - Sequence: AC → SR → Memory[MAR]
   - Used for: At->S(x) instruction

3. **AC ↔ ALU (Arithmetic Path)**
   - Sequence: AC + Operand → ALU → Result → AC
   - Used for: S(x)->Ac+, S(x)->Ah+, and other arithmetic operations

4. **AR ↔ AC (Multiply/Divide Path)**
   - Sequence: AC × AR → ALU → (AC: high 40 bits; AR: low 40 bits)
   - Used for: S(x)×R->A (multiplication), A÷S(x)->R (division)

5. **Input/Output Path**
   - Sequence: Input Device ↔ AC ↔ Output Device
   - Used for: INPUT, OUTPUT instructions

## References & Notes

This specification is based on the historical IAS machine documentation (see [link](https://deepblue.lib.umich.edu/items/60414b75-b85e-47e0-9d06-4adee8e68549)) as well as an existing emulator (see [link](https://www.cs.colby.edu/djskrien/IASSim/)) and represents the actual hardware and instruction set of the computer as built at Princeton's Institute for Advanced Study. The architecture exemplifies fundamental computer design principles and remains relevant for understanding modern CPU architectures.
