# CMOS LDO Voltage Regulator with Transistorized Adaptive Transient Boost

> Complete transistor-level CMOS LDO in LTspice — 3.302V regulated output from 5V supply with a novel fully-transistorized adaptive transient detector that automatically reduces output droop by 1.6× with zero static power overhead.

---

## Key Results

| Metric | Without Boost | With Adaptive Boost | Improvement |
|---|---|---|---|
| Output Voltage | 3.302 V | 3.302 V | — |
| Peak Droop | 360 mV | 222 mV | **1.6×** |
| Recovery Time | ~12 µs | ~7 µs | **1.7×** |
| Static Power Overhead | — | **0%** | — |
| Load Step | 1 mA → 100 mA | 1 mA → 100 mA | — |
| Supply | 5 V | 5 V | — |

---

## What Makes This Novel

Most student LDO projects use behavioral op-amp models. This project:

1. **Builds the entire error amplifier at transistor level** — differential pair, PMOS mirror load, second stage, Miller compensation — all from individual MOSFETs
2. **Implements a fully-transistorized adaptive detector** — no digital trigger, no ideal current sources — just 3 transistors and 2 passives that automatically detect droops and activate boost
3. **Achieves zero static power overhead** — the boost block is completely OFF at steady state, consuming no quiescent current until a load transient occurs

---

## Circuit Architecture

```
VDD (5V)
   │
[MP1 — PMOS Pass Transistor W=200µm/L=1µm]
   │
VOUT (3.302V) ────────────────────────────────┐
   │                                           │
[Two-Stage Error Amplifier]             [Output Network]
   ├── M2/M3 NMOS diff pair (50µA tail)    C1=1µF
   ├── M4/M5 PMOS mirror load              I_load (pulse)
   ├── M6/M7 second stage                  R1=200k
   └── R3/C2 Miller compensation           R2=114k → VFEEDBACK
   │
[VREF = 1.2V]

[Adaptive Boost Block — THE NOVEL PART]
   C_DETECT (1pF): VOUT → DETECT node
   RDOWN (47kΩ):   DETECT → VDD (pullup)
   M8 (PMOS):      Gate=DETECT, Source=VDD, Drain=AMPOUT
   MBOOST (NMOS):  Gate=AMPOUT, Drain=VGATE, Source=GND
```

## How the Detector Works

**Steady State:**
```
DETECT = VDD → Vgs(M8) = 0V → M8 OFF
AMPOUT = 0V → MBOOST OFF
Zero boost current, zero overhead ✓
```

**During Load Transient:**
```
VOUT droops → C_DETECT couples drop → DETECT falls below VDD
Vgs(M8) goes negative → M8 turns ON → AMPOUT rises
MBOOST turns ON → pulls VGATE down → MP1 turns ON harder
VOUT recovers faster ✓
DETECT pulled back to VDD through RDOWN → boost turns OFF automatically ✓
```

---

## Repository Structure

```
📁 LDO_Adaptive_Boost/
│
├── 📁 ltspice/
│   ├── LDO_Final.asc          ← Main LDO schematic
│   ├── boost_detect.sub       ← Adaptive boost subcircuit
│   └── proj1.asc              ← Standalone op-amp reference
│
├── 📁 results/
│   ├── schematic_full.png          ← Full annotated schematic
│   ├── transient_baseline.png      ← 360mV droop without boost
│   ├── transient_boosted.png       ← 222mV droop with boost
│   ├── transient_comparison.png    ← Before/after overlay
│   └── dc_operating_point.png      ← .op simulation log
│
└── README.md
```

---

## Running the Simulations

### Requirements
- LTspice 26.x (free from Analog Devices)
- Windows or macOS

### Step 1 — Verify DC Operating Point
1. Open `LDO_Final.asc`
2. Enable `.op`, disable `.tran`
3. Run → `Ctrl+L` to view log
4. Verify `V(vout) = 3.302V`

### Step 2 — Baseline Transient (No Boost)
1. Comment out `XBOOST` directive
2. Enable `.tran 0 60u`
3. Run → plot `V(vout)`
4. Measure droop = **360 mV**

### Step 3 — Enhanced Transient (With Boost)
1. Uncomment `XBOOST` directive
2. Run `.tran 0 60u`
3. Plot `V(vout)`
4. Measure droop = **222 mV**

---

## Design Equations

**Output Voltage:**
```
VOUT = VREF × (1 + R1/R2) = 1.2 × (1 + 200k/114k) = 3.302V
```

**First-Stage Transconductance:**
```
Gm1 = √(2 × Kn × W/L × ID) = √(2 × 120µ × 10 × 25µ) ≈ 245 µA/V
```

**Boost Recovery Time Constant:**
```
τ = RDOWN × C_DETECT = 47kΩ × 1pF = 47 ns
```

**Droop without boost:**
```
ΔV = ΔI × Δt / C1 = 99mA × 3.63µs / 1µF = 360mV
```

---

## Transistor Specifications

| Name | Type | W/L | Function |
|---|---|---|---|
| MP1 | PMOS | 200µm/1µm | Pass transistor |
| M2, M3 | NMOS | 10µm/1µm | Differential pair |
| M4, M5 | PMOS | 10µm/1µm | Current mirror load |
| M6 | NMOS | 20µm/1µm | Second stage driver |
| M7 | PMOS | 20µm/1µm | Second stage load |
| M8 | PMOS | 20µm/180nm | Transient detector |
| MBOOST | NMOS | 100µm/180nm | Boost transistor |

---

## Industry Relevance

This architecture mirrors techniques used in:
- **TI TPS7A53** — adaptive quiescent current LDO
- **TI LP38512** — transient-enhanced low-Iq LDO  
- **NXP PCAL** series — adaptive bias power management

Building this from scratch at transistor level demonstrates:
- MOSFET biasing and operating region analysis
- Feedback loop stability and Miller compensation
- Transient response characterization
- Power-performance tradeoff optimization
- AC coupling for event detection

---

## References

1. P. Hazucha et al., "Area-Efficient Linear Regulator With Ultra-Fast Load Regulation," IEEE JSSC, 2005.
2. T. Y. Man et al., "A High Slew-Rate Push-Pull Output Amplifier for Low-Quiescent Current LDOs," IEEE TCAS-II, 2007.
3. K. N. Leung and P. K. T. Mok, "A Capacitor-Free CMOS Low-Dropout Regulator," IEEE JSSC, 2003.
4. M. Al-Shyoukh et al., "A Transient-Enhanced Low-Quiescent Current LDO," IEEE JSSC, 2007.
5. R. J. Milliken et al., "Full On-Chip CMOS Low-Dropout Voltage Regulator," IEEE TCAS-I, 2007.

---

## Author

**Dixit** — B.Tech Electronics & Communication Engineering  
Simulated in LTspice 26.x using Generic SPICE Level-1 MOSFET Models — 2025