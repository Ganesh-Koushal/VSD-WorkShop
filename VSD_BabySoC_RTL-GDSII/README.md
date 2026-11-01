# 🚀 BabySoC RTL-to-GDSII Design Flow

### VLSI System Design Workshop | RTL Simulation • Logic Synthesis • Physical Design • Static Timing Analysis

This directory consolidates my **BabySoC RTL Logic Synthesis and Physical Design work** completed as part of the VLSI System Design (VSD) workshop.

The project documents the progression from RTL simulation and logic synthesis to physical implementation, routing analysis, and Static Timing Analysis (STA).

The repository contains design-related images, synthesized netlists, simulation waveforms, implementation reports, and timing-analysis results that document the different stages of the design flow.

---

## 📌 Table of Contents

- [1. Project Overview](#-1-project-overview)
- [2. Repository Structure](#-2-repository-structure)
- [3. RTL Logic Synthesis](#-3-rtl-logic-synthesis)
- [4. Physical Design](#-4-physical-design)
- [5. Design Flow](#-5-design-flow)
- [6. Static Timing Analysis](#-6-static-timing-analysis)
- [7. Key Learning Outcomes](#-7-key-learning-outcomes)
- [8. Project Documentation](#-8-project-documentation)
- [9. Author](#-9-author)

---

## 🎯 1. Project Overview

The objective of this work is to understand and document the major stages involved in transforming a digital design description into a physically implemented chip design.

The project is divided into two major sections.

| Stage | Description |
|---|---|
| RTL Simulation | Examines the functional behavior of the design through simulation results and waveforms. |
| Logic Synthesis | Documents the synthesized design, netlists, and synthesis statistics. |
| Physical Design | Covers the physical implementation stages, including floorplanning, placement, clock-tree synthesis, and routing. |
| Static Timing Analysis | Examines timing reports and slack results across multiple process-voltage-temperature (PVT) corners. |

The two project directories preserve the documentation and supporting files for their respective stages.

---

## 📂 2. Repository Structure

```text
VSD_BabySoC_RTL-GDSII/
│
├── README.md
│
├── BabySoC_RTL_Logic_Synthesis/
│   ├── README.md
│   ├── pre_synth_babysoc.png
│   ├── post_babysoc_sim.png
│   ├── stats.png
│   ├── stat_after_reduction.png
│   ├── vsdbabysoc_net.png
│   ├── rvmyth_net.png
│   ├── rvmyth_net2.png
│   └── clk_gate_babysoc_net.png
│
└── Lab_PD_VSDBabySoC/
    ├── README.md
    ├── Floorplan.png
    ├── Placement.png
    ├── CTS.png
    ├── POST_ROUTE.png
    ├── STA_VSDBabySoC.png
    ├── WNS_PLOT.png
    ├── TNS_PLOT.png
    └── ...additional reports and implementation images
```

*Note: The tree above highlights representative files. Refer to the individual directories for their complete contents.*

---

## 🔬 3. RTL Logic Synthesis

**Directory:** `BabySoC_RTL_Logic_Synthesis/`

This section documents the simulation and logic-synthesis work associated with the BabySoC design.

It contains visual evidence of the design at different stages, including simulation results, synthesized netlists, and synthesis statistics.

### 3.1 Simulation and Design Visualization

| File | Purpose |
|---|---|
| `pre_synth_babysoc.png` | Supporting visualization associated with the pre-synthesis design stage. |
| `post_babysoc_sim.png` | Simulation-related visualization used to document the design's behavior. |

These images help document the design before and around the synthesis stage.

### 3.2 Synthesis Statistics

| File | Purpose |
|---|---|
| `stats.png` | Captured synthesis statistics. |
| `stat_after_reduction.png` | Captured statistics following the documented reduction stage. |

Synthesis statistics provide information about the synthesized design and help document changes in the resulting logic representation.

### 3.3 Synthesized Netlists

| File | Purpose |
|---|---|
| `vsdbabysoc_net.png` | Visualization of the synthesized BabySoC-related netlist. |
| `rvmyth_net.png` | Netlist visualization associated with the RVMYTH design. |
| `rvmyth_net2.png` | Additional RVMYTH netlist visualization. |
| `clk_gate_babysoc_net.png` | Netlist visualization associated with the BabySoC clock-gating logic. |

Netlist visualizations help illustrate how the design is represented as interconnected logic elements after synthesis.

### What This Section Demonstrates

- Documentation of the RTL simulation and synthesis stages.
- Inspection of synthesized design representations.
- Examination of synthesis statistics.
- Visualization of design hierarchy and logic connectivity.
- Documentation of clock-gating-related logic.

**Explore the complete section:** [RTL Logic Synthesis README](BabySoC_RTL_Logic_Synthesis/README.md)

---

## 🏗️ 4. Physical Design

**Directory:** `Lab_PD_VSDBabySoC/`

This section documents the physical-design implementation and timing-analysis work associated with BabySoC.

It contains floorplanning, placement, clock-tree synthesis, routing, timing reports, and supporting implementation images.

### 4.1 Floorplanning

**Representative files:**
- `Floorplan.png`
- `pll_floorplan.png`

Floorplanning establishes the initial physical organization of the design within the available chip area.

The captured images provide visual documentation of the floorplan and associated implementation views.

### 4.2 Placement

**Representative files:**
- `Placement.png`
- `Place.png`
- `Place_Report.png`

Placement concerns the positioning of cells within the physical design.

These files document placement-related views and reports generated during the implementation workflow.

### 4.3 Clock-Tree Synthesis

**Representative files:**
- `CTS.png`
- `Clk_nets.png`
- `cts_rpt.png`

Clock-tree synthesis develops the clock distribution network used to deliver the clock signal throughout the design.

The associated images and report provide supporting documentation for the clock-tree synthesis stage.

### 4.4 Routing and Implementation

**Representative files:**
- `POST_ROUTE.png`
- `POST_ROUTE2.png`

These images document the post-route implementation views captured during the physical-design workflow.

Routing connects the placed cells through the physical interconnect network. Post-route analysis helps examine the implemented design and its resulting characteristics.

### 4.5 Physical-Design Analysis and Reports

**Representative files:**
- `Heatmap.png`
- `Heatmap2.png`
- `STA_VSDBabySoC.png`
- `STA_EXEC_ALL16FILES.png`

Additional timing-related files include:

- `WNS_PLOT.png`
- `TNS_PLOT.png`
- `WORST_REPORTED_SETUP_SLACK.png`
- `WORST_REPORTED_HOLD_SLACK.png`

These files provide supporting visual evidence for implementation analysis and static timing analysis.

The exact meaning and interpretation of individual plots should be taken from their associated reports and the detailed physical-design documentation.

**Explore the complete section:** [Physical Design README](Lab_PD_VSDBabySoC/README.md)

---

## 🔄 5. Design Flow

The project is organized around the following conceptual workflow:

```text
        RTL Design
            |
            v
     RTL Simulation
            |
            v
     Logic Synthesis
            |
            v
   Synthesized Netlist
            |
            v
      Floorplanning
            |
            v
        Placement
            |
            v
 Clock-Tree Synthesis
            |
            v
         Routing
            |
            v
 Static Timing Analysis
            |
            v
   Timing Evaluation
```

Each stage builds upon the representation produced by the preceding stage.

The synthesis section focuses on simulation-related documentation, netlists, and synthesis statistics, while the physical-design section focuses on implementation stages and timing-analysis evidence.

*This diagram illustrates the general design-flow progression. It does not imply that every listed stage or result was independently reproduced or verified from the files in this repository.*

---

## ⏱️ 6. Static Timing Analysis

Static Timing Analysis evaluates timing constraints and signal-propagation paths without requiring functional simulation of every possible input sequence.

The physical-design directory contains timing-related plots, reports, and supporting screenshots associated with the documented STA workflow.

### Timing Concepts

| Metric | Description |
|---|---|
| Setup Slack | Measures the timing margin available to satisfy setup-time requirements. |
| Hold Slack | Measures the timing margin available to satisfy hold-time requirements. |
| WNS — Worst Negative Slack | Represents the worst setup or hold slack, depending on the report and analysis being considered. |
| TNS — Total Negative Slack | Represents the aggregate negative slack under the report's defined analysis conditions. |
| PVT Corners | Represent process, voltage, and temperature conditions used to evaluate design behavior. |

### Timing Documentation

The following files provide visual documentation of the timing-analysis work:

- `STA_VSDBabySoC.png`
- `STA_EXEC_ALL16FILES.png`
- `WNS_PLOT.png`
- `TNS_PLOT.png`
- `WORST_REPORTED_SETUP_SLACK.png`
- `WORST_REPORTED_HOLD_SLACK.png`

The reported timing values and their interpretation should be read alongside the corresponding reports and the detailed physical-design README.

**Important:** A captured timing path or slack value should not automatically be interpreted as the design-wide WNS or TNS. Those metrics must be established from the corresponding complete timing reports and their analysis conditions.

---

## 🧠 7. Key Learning Outcomes

This work provides an opportunity to study and document the relationship between digital logic design and physical implementation.

### RTL and Synthesis

- Understanding the transition from RTL-oriented design representations to synthesized netlists.
- Inspecting design connectivity through netlist visualizations.
- Reading and organizing synthesis statistics.
- Examining clock-gating-related logic.

### Physical Design

- Understanding the purpose of floorplanning and placement.
- Studying clock distribution through clock-tree synthesis.
- Examining post-route implementation views.
- Organizing implementation screenshots and reports.

### Timing Analysis

- Understanding setup and hold slack.
- Distinguishing WNS from TNS.
- Examining timing results under different PVT conditions.
- Interpreting timing evidence in the context of the reports that generated it.

---

## 📚 8. Project Documentation

Detailed documentation is maintained separately for each major section.

| Documentation | Description |
|---|---|
| [RTL Logic Synthesis](BabySoC_RTL_Logic_Synthesis/README.md) | Simulation-related documentation, synthesis statistics, netlists, and supporting visualizations. |
| [Physical Design](Lab_PD_VSDBabySoC/README.md) | Floorplanning, placement, clock-tree synthesis, routing, implementation reports, and timing-analysis documentation. |

Keeping the documentation separated makes it easier to explore the individual stages while preserving a unified overview of the complete project.

---

## 👨‍💻 9. Author

**Ganesh Koushal**

Electronics and Communication Engineering (ECE)  
VLSI Design and Verification | Digital Design | RTL-to-GDSII Design Flow

### Repository

Explore the complete workshop repository:

**[VSD-WorkShop — GitHub Repository](https://github.com/Ganesh-Koushal/VSD-WorkShop)**

---

*This repository documents my learning and project work related to BabySoC RTL simulation, logic synthesis, physical design, and timing analysis as part of the VLSI System Design workshop.*
