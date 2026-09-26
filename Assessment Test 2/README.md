# Assessment Test 2 – Multi-Corner Static Timing Analysis: WNS & TNS on VSDBabySoC

## Objective

This assessment runs full Static Timing Analysis (STA) on the synthesized **VSDBabySoC** design across **15 SKY130 PVT (Process-Voltage-Temperature) library corners** using OpenSTA, then studies two of the most important sign-off timing metrics — **Worst Negative Slack (WNS)** and **Total Negative Slack (TNS)** — to understand exactly how much the design's timing margin shifts across process extremes, rather than trusting a single "typical corner" result.

---

## Contents

1. [Why Multi-Corner STA Matters](#1-why-multi-corner-sta-matters)
2. [Understanding Slack, WNS, and TNS](#2-understanding-slack-wns-and-tns)
3. [Automating STA Across All Corners](#3-automating-sta-across-all-corners)
4. [Results — All 15 Corners](#4-results--all-15-corners)
5. [Reading the Results by Corner Type](#5-reading-the-results-by-corner-type)
6. [WNS vs. TNS — Why Both Matter](#6-wns-vs-tns--why-both-matter)
7. [Overall Findings](#7-overall-findings)
8. [Conclusion](#8-conclusion)

---

## 1. Why Multi-Corner STA Matters

A chip doesn't get manufactured at one exact, fixed process/voltage/temperature point — real silicon varies, and a design has to work correctly across the full range the foundry guarantees. SKY130 models this with named corners like `tt` (typical-typical), `ss` (slow-slow), and `ff` (fast-fast), each combined with specific voltage and temperature conditions.

Checking timing against only the typical corner would completely miss how the design behaves at its actual worst case — which is exactly the gap this assessment closes by running STA across all 15 available corners in one pass.

---

## 2. Understanding Slack, WNS, and TNS

**Slack** on any timing path is the margin between when a signal is required to arrive and when it actually does:

```
Slack = Required Time − Arrival Time
```

A positive slack means the path meets timing with margin to spare; a negative slack means it's a genuine violation.

### Worst Negative Slack (WNS)

WNS is simply the single most negative slack value found across every analyzed path:

```
WNS = min(Slack)
```

- `WNS ≥ 0` → no setup violation reported at all
- `WNS < 0` → at least one path is failing, and the magnitude tells you by how much

A `WNS` of `−62.56 ns`, for example, means the single worst path in the design misses its timing requirement by 62.56 ns.

### Total Negative Slack (TNS)

TNS sums up the negative slack across **every** violating path, not just the worst one:

```
TNS = Σ Slack_i   (summed only over paths where Slack_i < 0)
```

- `TNS = 0` → no violations anywhere
- A large negative TNS means either many paths are violating, or the violations are individually severe (or both) — it's important not to read a number like `−13,957.59 ns` as "one path is over 13,000 ns late." It's the accumulated total across many failing paths, not a single path's delay.

**In short:** WNS tells you how bad the *worst single path* is; TNS tells you how *widespread* the violations are across the whole design.

---

## 3. Automating STA Across All Corners

Rather than manually re-running OpenSTA once per corner, a single Tcl script loops through all 15 libraries, reading each corner's Liberty file plus the shared macro libraries (`avsddac.lib`, `avsdpll.lib`), linking the design, applying the SDC constraints, and reporting checks, WNS, and TNS for that corner — before moving to the next:

```tcl
set libs {
    sky130_fd_sc_hd__tt_025C_1v80.lib
    sky130_fd_sc_hd__tt_100C_1v80.lib

    sky130_fd_sc_hd__ss_n40C_1v28.lib
    sky130_fd_sc_hd__ss_n40C_1v35.lib
    sky130_fd_sc_hd__ss_n40C_1v40.lib
    sky130_fd_sc_hd__ss_n40C_1v44.lib
    sky130_fd_sc_hd__ss_n40C_1v60.lib
    sky130_fd_sc_hd__ss_n40C_1v76.lib

    sky130_fd_sc_hd__ss_100C_1v40.lib
    sky130_fd_sc_hd__ss_100C_1v60.lib

    sky130_fd_sc_hd__ff_n40C_1v56.lib
    sky130_fd_sc_hd__ff_n40C_1v65.lib
    sky130_fd_sc_hd__ff_n40C_1v76.lib

    sky130_fd_sc_hd__ff_100C_1v65.lib
    sky130_fd_sc_hd__ff_100C_1v95.lib
}

foreach lib $libs {
    read_liberty -min /data/.../lib/$lib
    read_liberty -max /data/.../lib/$lib

    read_liberty -min /data/.../lib/avsddac.lib
    read_liberty -max /data/.../lib/avsddac.lib
    read_liberty -min /data/.../lib/avsdpll.lib
    read_liberty -max /data/.../lib/avsdpll.lib

    read_verilog /data/.../vsdbabysoc.synth.v
    link_design vsdbabysoc
    read_sdc /data/.../sdc/vsdbabysoc_synthesis.sdc

    report_checks -path_delay min_max -fields {nets cap slew input_pins fanout}
    report_wns
    report_tns
}
```

Run inside the OpenSTA Docker container:

```bash
sudo docker run -i -v $HOME:/data opensta \
  /data/Desktop/work/tools/openlane_working_dir/OpenSTA/vsdbabysoc/run_final.tcl
```

One important detail worth calling out: OpenSTA's actual commands are `report_wns` and `report_tns` — not `run_wns`/`run_tns` as might be guessed by analogy to `run_synthesis`/`run_floorplan` from the OpenLane side of the flow.

<img width="800" alt="Detailed OpenSTA report_checks output across corners" src="./TND%26WNS%20PLOT.png" />

---

## 4. Results — All 15 Corners

| Library Corner | WNS (ns) | TNS (ns) | Result |
|---|---|---|---|
| `sky130_fd_sc_hd__tt_025C_1v80.lib` | 0.00 | 0.00 | No violation |
| `sky130_fd_sc_hd__tt_100C_1v80.lib` | 0.00 | 0.00 | No violation |
| `sky130_fd_sc_hd__ss_n40C_1v28.lib` | **−62.56** | **−13957.59** | Violation |
| `sky130_fd_sc_hd__ss_n40C_1v35.lib` | −39.70 | −7993.79 | Violation |
| `sky130_fd_sc_hd__ss_n40C_1v40.lib` | −29.69 | −5466.48 | Violation |
| `sky130_fd_sc_hd__ss_n40C_1v44.lib` | −23.91 | −4107.19 | Violation |
| `sky130_fd_sc_hd__ss_n40C_1v60.lib` | −10.61 | −1355.80 | Violation |
| `sky130_fd_sc_hd__ss_n40C_1v76.lib` | −4.38 | −367.93 | Violation |
| `sky130_fd_sc_hd__ss_100C_1v40.lib` | −17.12 | −2599.51 | Violation |
| `sky130_fd_sc_hd__ss_100C_1v60.lib` | −7.87 | −892.70 | Violation |
| `sky130_fd_sc_hd__ff_n40C_1v56.lib` | 0.00 | 0.00 | No violation |
| `sky130_fd_sc_hd__ff_n40C_1v65.lib` | 0.00 | 0.00 | No violation |
| `sky130_fd_sc_hd__ff_n40C_1v76.lib` | 0.00 | 0.00 | No violation |
| `sky130_fd_sc_hd__ff_100C_1v65.lib` | 0.00 | 0.00 | No violation |
| `sky130_fd_sc_hd__ff_100C_1v95.lib` | 0.00 | 0.00 | No violation |

<img width="800" alt="WNS and TNS summary across all corners" src="./TNS%26WNS%20SUMMARY.png" />
<img width="800" alt="WNS and TNS values table" src="./TNS%26WNS%20Values.png" />

---

## 5. Reading the Results by Corner Type

**TT (Typical-Typical) corners** — both `tt_025C_1v80` and `tt_100C_1v80` came back completely clean, `WNS = TNS = 0.00 ns`. This is the "nominal" behavior the design was originally synthesized and timed against, so a clean result here is expected rather than notable on its own.

**FF (Fast-Fast) corners** — all five FF libraries also reported zero negative slack. This makes intuitive sense for a *setup*-focused check: faster transistors mean data arrives sooner, which only helps setup timing (though a fast corner is exactly where *hold* violations would be more likely to show up instead — a good reminder that a clean setup result doesn't automatically mean a corner is fully timing-clean in every sense).

**SS (Slow-Slow) corners** — every single violation in this assessment came from an SS corner. The worst case, `ss_n40C_1v28`, produced the most negative WNS (`−62.56 ns`) and TNS (`−13,957.59 ns`) of all 15 corners. Looking at the SS results together, there's a clear trend:

```
ss_n40C_1v28 → WNS −62.56 ns   (lowest voltage → worst timing)
ss_n40C_1v35 → WNS −39.70 ns
ss_n40C_1v40 → WNS −29.69 ns
ss_n40C_1v44 → WNS −23.91 ns
ss_n40C_1v60 → WNS −10.61 ns
ss_n40C_1v76 → WNS  −4.38 ns   (highest voltage → best timing, still SS)
```

As supply voltage increases within the SS corner family, timing steadily improves — which lines up with basic transistor behavior: lower voltage means slower switching, which directly worsens setup timing on a slow-process corner.

---

## 6. WNS vs. TNS — Why Both Matter

| | WNS | TNS |
|---|---|---|
| Answers | "How bad is the single worst path?" | "How much total violation exists across the whole design?" |
| For this design | −62.56 ns (worst corner) | −13,957.59 ns (worst corner) |
| Zero means | No path violates at all | No accumulated violation anywhere |
| More negative means | A more severe single failure | A more widespread timing problem |

The two metrics are genuinely independent pieces of information — a design could theoretically have a small WNS (one mildly-late path) but a huge TNS (thousands of paths each slightly late), or the reverse (one catastrophically late path, but everything else clean). Real timing closure work needs both numbers together to actually understand what's going on.

---

## 7. Overall Findings

- **15** total library corners analyzed
- **8** corners reported timing violations (all SS)
- **7** corners reported zero negative slack (all TT and FF)
- **Worst WNS:** −62.56 ns, at `ss_n40C_1v28`
- **Worst TNS:** −13,957.59 ns, at the same corner

The pattern is unambiguous: every violation traces back to the slow-process, low-voltage region of the PVT space — exactly where real silicon is expected to be at its electrical worst.

---

## 8. Conclusion

This assessment made one thing very concrete: a design's timing "passing" or "failing" isn't a single yes/no answer — it's entirely dependent on *which* corner you check. VSDBabySoC came back completely clean at every TT and FF corner tested, yet failed timing at every single SS corner, with the violation growing steadily worse as supply voltage dropped. Extracting both WNS and TNS (rather than just one) is what actually shows the full picture — WNS pinpointed the single worst offending path, while TNS quantified just how widespread the problem really was at that corner. This is exactly the kind of analysis real sign-off timing closure depends on: checking every guaranteed operating condition, not just the convenient nominal one.
