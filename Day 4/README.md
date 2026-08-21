# Day 4 – Gate-Level Simulation, Blocking vs. Non-Blocking, and Synthesis-Simulation Mismatch

## Objective

Day 4 covers three things that tend to trip people up once designs move past toy examples: what Gate-Level Simulation (GLS) actually verifies, why RTL simulation and gate-level results can disagree, and the real difference between blocking and non-blocking assignments in Verilog — including a few classic coding mistakes that cause exactly this kind of mismatch.

---

## Contents

1. [Gate-Level Simulation (GLS)](#1-gate-level-simulation-gls)
2. [Synthesis-Simulation Mismatch](#2-synthesis-simulation-mismatch)
3. [Blocking vs. Non-Blocking Assignments](#3-blocking-vs-non-blocking-assignments)
4. [Labs](#4-labs)
5. [Conclusion](#5-conclusion)

---

## 1. Gate-Level Simulation (GLS)

GLS means simulating the **synthesized gate-level netlist** instead of the original RTL — essentially checking whether the actual gates the tool produced behave the same way the RTL did, and whether they meet timing.

**What it verifies**
- Functional correctness of the netlist itself
- Timing behavior, when run with real delay annotations (SDF)
- Power estimates at the gate level
- DFT structures like scan chains, where applicable

**When it's run**
- Right after synthesis, before the design moves into physical design/layout — catching problems here is far cheaper than catching them after place-and-route.

**Two flavors**
- **Functional GLS** — logic-only check, usually with zero or unit delays; purely about correctness.
- **Timing GLS** — uses annotated real-world delays to catch setup/hold violations and similar timing issues.

---

## 2. Synthesis-Simulation Mismatch

This is what happens when RTL simulation (pre-synthesis) and gate-level simulation (post-synthesis) — or real hardware — don't agree, even though they're supposedly implementing the same design.

**Common causes**
- **Non-synthesizable constructs** — delays (`#`), `initial` blocks, or other simulation-only code that synthesis simply ignores or interprets differently
- **Incomplete/ambiguous RTL** — missing `else` branches, incomplete sensitivity lists — these can cause a simulator and a synthesis tool to infer different hardware from the same code
- **Tool-specific interpretation** — different tools resolving ambiguous RTL constructs differently

The fix isn't really a "fix" so much as a habit: write clean, fully-specified, synthesizable RTL from the start, so there's no ambiguity left for a tool to resolve differently.

---

## 3. Blocking vs. Non-Blocking Assignments

Verilog gives you two ways to assign values inside a procedural block, and picking the wrong one for the situation is one of the most common sources of synthesis-simulation mismatch.

### Blocking (`=`)

Executes immediately, in program order — each statement finishes before the next one starts.

```verilog
always @ (*)
  y = a & b;
```

Appropriate for **combinational logic** (`always @(*)`), where there's no real notion of "state" being held between evaluations.

### Non-Blocking (`<=`)

Scheduled rather than immediate — all right-hand sides are evaluated first, and the actual assignments happen together at the end of the current time step.

```verilog
always @ (posedge clk)
  q <= d;
```

Appropriate for **sequential logic** (`always @(posedge clk)`), which is exactly how real flip-flops behave — all of them update on the same clock edge, based on values sampled just before that edge.

### Quick Comparison

| | Blocking (`=`) | Non-Blocking (`<=`) |
|---|---|---|
| Execution | Immediate, in code order | Scheduled, applied at end of time step |
| Best suited for | Combinational logic | Sequential logic |
| Typical inference | Gates | Flip-flops / registers |
| Risk if misused | Simulation-synthesis mismatch, incorrect sequential behavior | Unnecessary latches/registers, or race-prone combinational logic |

---

## 4. Labs

### Lab 1 — Ternary Operator MUX

```verilog
module ternary_operator_mux (input i0, input i1, input sel, output y);
    assign y = sel ? i1 : i0;
endmodule
```

**Behavior:** `y = i1` when `sel = 1`, otherwise `y = i0` — a clean, unambiguous 2:1 mux written with a continuous assignment.

📷 <img width="955" height="970" alt="T_op_sim" src="https://github.com/user-attachments/assets/e55d5101-1938-4123-aef0-5fd0cf88e1f1" />


---

### Lab 2 — Synthesizing the MUX

Standard Yosys synthesis flow (same as Day 1) applied to `ternary_operator_mux`.

📷 <img width="1842" height="854" alt="TO_MUX M4" src="https://github.com/user-attachments/assets/bf447d7b-ac3b-436c-9401-3d354b2a019e" />


---

### Lab 3 — GLS of the MUX

Running gate-level simulation on the synthesized netlist requires including the Sky130 primitive and standard-cell models alongside the netlist and testbench:

```bash
iverilog /path/to/primitives.v /path/to/sky130_fd_sc_hd.v ternary_operator_mux.v testbench.v
```

Since the original RTL here was clean and unambiguous, GLS results match the RTL simulation exactly.

📷 <img width="1838" height="799" alt="t_op_mux_sim2" src="https://github.com/user-attachments/assets/51da278b-fd60-4e30-9b8e-cc430880363b" />



---

### Lab 4 — Bad MUX (Common Pitfalls)

```verilog
module bad_mux (input i0, input i1, input sel, output reg y);
    always @ (sel) begin
        if (sel)
            y <= i1;
        else
            y <= i0;
    end
endmodule
```

**Problems with this code:**
- **Incomplete sensitivity list** — only `sel` is listed, so `i0`/`i1` changing won't retrigger the block in simulation, even though real combinational hardware would respond to them.
- **Non-blocking assignment used for combinational logic** — `<=` here can cause the simulated behavior to lag behind what the synthesized gates will actually do.

**Corrected version:**

```verilog
always @ (*) begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

`always @(*)` automatically includes every signal read inside the block in the sensitivity list, and blocking assignments keep behavior consistent with how combinational logic actually works.

📷 <img width="955" height="970" alt="bad_mux_sim" src="https://github.com/user-attachments/assets/900a4b78-1e96-47a3-9abe-7163ff448f98" />


---

### Lab 5 — GLS of the Bad MUX

Running GLS on the uncorrected `bad_mux` typically surfaces mismatches or simulator warnings, directly caused by the two issues above — a good concrete demonstration of *why* those coding habits matter, rather than just an abstract rule.

📷 <img width="1920" height="966" alt="image" src="https://github.com/user-attachments/assets/f9123df2-c1c4-4f28-bf33-eaf3a424b83c" />


---

### Lab 6 — Blocking Assignment Caveat

```verilog
module blocking_caveat (input a, input b, input c, output reg d);
    reg x;
    always @ (*) begin
        d = x & c;
        x = a | b;
    end
endmodule
```

**The bug:** `d` is computed using `x` *before* `x` gets its new value in this same block — so `d` ends up using a stale value of `x` from the previous evaluation, not the one just computed from `a`/`b`. Since blocking assignments execute strictly top-to-bottom, statement order matters a lot here.

**Corrected version — compute dependencies first:**

```verilog
always @ (*) begin
    x = a | b;
    d = x & c;
end
```

📷<img width="955" height="970" alt="blocking_caveat_sim m4" src="https://github.com/user-attachments/assets/9bcd57c6-8d69-4466-9290-565cc02c0177" />

 <img width="955" height="970" alt="blocking_caveat_sim2 m4" src="https://github.com/user-attachments/assets/a704a91c-5932-44c1-ae2f-a804fea3cb19" />

---

### Lab 7 — Synthesizing the Corrected Module

Synthesizing the fixed `blocking_caveat` module and comparing the resulting netlist/behavior against the original buggy version.

📷 <img width="955" height="970" alt="blocking_caveat_netlist m4" src="https://github.com/user-attachments/assets/c7f5ccfe-3413-4610-93cf-40a1d180dc32" />


---

## 5. Conclusion

Day 4 was really about closing the gap between "the RTL simulates fine" and "the hardware will actually behave the way I think it will." GLS is the checkpoint that catches the difference — and Labs 4–6 show exactly how easy it is to introduce a mismatch through small, easy-to-miss habits: an incomplete sensitivity list, the wrong assignment type, or statements in the wrong order. The underlying lesson carries forward into every RTL design from here on: write complete, unambiguous, and correctly-ordered logic, and always sanity-check RTL behavior against the gate-level netlist rather than assuming synthesis "just works."
