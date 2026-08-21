# Day 3 – Combinational and Sequential Logic Optimization

## Objective

Day 3 shifts focus from just getting a design to simulate/synthesize correctly, to making it efficient. This session covers the core optimization techniques synthesis tools apply — constant propagation, state optimization, cloning, and retiming — followed by six hands-on labs that show these effects on real Verilog code.

---

## Contents

1. [Constant Propagation](#1-constant-propagation)
2. [State Optimization](#2-state-optimization)
3. [Cloning](#3-cloning)
4. [Retiming](#4-retiming)
5. [Labs](#5-labs)
6. [Conclusion](#6-conclusion)

---

## 1. Constant Propagation

When a signal in the design can be proven to always hold a fixed value, the synthesis tool substitutes that constant directly into the logic instead of routing it through actual gates. This lets downstream optimization collapse redundant logic that depended on that signal.

**Why it matters**
- Smaller, simpler netlist
- Fewer gates → less area and typically less delay
- Removes logic that would otherwise do nothing useful

<img width="526" height="298" alt="Constant propagation example" src="https://github.com/user-attachments/assets/3b90652b-3e89-4b8c-a808-8082c26e74e0" />

---

## 2. State Optimization

Applies specifically to FSMs (finite state machines) — the goal is to represent the same functional behavior using fewer states and more efficient encoding.

**Techniques involved**
- **State merging** — combining states that are functionally equivalent
- **Re-encoding** — choosing state bit patterns that minimize the next-state/output logic
- **Logic minimization** — simplifying the resulting Boolean equations
- **Power-aware techniques** — e.g. clock gating unused states to cut dynamic power

---

## 3. Cloning

Cloning duplicates a logic cell or module — usually one sitting on a timing-critical path — so that its output load can be split across two instances instead of one, reducing fanout delay.

**General flow**
1. Identify a critical-path cell via timing analysis
2. Duplicate that cell/module
3. Redistribute the downstream connections across the original and the clone
4. Re-place and re-route
5. Re-check timing/power to confirm the improvement actually helped

<img width="462" height="286" alt="Cloning example" src="https://github.com/user-attachments/assets/ae07617f-c1fc-41c4-b3a4-309adeebed08" />

---

## 4. Retiming

Retiming repositions flip-flops within a circuit to balance the combinational delay between register stages — without changing the circuit's overall input-output behavior.

**General flow**
1. Model the circuit as a directed graph of logic and registers
2. Move registers across combinational logic to even out path delays
3. Check that timing constraints and functional equivalence still hold
4. Settle on the register placement that minimizes the achievable clock period

---

## 5. Labs

### Lab 1

```verilog
module opt_check (input a, input b, output y);
    assign y = a ? b : 0;
endmodule
```

**Behavior:** `y = b` when `a = 1`, and `y = 0` when `a = 0` — a simple AND-like mux reduction.

Synthesis steps follow the same flow as Day 1, with one addition — an `opt_clean -purge` step inserted between `abc -liberty` and `synth -top` to strip out redundant/unused logic left over after mapping.

<img width="1920" height="983" alt="Opt_check M3" src="https://github.com/user-attachments/assets/8f37f868-d786-45aa-b279-13f6f3d62ede" />

---

### Lab 2

```verilog
module opt_check2 (input a, input b, output y);
    assign y = a ? 1 : b;
endmodule
```

**Behavior:** Acts as a 2:1 mux — `y = 1` when `a = 1`, otherwise `y = b`.

<img width="955" height="970" alt="Opt_check2 M3" src="https://github.com/user-attachments/assets/7036b972-32c3-416e-8c3d-682e6317b919" />

---

### Lab 3

```verilog
module opt_check3 (input a, input b, output y);
    assign y = a ? 1 : b;
endmodule
```

**Behavior:** Same logical structure as Lab 2 — reinforces how the synthesis tool reduces this ternary pattern to a small mux-like gate structure.

<img width="955" height="970" alt="Opt_check3 M3" src="https://github.com/user-attachments/assets/d4e115f8-3446-4783-b176-6724409c3a2e" />

---

### Lab 4

```verilog
module opt_check4 (input a, input b, input c, output y);
    assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

**Behavior:** Nested ternary logic with three inputs. Tracing through the cases:
- When `a = 1`: inner condition depends on `b`, but since `a = 1` the `a & c` term reduces to just `c` — so `y = c` regardless of `b`.
- When `a = 0`: `y = !c`.

This simplifies to `y = a ? c : !c` — a good example of how synthesis tools collapse seemingly complex nested conditionals once the redundant dependency on `b` is identified.

<img width="955" height="970" alt="opt_check4 M3" src="https://github.com/user-attachments/assets/f64ad986-c0ab-43ab-9a61-d3f71437be6f" />

---

### Lab 5

```verilog
module dff_const1 (input clk, input reset, output reg q);
    always @ (posedge clk, posedge reset)
    begin
        if (reset)
            q <= 1'b0;
        else
            q <= 1'b1;
    end
endmodule
```

**Behavior:** A D flip-flop with asynchronous reset to `0`; once out of reset, it always loads a constant `1` on every clock edge (there's no data input feeding `q`).

<img width="955" height="970" alt="dff_const1 M3" src="https://github.com/user-attachments/assets/d5390982-1325-4678-808c-cfb07ee28bb8" />

---

### Lab 6

```verilog
module dff_const2 (input clk, input reset, output reg q);
    always @ (posedge clk, posedge reset)
    begin
        if (reset)
            q <= 1'b1;
        else
            q <= 1'b1;
    end
endmodule
```

**Behavior:** `q` is set to `1` in both the reset and non-reset branches — meaning the flip-flop is functionally redundant, so synthesis optimizes it down to a tied-high constant, removing the actual flip-flop entirely.

<img width="955" height="970" alt="dff_const2 M3" src="https://github.com/user-attachments/assets/2564d1c4-c926-4937-9347-228eac369013" />
<img width="955" height="970" alt="dff_const3 M3" src="https://github.com/user-attachments/assets/a844e671-5be0-41fe-ad76-1361031698d4" />
<img width="955" height="970" alt="dff_const4 M3" src="https://github.com/user-attachments/assets/4f7e57d1-4cd3-40f8-af9f-083a76c97955" />
<img width="955" height="970" alt="dff_const5 M3" src="https://github.com/user-attachments/assets/358ff89a-d1cb-4376-bce1-20584a5b82ed" />

---

## 6. Conclusion

Day 3 was less about new simulation/synthesis commands and more about reading what synthesis actually does to RTL once optimization is enabled. The labs progressed from simple constant-driven mux reduction (Labs 1–4) to flip-flops that either load hardcoded values or are entirely redundant (Labs 5–6) — a good reminder that not all "sequential-looking" RTL actually needs sequential hardware once the tool sees through the logic. Understanding this is directly useful going forward: knowing when a design is being over-optimized (or under-optimized) is a core skill for both RTL design and verification.
