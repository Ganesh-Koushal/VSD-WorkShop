# Day 1 – Verilog RTL Design and Functional Simulation

## Objective

The goal of this experiment was to get hands-on with the basic RTL design and verification flow: writing a simple Verilog module, simulating it with **Icarus Verilog (iverilog)**, and checking its correctness by analyzing the output waveform in **GTKWave**. A 2:1 Multiplexer was used as the design under test for this exercise, run inside the VSDSquadron VM environment.

---

## Contents

1. [Core Concepts](#1-core-concepts)
2. [Simulation Flow](#2-simulation-flow)
3. [Lab Setup & Execution](#3-lab-setup--execution)
4. [Multiplexer Design](#4-multiplexer-design)
5. [Conclusion](#5-conclusion)

---

## 1. Core Concepts

**Simulator**
A tool that runs a Verilog design against a set of test inputs in software, so its behavior can be checked before any hardware is built. It's how functional bugs get caught early.

**Design**
The actual Verilog module — the RTL description of the circuit's logic and how it should behave.

**Testbench**
A separate, non-synthesizable Verilog file whose only job is to drive inputs into the design and let us observe/verify the outputs. It applies primary inputs through a stimulus generator and checks the primary outputs through a stimulus observer, as shown below.

<img width="1212" height="570" alt="image" src="https://github.com/user-attachments/assets/97087622-b7ad-499c-bfac-c53ee50dee59" />




---

## 2. Simulation Flow

`iverilog` compiles the design and testbench together into a single executable. Running that executable produces a `.vcd` (Value Change Dump) file, which records every signal transition over simulated time. GTKWave reads that file and renders it as waveforms.

<img width="1402" height="649" alt="image" src="https://github.com/user-attachments/assets/e447bc29-5bbc-4afe-ac25-b183dc5c8d39" />


---

## 3. Lab Setup & Execution

Environment: Ubuntu (VSDSquadron VM), working directory `~/sky130RTLDesignAndSynthesisWorkshop/verilog_files`

**Step 1 — Install the tools**

```bash
sudo apt install iverilog
sudo apt install gtkwave
```

**Step 2 — Compile design + testbench**

```bash
cd sky130RTLDesignAndSynthesisWorkshop/verilog_files
iverilog good_mux.v tb_good_mux.v
```

**Step 3 — Run the simulation**

```bash
./a.out
```

This generates the `.vcd` waveform file.

**Step 4 — View the waveform**

```bash
gtkwave tb_good_mux.vcd
```

📷 *Waveform output:

`<img width="1920" height="983" alt="Simulation WF" src="https://github.com/user-attachments/assets/35511dda-d0d1-4b24-aa89-dd75f2b290ba" />


**Step 5 — Synthesize with Yosys (introductory check)**

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_mux.v
synth -top good_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The `show` command generates a Graphviz-based schematic of the synthesized design.

📷 *Netlist output:

<img width="1920" height="983" alt="Netlist" src="https://github.com/user-attachments/assets/ad2c6d73-04b6-45e5-ac39-28f9a0c3bee5" />


---

## 4. Multiplexer Design

```verilog
module good_mux (
    input  i0,
    input  i1,
    input  sel,
    output reg y
);

always @ (*) begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

**Ports**

| Signal | Direction | Description |
|--------|-----------|--------------|
| `i0`   | input     | Selected when `sel = 0` |
| `i1`   | input     | Selected when `sel = 1` |
| `sel`  | input     | Select line |
| `y`    | output    | Mux output |

**Behavior**
The `always @(*)` block means the logic re-evaluates any time `i0`, `i1`, or `sel` changes. When `sel = 0`, `y` follows `i0`; when `sel = 1`, `y` follows `i1` — standard 2:1 mux behavior.

---

## 5. Conclusion

This lab walked through the core RTL verification loop — design, testbench, compile, simulate, view — using a simple 2:1 mux as the example, and took a first look at synthesis using Yosys with the Sky130 standard cell library. The main takeaway was understanding *why* each piece exists: the testbench isolates verification from the design itself, and the simulator/GTKWave combo gives a way to catch logic errors before ever touching synthesis. This sets up the foundation for Day 2 onward, where synthesis and timing come into the picture in more depth.
