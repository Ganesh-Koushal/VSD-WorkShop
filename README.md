# RTL Design Workshop

A structured, hands-on log of my RTL design and verification learning journey — covering Verilog fundamentals, functional simulation, and logic synthesis using open-source EDA tools.

This repository documents each session's concepts, lab work, and results as I build toward proficiency in digital design and verification, with the long-term goal of specializing in RTL Design / Digital Design Verification (DV).

---

## 📚 Workshop Progress

| # | Topic | Status |
|---|-------|--------|
| [Day 1](./Day_1) | Verilog RTL Design & Functional Simulation | ✅ Completed |
| [Day 2](./Day_2) | Timing Libraries & Synthesis Strategies | ✅ Completed |
| [Day 3](./Day_3) | Combinational & Sequential Optimization | ✅ Completed |
| [Day 4](./Day_4) | Gate-Level Simulation, Blocking vs. Non-Blocking & Synthesis-Simulation Mismatch | ✅ Completed |
| [Day 5](./Day_5) | Synthesis Optimization — If-Else, Case, For Loops & Generate Blocks | ✅ Completed |

*(New sessions are added as the coursework progresses.)*

---

## 📂 Repository Structure

```
RTL_Workshop
│
├── README.md
│
├── Day_1
│   ├── README.md
│   ├── good_mux.v
│   ├── tb_good_mux.v
│   └── good_mux_waveform.png
│
├── Day_2
│   ├── README.md
│   ├── SKY1300DK.png
│   ├── Hierarchial Modules.png
│   ├── Sub_module Ex.png
│   ├── Flatten Netlist.png
│   ├── Complete Netlist.png
│   ├── Async FF Netlist.png
│   └── DFF_waveform.png
│
├── Day_3
│   ├── README.md
│   ├── Opt_check M3.png
│   ├── Opt_check2 M3.png
│   ├── Opt_check3 M3.png
│   ├── opt_check4 M3.png
│   ├── Seq Optimisations.png
│   ├── counter_opt.png
│   ├── counter_opt1.png
│   ├── seq_opt_sim.png
│   ├── seq_opt_sim2.png
│   ├── dff_const1 M3.png
│   ├── dff_const2 M3.png
│   ├── dff_const3 M3.png
│   ├── dff_const3_sim M3.png
│   ├── dff_const4 M3.png
│   └── dff_const5 M3.png
│
├── Day_4
│   ├── README.md
│   ├── TO_MUX M4.png
│   ├── T_op_sim.png
│   ├── t_op_mux_sim2.png
│   ├── bad_mux_sim.png
│   ├── bad_mux_sim_gls m4.png
│   ├── blocking_caveat_sim m4.png
│   ├── blocking_caveat_sim2 m4.png
│   └── blocking_caveat_netlist m4.png
│
└── Day_5
    ├── README.md
    ├── Comp_case_sim.png
    ├── Partial_case_assign.png
    ├── bad_case_net.png
    ├── bad_case_sim.png
    ├── comp_case_syn.png
    ├── demux_case_net.png
    ├── demux_case_sim.png
    ├── incom_if_net.png
    ├── incomp_case_net.png
    ├── incomp_case_sim.png
    ├── incomp_if2_net.png
    ├── incomp_if2_sim.png
    ├── incomp_if_sim.png
    ├── m5 rca_sim.png
    └── mux_generate_sim.png
```

Each session folder contains its own README with the concepts covered, the lab exercises performed, the Verilog source files used, and simulation/synthesis result screenshots.

---

## 🧰 Tools & Technologies

- **Verilog HDL** — RTL design
- **Icarus Verilog (iverilog)** — Functional simulation
- **GTKWave** — Waveform analysis
- **Yosys** — Logic synthesis
- **Sky130 PDK** — Open-source standard cell library
- **Linux (Ubuntu)** — Development environment

---

## 🎯 Purpose

This repository serves as both a personal learning record and a portfolio of practical RTL design skills — from writing synthesizable Verilog to verifying functional correctness and understanding how RTL maps to real silicon-ready gates. It reflects an ongoing effort to build a strong foundation for a career in RTL Design and Verification.

---

## 👤 Author

**K. Venkata Ganesh Koushal**
B.Tech ECE (VLSI Track)
