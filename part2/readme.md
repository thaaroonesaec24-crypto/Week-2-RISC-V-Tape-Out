
````markdown
#  BabySoC: Functional Verification and Hands-on Modelling

This guide demonstrates the **hands-on simulation and verification of the BabySoC system-on-chip**, providing step-by-step instructions for functional testing, waveform analysis, and post-synthesis validation. It is intended for users aiming to **bridge theoretical SoC concepts with practical implementation**.

---

##  Quick Navigation

1. [Project Introduction](#project-introduction)  
2. [Setting Up the Repository](#setting-up-the-repository)  
3. [Directory Overview & Module Descriptions](#directory-overview--module-descriptions)  
4. [Converting TL-Verilog CPU Core](#converting-tl-verilog-cpu-core)  
5. [Module-Level RTL Simulation](#module-level-rtl-simulation)  
   - DAC  
   - PLL  
   - CPU  
6. [System-Level Pre-Synthesis Simulation](#system-level-pre-synthesis-simulation)  
7. [Monitoring and Signal Insights](#monitoring-and-signal-insights)  
8. [Gate-Level Synthesis Workflow](#gate-level-synthesis-workflow)  
9. [Comparative Analysis: Before vs After Synthesis](#comparative-analysis-before-vs-after-synthesis)  
10. [Concluding Notes](#concluding-notes)  

---

##  Project Introduction

BabySoC is a **miniature educational SoC** integrating:

| Component | Function |
|-----------|---------|
| **RVMYTH CPU Core** | Executes instructions, performs ALU and memory operations |
| **Phase-Locked Loop (PLL)** | Generates and stabilizes clock signals for the CPU |
| **10-bit DAC** | Converts digital CPU outputs to analog voltages |

**Purpose:** Learn module communication, clock management, and digital-to-analog conversion within an SoC framework.  

**Why this matters:** Functional modelling validates logic before synthesis, avoiding costly design errors.

---

##  Setting Up the Repository

1. Navigate to your workspace:

```bash
cd ~/Documents/Verilog/Labs
````

2. Clone the source repository:

```bash
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC
ls
```

>  **Reason:** Ensures all required modules, testbenches, and libraries are available locally.

---

##  Directory Overview & Module Descriptions

Project layout:

```
VSDBabySoC/
├── LICENSE
├── README.md
├── src/
│   ├── module/
│   │   ├── avsddac.v
│   │   ├── avsdpll.v
│   │   ├── rvmyth.tlv
│   │   └── vsdbabysoc.v
│   ├── include/
│   │   ├── sandpiper.vh
│   │   └── sandpiper_gen.vh
│   └── gls_model/
│       ├── sky130_fd_sc_hd.v
│       └── primitives.v
└── testbench.v
```

| Module           | Input Signals                              | Output Signals | Description                        |
| ---------------- | ------------------------------------------ | -------------- | ---------------------------------- |
| **avsddac.v**    | D[9:0], VREFH, VREFL                       | OUT            | Digital-to-analog conversion       |
| **avsdpll.v**    | REF, ENb_VCO, VCO_IN                       | CLK            | Clock generation and stabilization |
| **rvmyth.tlv**   | CLK, reset                                 | OUT[9:0]       | Behavioral RISC-V CPU core         |
| **vsdbabysoc.v** | reset, VCO_IN, ENb_CP, ENb_VCO, REF, VREFH | OUT            | Top-level SoC integration          |

>  **Reason:** Understanding the structure helps with compilation and simulation planning.

---

##  Converting TL-Verilog CPU Core

RVMYTH is written in **TL-Verilog** and needs to be converted to standard Verilog:

1. Install dependencies:

```bash
sudo apt install iverilog gtkwave python3-pip git
pip3 install sandpiper-saas pyyaml click
```

2. Generate Verilog from TL-Verilog:

```bash
python3 -m sandpiper \
-i src/module/rvmyth.tlv \
-o rvmyth.v --bestsv --noline -p verilog \
--outdir src/module
```

> **Reason:** Creates a Verilog CPU module compatible with Icarus Verilog simulations.

---

##  Module-Level RTL Simulation

### DAC Module

```bash
iverilog -o avsddac.vvp src/module/avsddac.v tb_avsddac.v
vvp avsddac.vvp
gtkwave tb_avsddac.vcd
```

| Input D | Extended Value | Analog OUT (V) |
| ------- | -------------- | -------------- |
| 3FE     | 1022           | 3.297          |

>  **Reason:** Ensures DAC outputs match expected analog voltages.

---

### PLL Module

```bash
iverilog -o avsdpll.vvp src/module/avsdpll.v tb_avsdpll.v
vvp avsdpll.vvp
gtkwave tb_pll.vcd
```

| Signal  | Description                 |
| ------- | --------------------------- |
| REF     | Reference clock input       |
| CLK     | PLL-generated clock, 8× REF |
| ENb_VCO | Clock enable                |

>  **Reason:** Verifies stable clock generation before connecting CPU.

---

### CPU Module

```bash
iverilog -o rvmyth.vvp -I src/include -I src/module src/module/rvmyth.v tb_rvmyth.v src/module/clk_gate.v
vvp rvmyth.vvp
gtkwave tb_rvmyth.vcd
```

| Signal   | Role               |
| -------- | ------------------ |
| CLK      | Timing reference   |
| reset    | CPU initialization |
| OUT[9:0] | Data for DAC       |

>  **Reason:** Confirms CPU instruction execution and data outputs are correct.

---

##  System-Level Pre-Synthesis Simulation

```bash
iverilog -o pre_synth_sim.vvp -DPRE_SYNTH_SIM -I src/include -I src/module src/module/testbench.v
vvp pre_synth_sim.vvp
gtkwave pre_synth_sim.vcd
```

**Signals to Observe:**

| Signal         | Purpose                   |
| -------------- | ------------------------- |
| CLK            | CPU timing reference      |
| reset          | CPU initialization        |
| RV_TO_DAC[9:0] | Data path from CPU to DAC |
| OUT            | Analog output             |

>  **Reason:** Validates full SoC functionality at RTL level.

---

##  Monitoring and Signal Insights

Key points to check in GTKWave:

| Signal         | Expected Behavior              |
| -------------- | ------------------------------ |
| CLK            | Continuous toggle              |
| reset          | Active at simulation start     |
| RV_TO_DAC[9:0] | Incrementing CPU output values |
| OUT            | Corresponding analog values    |

---

##  Gate-Level Synthesis Workflow

1. Copy necessary include files:

```bash
cp src/include/sp_verilog.vh .
cp src/include/sandpiper.vh .
cp src/include/sandpiper_gen.vh .
```

2. Start Yosys and run synthesis:

```bash
yosys
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include src/module/rvmyth.v
synth -top vsdbabysoc
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
setundef -zero
clean -purge
rename -enumerate
write_verilog vsdbabysoc_synth.v
```

>  **Reason:** Converts RTL to optimized gate-level netlist.

---

##  Comparative Analysis: Before vs After Synthesis

| Feature       | RTL (Pre-Synth)   | Gate-Level (Post-Synth)  |
| ------------- | ----------------- | ------------------------ |
| Functionality | Verified          | Verified                 |
| Clock         | Behavioral        | Mapped to standard cells |
| Output        | Matches CPU logic | Matches CPU logic        |

>  **Conclusion:** Functional integrity preserved after synthesis.

---

##  Concluding Notes

* BabySoC demonstrates **CPU → PLL → DAC integration**.
* Step-by-step simulation validates **signal integrity and module communication**.
* Pre- and post-synthesis verification ensures **correct operation**.
* Tables and waveform analysis provide a **clear view of system behavior**.

> **Tips:**
>
> * Use `vsdbabysoc_synth.v` for post-synthesis simulation.
> * Testbench modifications may be necessary to match netlist ports.
> * This guide blends **theoretical SoC design concepts with hands-on verification**.

```

---


Do you want me to do that next?
```
