Asynchronous FIFO Design.

This repo contains verilog code for an asynchronous FIFO.

Introduction
FIFO stands for "First-In, First-Out." It is a type of data structure or buffer in which the first data element added (the "first in") is the first one to be removed (the "first out"). This structure is commonly used in scenarios where order of operations is important.

Async FIFO, or Asynchronous FIFO, is a FIFO buffer where the read and write operations are controlled by independent clock domains. This means that the writing process and the reading process are driven by different clocks, which are not synchronized. Async FIFOs are used to safely transfer data between these asynchronous clock domains.
<img width="652" height="222" alt="image" src="https://github.com/user-attachments/assets/f2cb9967-e6f1-4708-a9fd-220ab91cacd1" />

Design Space Exploration and Design Strategies
The block diagram of async. FIFO that is implemented in this repo is given below. Thin lines represent single bit signal where as thisck lines represent multi-bit signal.

<img width="1128" height="575" alt="image" src="https://github.com/user-attachments/assets/bba3cdda-b81b-405d-a8f4-37c8cba664a2" />

Full, empty and wrapping condition
The conditions for the FIFO to be full or empty are as follows:

Empty Condition: The FIFO is empty when the read and write pointers are both equal. This condition happens when both pointers are reset to zero during a reset operation, or when the read pointer catches up to the write pointer, having read the last word from the FIFO.

Full Condition: The FIFO is full when the write pointer has wrapped around and caught up to the read pointer. This means that the write pointer has incremented past the final FIFO address and has wrapped around to the beginning of the FIFO memory buffer.

To distinguish between the full and empty conditions when the pointers are equal, an extra bit is added to each pointer. This extra bit helps in identifying whether the pointers have wrapped around:

Wrapping Around Condition: When the write pointer increments past the final FIFO address, it will increment the unused Most Significant Bit (MSB) while setting the rest of the bits back to zero. The same is done with the read pointer. If the MSBs of the two pointers are different, it means that the write pointer has wrapped one more time than the read pointer. If the MSBs of the two pointers are the same, it means that both pointers have wrapped the same number of times.

Gray code counter
Gray code counters are used in FIFO design because they only allow one bit to change for each clock transition. This characteristic eliminates the problem associated with trying to synchronize multiple changing signals on the same clock edge, which is crucial for reliable operation in asynchronous systems.


Signals Defination
Following is the list of signals used in the design with their defination:-

wclk: Write clock signal.
rclk: Read clock signal.
wdata: Write data bits.
rdata: Read data bits.
wclk_en: Write clock enable, this signal controls the write operation to the FIFO memory. Data must not be written if the FIFO memory is full (wfull = 1).
wptr: Write pointer (Gray).
rptr: Read pointer (Gray).
winc: Write pointer increment. Controls the increment of the write pointer (wptr).
rinc: Read pointer increment. Controls the increment of the read pointer (rptr).
waddr: Binary write pointer address. Loaction (address) of the FIFO memory to which data (wdata) to be written.
raddr: Binary read pointer address. Loaction (address) of the FIFO memory from where data (rdata) to be read.
wfull: FIFO full flag. Goes high if the FIFO memory is full.
rempty: FIFO empty flag. Goes high if the FIFO memory is empty.
wrst_n: Active low asynchronous reset for the write pointer handler.
rrst_n: Active low asynchronous reset for the read pointer handler.
w_rptr: Read pointer signal synchronized to the wclk domain via 2 flip-flop synchronized.
r_wptr: Write pointer signal synchronized to the rclk domain via 2 flip-flop synchronized.

Dividing System Into Modules
For implementing this FIFO, I have divided the design into 5 modules:-

FIFO.v: The top-level wrapper module includes all clock domains and is used to instantiate all other FIFO modules. In a larger ASIC or FPGA design, this wrapper would likely be discarded to group the FIFO modules by clock domain for better synthesis and static timing analysis.
FIFO_memory.v: This modules contains the buffer or the moeory of the FIFO, which has both the clocks. This is a dual port RAM.
two_ff_sync.v: This module consist of two flip-flops that are connected to each other to form a 2 flip-flop synchronizer. This module will have two instances, one for write to read clock pointer synchronization and other for read to write clock pointer synchronization
rptr_empty.v: This modlue consist of the logic for the Read pointer handler. It is completely synchronized by read clock and consist of the logic for generation of FIFO empty signal.
wptr_empty.v: This modlue consist of the logic for the Write pointer handler. It is completely synchronized by write clock and consist of the logic for generation of FIFO full signal.

Testbench Case Implementation

The testbench for the FIFO module generates random data and writes it to the FIFO, then reads it back and compares the results. The testbench includes three test cases:-

1.) Write data and read it back.
2.) Write data to make the FIFO full and try to write more data.
3.) Read data from an empty FIFO and try to read more data.
The testbench uses clock signals for writing and reading, and includes reset signals to initialize the FIFO. The testbench finishes after running the test cases.


Waveforms: 
<img width="1335" height="316" alt="image" src="https://github.com/user-attachments/assets/56cf20df-8f95-45b0-9b20-38a92ccff962" />
<img width="1334" height="311" alt="image" src="https://github.com/user-attachments/assets/d2078d60-0cd7-48db-a985-86ab497a3a51" />
<img width="1352" height="390" alt="image" src="https://github.com/user-attachments/assets/9c4f88a5-a391-495b-8600-89d7a1efd9d3" />
![WhatsApp Image 2025-12-10 at 17 21 05](https://github.com/user-attachments/assets/404fff6e-c057-4d8e-9b5f-8edeef740b2e)
![WhatsApp Image 2025-12-10 at 17 21 04](https://github.com/user-attachments/assets/14bfc653-537b-4cc8-b0cf-16914a04158b)



