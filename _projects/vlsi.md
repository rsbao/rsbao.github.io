---
layout: page
title: 32-bit RISC-V Datapath Layout
description: a 32-bit datapath including alu, regfile, shifter, etc.
img: assets/img/datapath.png
importance: 1
category: school
related_publications: false
---
## Overview
For ECE 425's MP2, I created (by hand) a bit-sliced 32-bit RISC-V datapath using the FreePDK45 library and Cadence Virtuoso. The datapath uses a Harvard architecture, meaning that the instruction and data use different memory ports. The register file is write on low clock, read on high clock. Starting from the ground up, I first created basic cells like NAND2, NOR2, MUX2, DFF, etc. These were put together to make 1-bit modules like a register, alu, pc, etc. After integrating these into a one row bitslice, I slapped 32 of them together to make the full datapath. (Of course it wasn't that simple)
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/datapath.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Full datapath (shifter mostly finished)
</div>
## Basic cells
Shown below is a subset of all the basic cells I created.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/mux2_layout.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/xnor2_layout.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    MUX2, XNOR2 layouts
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dff_layout.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    DFF layout
</div>
## Bitslice Modules
Some noteworthy bitslice modules are shown below.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/alu_bitslice.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    1-bit ALU layout, composed of full adder, AND2, OR2, XOR2, MUX4 cells
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PC_bitslice.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    1-bit PC
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/regfile_1bit.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    1-bit Register file, composed of 32 1-bit registers
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/SHIFT_bitslice_NEW.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Part of a barrel shifter
</div>
## Full Bitslice
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bitslice.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Wide boi
</div>
## Full Datapath
Stacking 32 bitslices and connecting control and I/O signals gives us this lovely mess. I also had to rework the shifter many times to achieve the nice looking pattern shown below.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/datapath.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Full datapath
</div>
## Acknowledgements
Thanks to classmates/friends Jake Cheng, Amaan Shah, and Jack Tipping for staying many days and nights in Grainger 4th floor working on this abomination. Also thanks to TA Stanley Wu for creating this mental torture device!


