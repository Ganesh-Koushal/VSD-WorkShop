# Day 6 – Open-Source EDA, OpenLane & the RTL-to-GDSII Flow

## Objective

Day 6 zooms out from individual RTL/synthesis labs to look at the *entire* ASIC implementation pipeline — from a Verilog description all the way to a manufacturable GDSII layout — using fully open-source tooling. This covers what a semiconductor foundry and a PDK actually are, why SkyWater's SKY130 process matters for open-source silicon, how OpenLane stitches together the individual tools into one automated flow, and what each stage (synthesis, STA, floorplanning, placement, CTS, routing, physical verification) is actually responsible for.

---

## Contents

1. [The Open-Source EDA Ecosystem](#1-the-open-source-eda-ecosystem)
2. [Foundries and PDKs](#2-foundries-and-pdks)
3. [SkyWater & SKY130](#3-skywater--sky130)
4. [OpenLane](#4-openlane)
5. [PicoRV32 as a Realistic Test Design](#5-picorv32-as-a-realistic-test-design)
6. [The Full RTL-to-GDSII Flow](#6-the-full-rtl-to-gdsii-flow)
7. [Stage-by-Stage Breakdown](#7-stage-by-stage-breakdown)
8. [Tools Reference](#8-tools-reference)
9. [Conclusion](#9-conclusion)

---

## 1. The Open-Source EDA Ecosystem

Commercial ASIC design has traditionally depended on expensive, closed-source EDA tools. Open-source alternatives make the entire flow accessible for learning and experimentation, without needing a commercial license at any stage.

| Purpose | Open-Source Tool |
|---|---|
| RTL Synthesis | Yosys |
| Logic optimization / tech mapping | ABC |
| Static Timing Analysis | OpenSTA |
| Physical design (floorplan, placement, etc.) | OpenROAD |
| Detailed routing | TritonRoute |
| Layout, DRC, physical verification | Magic |
| LVS | Netgen |
| Layout viewing | KLayout |
| DFT-related work | Fault |

Chained together correctly, these tools form a complete, license-free RTL-to-GDSII pipeline — which is exactly what OpenLane automates (Section 4).

---

## 2. Foundries and PDKs

A **semiconductor foundry** is the manufacturing side of the equation — it fabricates chips based on a physical layout the designer produces, and its process determines things like transistor characteristics, available metal layers, design rules, and standard-cell offerings.

```
Chip Designer → RTL → EDA Flow → Physical Layout → GDSII → Foundry → Fabricated Chip
```

A **PDK (Process Design Kit)** is the bridge between the design tools and that specific manufacturing process — it packages everything the tools need to understand the target technology: standard-cell and timing libraries, LEF/GDS files, SPICE models, DRC/LVS rules, and RC extraction data. Without a PDK, an EDA tool has no way to know what the actual manufacturing process supports or requires.

---

## 3. SkyWater & SKY130

**SkyWater Technology**, in partnership with Google, opened up its 130nm process (**SKY130**) as a fully open-source PDK — a genuinely significant moment for open-source silicon, since it made it possible to run a real, manufacturable process through an entirely open EDA flow rather than a simulated/toy technology.

SKY130 isn't cutting-edge (modern commercial chips use far smaller nodes), but that's not really the point here — it's mature, well-documented, and open, which makes it ideal for actually *learning* the complete ASIC flow rather than fighting an unfamiliar or restricted toolchain.

*(Note: "130nm" is a process-node label, not a literal statement that every feature on the chip measures exactly 130nm.)*

---

## 4. OpenLane

**OpenLane** is the automation layer that ties the individual open-source tools above into one coherent, scripted flow — taking a digital RTL design through synthesis, floorplanning, placement, CTS, routing, and verification, and producing a GDSII file at the end, without manually invoking each tool by hand.

```
RTL → Synthesis → Gate-Level Netlist → Floorplanning → Placement
    → CTS → Routing → Physical Verification → GDSII
```

📷 *`GDSII Flow.png` — the end-to-end flow OpenLane automates.*

---

## 5. PicoRV32 as a Realistic Test Design

**PicoRV32** (by Clifford Wolf) is a compact RISC-V RV32 processor core — small enough to be practical for full-flow experimentation, but complex enough (instruction decode, register file, ALU, control logic, memory interface) to actually stress-test synthesis and physical implementation in a way a simple counter or mux never could.

Running PicoRV32 through the flow demonstrates that this isn't just a toolchain for trivial examples — it can take a genuine processor core all the way to layout.

---

## 6. The Full RTL-to-GDSII Flow

```
RTL Design
   ↓
RTL Synthesis (Yosys + ABC)
   ↓
Gate-Level Netlist
   ↓
Static Timing Analysis (OpenSTA)
   ↓
Floorplanning
   ↓
Power Planning
   ↓
Placement
   ↓
Clock Tree Synthesis (CTS)
   ↓
Optimization
   ↓
Global Routing
   ↓
Detailed Routing
   ↓
RC Extraction
   ↓
Post-Route STA
   ↓
Physical Verification (DRC + LVS)
   ↓
GDSII
```

📷 *`openlane_tcl.png` — running this flow through OpenLane's Tcl interface.*

---

## 7. Stage-by-Stage Breakdown

### RTL Design
Describes hardware behavior/structure in an HDL (Verilog, SystemVerilog, VHDL) without committing to physical geometry. A one-line example:
```verilog
always @(posedge clk)
    q <= d;
```
describes a register — nothing about how it's physically laid out yet.

### RTL Synthesis
Yosys converts the behavioral RTL into a structural netlist; ABC then performs logic optimization and technology mapping, matching that structural logic onto real SKY130 standard cells (AND/OR/NAND/NOR gates, inverters, buffers, muxes, flip-flops).

### Static Timing Analysis (STA)
Checks whether every signal path meets its timing budget, without needing to actually simulate the design. Two key metrics:
- **WNS (Worst Negative Slack)** — the single worst timing margin across all paths; `WNS ≥ 0` generally means that timing requirement is met.
- **TNS (Total Negative Slack)** — the sum of all negative slack across every violating path; `TNS = 0` means no violations in that category.

Other relevant terms: clock period, data arrival/required time, slack, critical path, setup/hold violations.

### Floorplanning
Lays out the chip's high-level physical organization — die area, core area, standard-cell region, I/O placement, and power structure:
```
+--------------------------------+
|              DIE               |
|     +----------------------+   |
|     |         CORE         |   |
|     |    Standard Cells    |   |
|     +----------------------+   |
+--------------------------------+
```
A good floorplan has downstream effects on timing, routability, congestion, and power — getting it wrong early causes problems that are expensive to fix later.

### Power Planning
Builds the power/ground distribution network — rings, straps, and rails connecting VDD/VSS down to every standard cell — to keep supply voltage stable and limit IR drop across the die.

### Placement
- **Global placement** finds approximate cell positions while optimizing the design overall.
- **Detailed placement** then legalizes those positions into valid physical sites.

Both stages are balancing wire length, timing, congestion, and area utilization simultaneously.

### Clock Tree Synthesis (CTS)
Builds a buffered distribution tree so the clock reaches every flip-flop with minimal skew:
```
        CLOCK
          |
        Buffer
       /      \
   Buffer    Buffer
   /   \      /   \
  FF   FF    FF   FF
```
CTS specifically targets clock skew, delay, fanout, and transition time.

### Routing
- **Global routing** determines approximate wiring paths between cells.
- **Detailed routing** (via TritonRoute) then creates the actual physical wires and vias, obeying the technology's design rules for spacing, width, and via placement.

### RC Extraction
Real wires have resistance and capacitance that add delay beyond what earlier estimates assumed. RC extraction measures these parasitics from the actual routed geometry, feeding more accurate numbers into the next timing check.

### Post-Route STA
Timing analysis re-run after routing, now using extracted parasitics — this is the most realistic timing picture available before tapeout, since it accounts for delay contributed by the actual physical interconnect rather than estimates.

### Physical Verification
- **DRC (Design Rule Check)** — confirms the layout obeys the foundry's manufacturing rules (minimum widths, spacing, via rules, etc.).
- **LVS (Layout Versus Schematic)** — confirms the physical layout actually matches the intended gate-level netlist.
```
Gate-Level Netlist → (compare) → Physical Layout → LVS Result
```
Open-source tools Magic (DRC) and Netgen (LVS) handle this stage.

### GDSII Generation
The final, verified layout is exported as a **GDSII** file — the standard format encoding all physical shapes, layers, cells, and interconnect geometry that the foundry needs to fabricate the chip.
```
... → Routing → Physical Verification → GDSII → Tapeout → Fabrication
```

---

## 8. Tools Reference

| Stage | Tool | Function |
|---|---|---|
| RTL Synthesis | Yosys | RTL → gate-level netlist |
| Logic Optimization | ABC | Optimization + technology mapping |
| Static Timing Analysis | OpenSTA | Timing verification |
| Physical Design | OpenROAD | Floorplan, placement, physical implementation |
| Clock Tree Synthesis | OpenROAD / CTS | Clock network construction |
| Global Routing | OpenROAD | Approximate wiring paths |
| Detailed Routing | TritonRoute | Physical wires and vias |
| RC Extraction | OpenRCX | Parasitic extraction |
| DRC | Magic | Design-rule checking |
| LVS | Netgen | Layout-vs-schematic checking |
| Layout Viewing | KLayout / Magic | Visual inspection |
| Final Output | — | GDSII |

---

## 9. Conclusion

Day 6 connected all the earlier RTL-level work to the physical reality of chip manufacturing — synthesis, optimization, STA, floorplanning, placement, CTS, routing, and physical verification, run end-to-end through OpenLane on the SKY130 PDK, using PicoRV32 as a realistically complex test design. The big-picture takeaway: RTL is only the starting description of intended behavior — everything from here (technology mapping, timing closure, physical placement, real metal wires, and manufacturing rule compliance) is what actually turns that description into a fabricable chip. Understanding this full pipeline, even at a conceptual level, makes it much clearer why decisions made way back at the RTL stage (Days 1–5) ripple all the way down to area, timing, and power in the final GDSII.
