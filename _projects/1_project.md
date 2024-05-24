---
layout: page
title: OoO RISC-V Processor
description: a superscalar out-of-order cpu
img: assets/img/mp_ooo_diagram.png
importance: 1
category: school
related_publications: false
---
## Overview
For ECE 411's mp_ooo my team (Basic Macro Devices); Jake Cheng, Allen Peng, and I, completed a fully functioning out-of-order execution cpu in SystemVerilog. We used explicit register renaming in order to maintain in order commits while exploiting instruction level parallelism (ILP) to increase instruction throughput.

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

Use low bits of PC to index into BTB and LBHT, compare tag to guarantee correctness of branch target
Hash GHV and PC to index into GShare table
Update LBHT on commits, 1 bit taken/not taken
Update GShare speculatively (when finished executing, but before commit)
Arbiter prefers predictor that is less wrong (go away from predictor that mispredicted)
##### Dadda Multiplier
Designed a 8-bit Dadda Multiplier as the base and built on top of that
16-bit Dadda is built from 4 8-bit Dadda and 32-bit Dadda is built on top of 4 16-bit Dadda
Signed multiplies are taken care of in the 32-bit Dadda module. 
The signed bit of inputs a and b are first checked and the actual multiplication is always done using the unsigned version which is done by inverting and adding one. After the multiplication is done, the result is then flipped depending on the signed bits of the input. There is only 1 case where one of the inputs is negatively signed. 
##### Synopsys IP Sequential Divider

Hooked up the synopsys IP
Had to create a shadow register to make division complete signal 1 cycle because the IP holds the complete signal high until the next division
##### Pipelined I-Cache

Allows a throughput of one cache response per cycle on hit
Uses three pipeline stages and stalling logic to achieve targeted throughput
Trade-offs: pipelined i-cache increases the latency of first hit from two cycles to three cycles, but greatly improves the performance of multiple hits
Performs best on programs that have branches and loops. After first loop traversal, the contents of the loop will be stored in the cache and the pipelined i-cache can service one request per cycle because loop body will always hit inside the cache
##### Non-blocking I-Cache

Uses a reorder buffer in i-cache to store all i-cache requests in order. 
Uses an issue queue to service read misses to burst memory
When requests in the issue queue are serviced, their data is sent back to the reorder buffer so that the corresponding entry inside the reorder buffer can be woken up
Head of the reorder buffer commits when done by sending the stored address and stored data back to the processor
Trade-offs: non-blocking i-cache requires a rob and issue queue structure, which increases area and power. Clogs the burst memory queue by constantly issuing read requests, so data requests may take significantly longer to be serviced. Thus, overly aggressive prefetching could decrease performance. 
Performs best on programs that have many linear instruction address access and few branches. Performs the worst on branches, as the prefetcher would be fetching the wrong address if a branch is taken. Non-blocking i-cache generally provides the same speedup as an i-cache with a next line prefetcher.

##### Memory Disambiguation

Partial Ordering where loads can go out of order
Implementation is similar to the memory unit of MIPS R10000 where an indetermination matrix and dependency matrix is used for disambiguation as shown in the block diagram below

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/mips_r10k.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    From  {% cite gonzalez2011microarch %}
</div>

Indetermination matrix issues memory operations when both older addresses are computed and their source operands are ready
Dependency matrix checks for address dependencies between the loads and stores and allows loads to read from dmem as long as older stores that have matching addresses are done


<!-- To give your project a background in the portfolio page, just add the img tag to the front matter like so:

    ---
    layout: page
    title: project
    description: a project with a background image
    img: /assets/img/12.jpg
    --- -->

<!-- <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>

You can also put regular text between your rows of images, even citations {% cite einstein1950meaning %}.
Say you wanted to write a bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, _bled_ for your project, and then... you reveal its glory in the next row of images.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above: -->

<!-- {% raw %}

```html
<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/mp_ooo_diagram.png" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
```

{% endraw %} -->
