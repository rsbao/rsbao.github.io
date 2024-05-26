---
layout: page
title: 32-bit RISC-V Datapath Layout
description: a 32-bit datapath including alu, regfile, shifter, etc.
img: assets/img/mp_ooo_diagram.png
importance: 1
category: school
related_publications: false
---
## Overview
For ECE 425's MP2, I created (by hand) a bit-sliced 32-bit RISC-V datapath. Starting from the ground up, I first created basic cells like NAND2, NOR2, MUX2, DFF, etc. 

In addition to the base functionality, we added several advanced features to further boost performance. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/mp_ooo_diagram.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    General block diagram of our CPU.
</div>
## Advanced Features
##### 2-way Superscalar 
Our processor decodes and renames up to 2 instructions in parallel, then sending them to 2 issue queues (and 1 load/store queue). Each of these queues feeds a set of function units. Finally, we commit up to 2 instructions in parallel.
##### 8-way fetch
Instead of fetching one instructinon at a time, our processor fetchs the entire 256-bit cacheline at once, alleviating the fetch bottleneck seen with a pipelined processor. This wide fetch caused many branch prediction implementation related headaches as described later.
##### BTB and LBHT/GShare Branch Predictors
Originally, I implemented a fully combinational branch target buffer (BTB) and branch history table (BHT) as a proof of concept. These tables were originally in registers. Later, due to power and area constraints, we switched to storing the BTB and BHT data in SRAM and predecoding instructions and predicting up to 2 branch/jump instructions per fetch cycle (since SRAM only has max 2 ports).
Due to the use of SRAM to save power and area, branch prediction takes 1 cycle longer than using registers

We use low bits of PC to index into BTB and LBHT and compare a tag to guarantee correctness of branch target.
A hash of the global history vector(GHV) and PC to used to index into GShare table.
We update LBHT on commits with a 1 bit taken/not taken counter.
We update GShare speculatively (when finished executing, but before commit).
An arbiter prefers whichever predictor that is less wrong (move away from predictor that mispredicted).
##### Dadda Multiplier
We designed a 8-bit Dadda Multiplier as a base module then put them together to make larger multipliers.
A 16-bit Dadda is built from 4 8-bit Daddas and a 32-bit Dadda is built on top of 4 16-bit Daddas.
Signed multiplies are taken care of in the 32-bit Dadda module. 
The signed bit of inputs a and b are first checked and the actual multiplication is always done using the unsigned version which is done by inverting and adding one. After the multiplication is done, the result is then flipped depending on the signed bits of the input. There is only 1 case where one of the inputs is negatively signed. 
##### Synopsys IP Sequential Divider

We hooked up the Synopsys IP sequential divider by adding some logic to deal with the complete signals and also handle signed division.
##### Pipelined I-Cache

The piplined I-Cache allows for a throughput of one cache response per cycle on hits.
It uses three pipeline stages and stalling logic to achieve this.

Trade-offs: pipelined i-cache increases the latency of first hit from two cycles to three cycles, but greatly improves the performance of multiple hits.

Performs best on programs that have branches and loops. After first loop traversal, the contents of the loop will be stored in the cache and the pipelined i-cache can service one request per cycle because loop body will always hit inside the cache
##### Non-blocking I-Cache

The I-Cache uses a reorder buffer in i-cache to store all i-cache requests in order. 
It also contains an issue queue to service read misses to burst memory. 
When requests in the issue queue are serviced, their data is sent back to the reorder buffer so that the corresponding entry inside the reorder buffer can be woken up.
The head of the reorder buffer "commits" when done by sending the stored address and stored data back to the processor.

Trade-offs: non-blocking i-cache requires a rob and issue queue structure, which increases area and power. Clogs the burst memory queue by constantly issuing read requests, so data requests may take significantly longer to be serviced. Thus, overly aggressive prefetching could decrease performance. 
Performs best on programs that have many linear instruction address access and few branches. Performs the worst on branches, as the prefetcher would be fetching the wrong address if a branch is taken. Non-blocking i-cache generally provides the same speedup as an i-cache with a next line prefetcher.

##### Memory Disambiguation

We implement partial memory ordering where loads can go out of order.
The implementation is based on the memory unit of MIPS R10000 where an indetermination dependency matrix is used for disambiguation as shown in the block diagram below.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/mips_r10k.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    From "Processor Microarchitecture: An Implementation Perspective"
</div>

Indetermination matrix issues memory operations when both older addresses are computed and their source operands are ready
Dependency matrix checks for address dependencies between the loads and stores and allows loads to read from dmem as long as older stores that have matching addresses are done.

