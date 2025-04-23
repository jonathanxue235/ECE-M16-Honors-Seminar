FPGAs greatly reduce the cost of chip manufacturing as you wouldn't need to design a custom chip every time you're prototyping. 

However, FPGA’s programmable logic blocks average 35x larger and 4x slower than corresponding ASIC implementations

Basic FPGA Architecture Evaluation flow: a suite of benchmark applications, an architecture model, and a CAD system.

#### Benchmark Applications
Used to test the efficiency of the FPGA's ability to implement various architectures
Examples: MCNC20, the VTR, and the Titan23
#### FPGA Architecture
Number and types of blocks, distribution of wire segment lengths, size of logic clusters and logic elements
The nuances of the architecture of an FPGA is captured in an architecture description file
#### CAD System
a re-targetable CAD system such as VTR is used to map the selected benchmark applications on the specified FPGA architecture. Such a CAD system consists of a sequence of complex optimization algorithms that synthesizes a benchmark written in an HDL into a circuit netlist, maps it to the different FPGA blocks, places the mapped blocks at specific locations on the FPGA, and routes the connections between them using the specified programmable routing architecture. 

Some questions to ponder when designing FPGA architecture:
1. What functionality should be hardened (i.e. implemented as a new ASIC-style block) in the FPGA architecture? 
2. How flexible should this block be? 
3. How much of the FPGA die area should be dedicated to it?

We want to weigh the pros and cons of whether a block should be hardened or not, as hardening it makes it faster but also limited in use cases. 

### FPGA Architecture Evolution
#### Programmable Logic
In the early days, FPGAs consisted of programmable array logic (PAL) architectures, which consisted of a bunch of `AND` gates feeding into a bunch of `OR` gates. 
![[./Images/0416113631.png]]
##### Problems with this architecture
Not scalable, as number of programmable switches grew exponentially with the increase in complexity. 

#### Look-Up-Tables
Xilinx pioneered the first lookup-table-based (LUT-based) FPGA in 1984, which consisted of an array of SRAM-based LUTs with programmable interconnect between them.

LUT-based FPGAs dominate to this day. 

A K-LUT can implement any K-input Boolean function by storing its truth table in configuration SRAM cells
![[./Images/0416122206.png]]
![[./Images/0416114800.png]]
To better help understand the architecture of a LUT, here's a good [link](https://hardwarebee.com/overview-of-lookup-tables-in-fpga-design/)

LUTs are a truth tables. Essentially, the compiler for HDL just compiles all of the logic and stores it as a truth table. They are stored in SRAM cells

LUTs have increased gradually (the number k has increased over the years)

#### High-Level Overview of FPGA Architecture
**Basic Logic Element (BLE)** consists of a K-LUT coupled with an output register and bypassing 2:1 multiplexers 
![[./Images/0416121835.png]]
This allows the BLE to both implement combinational and sequential logic (flip flops)

**Logic Blocks (LB)** are made up of a bunch of BLEs and local interconnects, which consists of multiplexers made up of signal sources (BLE outputs and logic block inputs) and destinations (BLE inputs)
![[./Images/0416122148.png]]

Size of LUTs (K) and LBs (N) have increased over the years. However, an increase in K is exponential, but the speed increase given is linear, so the tradeoff is not great after a certain point. 
N. Ahmed and Rose [24] empirically evaluated these trade-offs and concluded that LUTs of size 4–6 and LBs of size 3–10 BLEs offer the best area-delay product for an FPGA architecture, with 4-LUTs leading to a better area but 6-LUTs yielding a higher speed

#### Fracturable LUTs
The adaptive logic module (ALM) in the Stratix II architecture implemented a {6, 2}-LUT that had 8 input and 2 output ports. Thus, an ALM can implement a 6-LUT or two 5-LUTs sharing 2 inputs (and therefore a total of 8 distinct inputs). Pairs of smaller LUTs could also be implemented without any shared inputs, such as two 4-LUTs or one 5-LUT and one 3-LUT. With a fracturable 6-LUT, larger logic functions are implemented in 6-LUTs reducing the logic levels on the critical path and achieving performance improvement. On the other hand, smaller logic functions can be packed together (each us- ing only half an ALM), improving area-efficiency. The LB in Stratix II not only increased the performance by 15%, but also reduced the logic and routing area by 2.6% com- pared to a baseline 4-LUT-based LB.

