# Asynchronous FIFO Design

This repository contains Verilog code for an asynchronous FIFO (First-In, First-Out) buffer where read and write operations are controlled by independent clock domains.

---

## Table of Contents

- [Introduction](#introduction)
- [Block Diagram](#block-diagram)
- [Design Space Exploration and Strategies](#design-space-exploration-and-strategies)
- [Full, Empty and Wrapping Conditions](#full-empty-and-wrapping-conditions)
  - [Empty Condition](#empty-condition)
  - [Full Condition](#full-condition)
  - [Wrapping Around Condition](#wrapping-around-condition)
- [Gray Code Counter](#gray-code-counter)
- [Signal Definitions](#signal-definitions)
- [Design Modules](#design-modules)
- [Testbench Implementation](#testbench-implementation)
- [Waveforms](#waveforms)

---

## Introduction

FIFO (First-In, First-Out) is a buffer where the first data element written is the first one read. An asynchronous FIFO has separate write and read clock domains, so writing and reading are driven by independent clocks that are not synchronized.

The implementation in this repository provides a well-structured asynchronous FIFO with pointer synchronization, dual-port memory, and status flags (full/empty).

---

## Block Diagram

Thin lines represent single-bit signals while thick lines represent multi-bit signals.

![Block diagram 1](https://github.com/user-attachments/assets/ea434eff-1b29-4554-bc36-2276b18e9e56)

![Block diagram 2](https://github.com/user-attachments/assets/74d71bfa-54b8-4d00-99dd-3721617b8e8e)

---

## Design Space Exploration and Strategies

The design separates concerns into distinct modules for clarity and for easier verification. Key strategies include:

- Using dual-port RAM for independent read/write access.
- Using Gray-coded pointers to reduce synchronization hazards.
- Synchronizing pointers across clock domains with two-flop synchronizers.
- Using an extra MSB on pointers to distinguish full vs empty when binary pointer values are equal.

---

## Full, Empty and Wrapping Conditions

### Empty Condition
The FIFO is empty when the read and write pointers are equal. This happens on reset (both pointers at zero) or when the read pointer catches up to the write pointer after reads.

### Full Condition
The FIFO is full when the write pointer wraps around and catches up with the read pointer such that the used depth equals capacity — the write pointer has wrapped one extra time relative to the read pointer.

### Wrapping Around Condition
To distinguish full vs empty when binary pointers are equal, an extra MSB (unused MSB) is added to each pointer to track wrap counts:

- When the write pointer increments past the last FIFO address, its MSB toggles while the lower bits reset.
- The read pointer behaves similarly on wrap.
- If the MSBs differ, the write pointer has wrapped one more time than the read pointer (indicating potentially full).
- If the MSBs are the same, both pointers have wrapped the same number of times (indicating empty when binary addresses match).

---

## Gray Code Counter

Gray-code counters are used for pointer representation because only one bit changes per increment. This minimizes metastability issues when synchronizing multi-bit pointers across clock domains.

---

## Signal Definitions

- wclk : Write clock signal
- rclk : Read clock signal
- wdata : Write data bus
- rdata : Read data bus
- wclk_en : Write clock enable — controls writes to memory (no writes when FIFO is full, i.e., wfull = 1)
- wptr : Write pointer (Gray code)
- rptr : Read pointer (Gray code)
- winc : Write increment (trigger to increment wptr)
- rinc : Read increment (trigger to increment rptr)
- waddr : Binary write pointer address (index into FIFO memory for writes)
- raddr : Binary read pointer address (index into FIFO memory for reads)
- wfull : FIFO full flag (asserted when FIFO cannot accept more writes)
- rempty : FIFO empty flag (asserted when FIFO has no data to read)
- wrst_n : Active-low asynchronous reset for the write pointer domain
- rrst_n : Active-low asynchronous reset for the read pointer domain
- w_rptr : Read pointer synchronized into the write-clock domain (via 2-FF synchronizer)
- r_wptr : Write pointer synchronized into the read-clock domain (via 2-FF synchronizer)

---

## Design Modules

The design is divided into five primary modules:

1. FIFO.v  
   - Top-level wrapper. Instantiates the memory, pointer handlers, and synchronizers. In larger integrations this wrapper may be discarded and the internal modules instantiated directly.

2. FIFO_memory.v  
   - Dual-port RAM that provides simultaneous read and write ports with both clocks.

3. two_ff_sync.v  
   - Two-flop synchronizer module used to safely transfer pointer values between clock domains. There are two instances:
     - Write-to-read pointer synchronization
     - Read-to-write pointer synchronization

4. rptr_empty.v  
   - Read-pointer handler. Synchronized to the read clock and generates the FIFO empty signal (rempty) and next read address logic.

5. wptr_empty.v  
   - Write-pointer handler. Synchronized to the write clock and generates the FIFO full signal (wfull) and next write address logic.

---

## Testbench Implementation

The testbench validates FIFO functionality by generating stimuli and checking responses. Main cases included:

- Write random data and read it back, verifying data integrity.
- Fill FIFO to the full condition and attempt additional writes (should be blocked).
- Read from an empty FIFO and attempt additional reads (should be blocked or flagged).

The testbench uses independent read and write clocks and applies reset sequences to initialize the FIFO. Simulation stops after the test cases complete.

---

## Waveforms

Waveform screenshots demonstrating FIFO operation and verification are included below:

![Waveform 1](https://github.com/user-attachments/assets/0539f631-7e52-4d14-adfc-0bb51fefa1e2)
![Waveform 2](https://github.com/user-attachments/assets/89e49ce7-aee6-4af7-bf75-ac7ebe3f203a)
![Waveform 3](https://github.com/user-attachments/assets/a52c04f7-3074-4184-868e-3800d666ad24)

Elaborated design :
<img width="1058" height="514" alt="image" src="https://github.com/user-attachments/assets/eb617d20-72a6-4e14-809c-c90e488c3692" />

Implemented design :

![WhatsApp Image 2](https://github.com/user-attachments/assets/15c1dec8-5707-426d-9c66-4f28a1087278)

---


