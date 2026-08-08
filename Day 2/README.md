# Day 2 – Timing Libraries, Synthesis Approaches & Flip-Flop Coding Styles

## Objective

Day 2 builds on the basic simulation flow from Day 1 and moves into three areas that matter once a design actually needs to be synthesized: understanding what's inside a `.lib` timing library, comparing hierarchical vs. flattened synthesis in Yosys, and writing clean, synthesis-friendly RTL for flip-flops with different reset/set behavior.

---

## Contents

1. [Understanding the SKY130 Timing Library](#1-understanding-the-sky130-timing-library)
2. [Hierarchical vs. Flattened Synthesis](#2-hierarchical-vs-flattened-synthesis)
3. [Flip-Flop Coding Styles](#3-flip-flop-coding-styles)
4. [Lab Execution](#4-lab-execution)
5. [Conclusion](#5-conclusion)

---

## 1. Understanding the SKY130 Timing Library

### What is the SKY130 PDK

The SKY130 PDK is SkyWater Technology's open-source 130nm process design kit. It ships with characterized standard cell libraries that Yosys (and other synthesis tools) use to map generic RTL logic onto real, silicon-ready gates — along with their timing, power, and area data.

### Decoding the Library Filename

The library used throughout this workshop is `sky130_fd_sc_hd__tt_025C_1v80.lib`. Breaking down the corner naming:

| Segment | Meaning |
|---------|---------|
| `tt`    | Typical process corner (as opposed to fast-fast or slow-slow) |
| `025C`  | Characterized at 25°C |
| `1v80`  | Characterized at a 1.8V core supply |

This tells the synthesis tool exactly which process/voltage/temperature (PVT) corner the cell timing and power numbers correspond to.

📷 * <img width="1920" height="983" alt="SKY1300DK" src="https://github.com/user-attachments/assets/2959e21c-8ce6-4dc9-8ff8-7f58f77489d2" />
— SKY130 PDK overview.*

### Inspecting the `.lib` File

```bash
sudo apt install gedit
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

Opening the file shows each standard cell (AND, OR, NAND, flip-flops, etc.) along with its timing arcs, input capacitance, and power characteristics — this is the data Yosys draws on during technology mapping.

---

## 2. Hierarchical vs. Flattened Synthesis

### Hierarchical Synthesis

Keeps the RTL's module hierarchy intact — each sub-module is synthesized as its own block rather than being merged into one flat design.

**Advantages**
- Faster synthesis runtime, especially on larger designs
- Easier to trace results back to the original RTL for debugging
- Fits naturally into a modular, block-based design flow

**Trade-offs**
- Optimizations are limited to within each module — no cross-module optimization
- Some report/analysis steps need extra setup to work across hierarchy boundaries

**Example: `multiple_modules.v`**

```verilog
module sub_module1 (input a, input b, output y);
    assign y = a & b;
endmodule

module sub_module2 (input a, input b, output y);
    assign y = a | b;
endmodule

module multiple_modules (input a, input b, input c, output y);
    wire net1;
    sub_module1 u1 (.a(a), .b(b), .y(net1));   // net1 = a & b
    sub_module2 u2 (.a(net1), .b(c), .y(y));   // y = net1 | c, i.e. y = (a&b) + c
endmodule
```

Here, `multiple_modules` instantiates `sub_module1` and `sub_module2` rather than flattening their logic in directly. Running `synth -top multiple_modules` in Yosys without a `flatten` step keeps `u1` and `u2` as distinct sub-blocks in the resulting netlist — this is what hierarchical synthesis preserves.

📷 *<img width="1920" height="983" alt="Hierarchial Modules" src="https://github.com/user-attachments/assets/260e242c-82ca-4faa-94ee-0bd1884ec135" />
* · *<img width="1920" height="983" alt="Sub_module Ex" src="https://github.com/user-attachments/assets/2aaab61a-d11c-4abd-95d3-9470794a1886" />
*

### Flattened Synthesis

Collapses the entire module hierarchy into a single netlist using Yosys's `flatten` command, before optimization runs.

**Advantages**
- Enables aggressive, whole-design optimization across former module boundaries
- Produces one unified netlist, which can simplify certain downstream steps

**Trade-offs**
- Longer runtime on large designs
- Harder to debug — no clean mapping back to individual RTL modules
- Netlist complexity and memory usage go up

📷 *<img width="1920" height="983" alt="Flatten Netlist" src="https://github.com/user-attachments/assets/cff965e6-f38e-4a4e-b9c3-f1d5fe86063e" />
* · *<img width="1920" height="923" alt="Complete Netlist" src="https://github.com/user-attachments/assets/4337f00a-5a5e-4983-a933-9d144e4539ab" />
*

### Summary Comparison

| Aspect | Hierarchical | Flattened |
|---|---|---|
| Hierarchy | Preserved | Collapsed |
| Optimization scope | Per-module | Whole design |
| Runtime (large designs) | Faster | Slower |
| Debuggability | Easier — traces to RTL | Harder |
| Typical use case | Modularity, reporting, reuse | Maximum optimization |

---

## 3. Flip-Flop Coding Styles

Flip-flops are the basic storage elements in sequential logic, and how they're coded directly affects whether synthesis infers the correct hardware. Three common reset/set styles:

### Asynchronous Reset D Flip-Flop

```verilog
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @ (posedge clk, posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
The reset acts independently of the clock — as soon as `async_reset` goes high, `q` is forced to 0 immediately, regardless of the clock edge.

### Asynchronous Set D Flip-Flop

```verilog
module dff_async_set (input clk, input async_set, input d, output reg q);
  always @ (posedge clk, posedge async_set)
    if (async_set)
      q <= 1'b1;
    else
      q <= d;
endmodule
```
Same idea as above, but forces `q` to 1 instead of 0 when triggered.

### Synchronous Reset D Flip-Flop

```verilog
module dff_syncres (input clk, input sync_reset, input d, output reg q);
  always @ (posedge clk)
    if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
Unlike the async versions, this reset only takes effect on a clock edge — it's part of the sensitivity list's normal clocked behavior rather than an override.

---

## 4. Lab Execution

### Simulation (Icarus Verilog)

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd
```

📷 *<img width="1920" height="938" alt="DFF_waveform" src="https://github.com/user-attachments/assets/75971288-99f7-4644-9f93-c4a99cf8c9d7" />
 — simulated output waveform.*

### Synthesis (Yosys)

```bash
yosys

read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The extra `dfflibmap` step (compared to Day 1's combinational-only flow) is what maps the generic flip-flop inferred from RTL onto an actual sequential standard cell from the library, before `abc` handles the rest of the technology mapping.

📷 *`Async FF Netlist.png` — synthesized netlist for the asynchronous reset flip-flop.*

---

## 5. Conclusion

Day 2 connected the dots between RTL and real silicon: understanding what a `.lib` file actually encodes, seeing how the choice between hierarchical and flattened synthesis trades off runtime, debuggability, and optimization scope, and writing flip-flop RTL in a way that maps cleanly to real sequential cells. The `dfflibmap` step in particular is a good reminder that sequential elements need their own explicit library mapping — something that becomes more important once designs move past pure combinational logic like Day 1's mux.