![[./Images/0416123901.png]]
When architectures moved towards fracturable LUTs, they started adding a second Flip Flop (FF) and later on more flip flops as shown in the Stratix V architecture



![[./Images/0416132758.png]]
Pulse latch is more area efficient but there are cons (**research this**)



#### Arithmetic blocks
 Murray et al. found that 22% of the logic elements in a suite of FPGA designs were implementing arithmetic. 
 It is highly inefficient to implement arithmetic in LUTs as it takes up 2 LUTs for each bit in a ripple carry adder.
 **Therefore, all modern FPGA architectures include hardened arithmetic circuitry in their logic blocks** 
 A lot of other arithmetic related operations are also hardened, can be found in page 11 of the PDF.

#### Deep Learning Applications
Many modern ML tasks require significant Multiply Accumulate (MAC) operations. Thus, there are specialized FPGAs that have hardened things that help with MAC. The most promising solutions (as of this paper) can increase speed by 1.7x. 

![[./Images/0416133714.png]]


### Programmable Routing
Programmable routing commonly accounts for over 50% of both the fabric area and the critical path delay of applications, so its efficiency is crucial. Programmable routing is composed of pre-fabricated wiring segments and programmable switches. By programming an appropriate sequence of switches to be on, any function block output can be connected to any input. 
Two main classes of FPGA routing architecture: hierarchical FPGAs and 

#### Hierarchical FPGAs
More frequent LBs are closer to each other. Less frequently communicated LBs are further apart.
![[./Images/0416145031.png]]
Now obsolete due to the long wiring required for information to be sent from one end to another

#### Island-Style FPGAs
The modern method of FPGA architectures
Island-style routing includes three components: routing wire segments, connection blocks (multiplexers) that connect function block inputs to the routing wires, and switch blocks (programmable switches) that connect routing wires together to realize longer routes


Some of the routing architecture parameters include: how many routing wires each logic block input or output can connect to ($F_c$), how many other routing wires each wire can connect to ($F_s$), the lengths of the routing wire segments, the routing switch pattern, the electrical design of the wires and switches themselves, and the number of routing wires per channel.
![[./Images/0416151451.png]]

$F_c$ = how many routing wires each logic block input or output can connect to 
$F_s$ = how many other routing wires each wire can connect to

There are varying lengths in wires after people found out that only using short wires was not efficient

**Creating a good routing architecture involves managing many complex trade-offs. It should contain enough programmable switching and wire segments that the vast majority of circuits can be implemented; however, too many wires and switches waste area.**


#### Switches
Early FPGAs used pass gate transistors controlled by SRAM cells to connect wires. While this is the smallest switch possible in a conventional CMOS process, the delay of routing wires connected in series by pass transistors grows quadratically, making them very slow for large FPGAs. Adding some tri-state buffer switches costs area, but improves speed

Direct drive switches 

#### Wiring Delay
Long wires still have very long delay
By placing registers directly within or strategically near the routing paths, the long delay associated with these wires can be broken up across multiple clock cycles. While adding the register itself introduces a small delay, enabling pipelining allows the circuit designer to increase the overall clock frequency, thereby "reducing" the effective delay per clock cycle for long routes

### Programmable IO
It's hard for one set of physical IOs to programmably support a wide variety of IO interfaces, which is why so much of the space in FPGAs are dedicated to IO
This is because it requires adapting to various voltage levels, electrical characteristics, timing specifications, and command protocols
#### How IO can be programmable

##### Electrical and Timing Programmability
**Voltage Adaptation**: IOs use IO buffers that can operate across a range of voltages. These IOs are grouped into banks, each with a separate voltage supply (Vddio rail), allowing different banks to operate at different voltage levels.

**Single-ended and Differential Support**: Each IO can be used individually for single-ended standards, or pairs of IOs can be programmed to form the positive and negative lines for differential IO standards.

**Programmable Drive Strength and Termination**: IO buffers are implemented with multiple parallel transistors, allowing their drive strengths to be programmably adjusted. By enabling certain transistors even when not driving an output, IOs can also be programmed to implement different on-chip termination resistances to minimize signal reflections.

**Timing Adjustment**: Programmable delay chains provide configurability for making fine delay adjustments to signal timing to and from the IO buffer

There are also high-speed IOs for PCIE / Ethernet, which can lead up to speeds of up to 28 Gb/s. However, these high speed IOs are limited in programmability.

### On-Chip Memory
Flip Flops (FFs) were first used as on-chip memory. 
However, as the need for memory grew for various use cases, FPGAs began to implement BRAM (or block RAM). 

#### BRAM

**BRAM** consisted of an SRAM-based memory core with additional peripheral circuitry to make them more configurable for multiple purposes and to con- nect them to the programmable routing. 

![[./Images/0423121651.png]]

**The main architectural decisions in designing FPGA BRAMs are choosing their capacity, data word width, and number of read/write ports. More capable BRAMs cost more silicon area, so architects must carefully balance BRAM design choices while taking into account the most common use cases in application circuits.**

SRAM area grows linearly while peripherals grow sub-linearly, meaning **larger** BRAMs have higher area per bit

Because the FPGA on-chip memory must satisfy the needs of every application implemented on that FPGA, it is also common to add extra configurability to BRAMs to allow them to adapt to application needs. 
	Example: Configurable Width
		Rather than just having 4x8 bits of data, you can have 2x16 or 1x32


#### LUT-RAMs
In addition to building BRAMs, FPGA vendors can add circuitry that allows designers to repurpose the LUTs that form the logic fabric into additional RAM blocks.

LUTs are read only post-flash, which is not desirable for memory
Only a portion (e.g., half) of the logic blocks are made LUT-RAM capable to reduce area cost, as it's unusual for designs to convert more than 50% of logic fabric to LUT-RAMs


#### RAM Mapping
  
Designers typically need many different memory configurations. Manually determining how to combine BRAMs and LUT-RAMs for each configuration would be laborious and architecture-specific

FPGA vendor CAD tools include a RAM mapping stage that automatically implements logical memories defined by the user onto the physical BRAMs and LUT-RAMs on the chip. The mapper selects the physical memory implementation and generates necessary soft logic (using LUTs and routing) to combine multiple physical memories if needed. The figure below illustrates mapping a large logical RAM onto smaller physical BRAMs by combining them in parallel.

![[./Images/Fig13.png]]

Trend of memory to logic ratio on FPGAs
![[./Images/0423123007.png]]


Magnetic Tunnel Junctions could serve as a possible replacement to the currently dominating SRAM based BRAMs in FPGAs. 


### DSP Blocks
As multiplier needs increased with signal processing and communication demands, multipliers became a topic of hardening as dedicated circuits in FPGAs. 
Xilinx first introduced 18x18 hard multiplier blocks, with Altera later implementing a dedicated DSP block

![[./Images/0423125242.png]]

DSP blocks are crucial for applications involving arithmetic operations, particularly multiply-accumulate (MAC), which is core to signal processing and deep learning

In addition, FIR filters are a key use case. Figure 17 illustrates a systolic symmetric FIR filter. Early DSP blocks could implement portions of this structure (highlighted by dotted boxes). Later blocks, like those in Stratix V, could implement the entire symmetric FIR filter structure within the DSP block.

#### Deep Learning
Deep learning (DL) has become a major workload, with MACs as a core operation. While low-precision MACs can be implemented in soft logic, DSP blocks are increasingly optimized for DL inference
This has resulted in the following: 
High-performance computing (HPC): Adding native support for single-precision floating-point (fp32) multiplication
Increased density for low-precision integer multiplication: Specifically targeting DL inference, which often uses 8-bit or narrower operands

**Example** Intel's Agilex DSP block supports int9, half-precision floating-point (fp16), and brain float (bfloat16)15 . Xilinx Versal supports int8 multiplications in its DSP58 tiles


There have also been shifts toward AI tensor blocks. These blocks may introduce features like data reuse register networks to efficiently feed inputs to the large number of multipliers


### Interposers
Interposers serve as a means to bridge multiple silicon dice together with dense interconnections. 

This is especially useful because state-of-the-art fabrication processes result in low yield, so breaking the silicon into smaller die will help with yield. 

Another motivation is to enable integration of different specialized chiplets 

The core challenge is that interposer connections are larger and slower than the fine-grained routing tracks within a single die. This can limit the amount of routing bandwidth that can cross die boundaries. FPGA vendors address this through CAD tools that optimize placement to minimize inter-die crossings and architectural changes that improve the flexibility of connections to interposer tracks. Using high-bandwidth, packet-switched structures like the NoC is an efficient way to utilize the limited interposer bandwidth


### Other FPGA Components
There are many other FPGA components such as the configuration circuitry, enabling **partial reconfiguration**

There are also encoders to encrypt bitstreams for security measures.
