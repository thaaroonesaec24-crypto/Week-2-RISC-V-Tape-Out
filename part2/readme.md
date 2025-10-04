
---

# BabySoC: Functional Verification and Hands-on Modelling

This guide demonstrates the **hands-on simulation and verification of the BabySoC system-on-chip**, providing step-by-step instructions for functional testing, waveform analysis, and post-synthesis validation. It is intended for users aiming to **bridge theoretical SoC concepts with practical implementation**.

BabySoC is designed as an educational platform to explore the interplay between CPU, memory, peripherals, and timing management within a compact SoC environment.

---

## Quick Navigation

1. [Project Introduction](#project-introduction)
2. [Setting Up the Repository](#setting-up-the-repository)
3. [Directory Overview & Module Descriptions](#directory-overview--module-descriptions)
4. [Converting TL-Verilog CPU Core](#converting-tl-verilog-cpu-core)
5. [Module-Level RTL Simulation](#module-level-rtl-simulation)

   * DAC
   * PLL
   * CPU
6. [System-Level Pre-Synthesis Simulation](#system-level-pre-synthesis-simulation)
7. [Monitoring and Signal Insights](#monitoring-and-signal-insights)
8. [Gate-Level Synthesis Workflow](#gate-level-synthesis-workflow)
9. [Comparative Analysis: Before vs After Synthesis](#comparative-analysis-before-vs-after-synthesis)
10. [Concluding Notes](#concluding-notes)

---

## Project Introduction

BabySoC is a **miniature educational SoC** integrating:

| Component                   | Function                                                  |
| --------------------------- | --------------------------------------------------------- |
| **RVMYTH CPU Core**         | Executes instructions, performs ALU and memory operations |
| **Phase-Locked Loop (PLL)** | Generates and stabilizes clock signals for the CPU        |
| **10-bit DAC**              | Converts digital CPU outputs to analog voltages           |

The RVMYTH CPU core demonstrates a simple RISC-V pipeline, while the PLL ensures a stable clock frequency for proper CPU operation. The DAC allows verification of CPU outputs in an analog format, mimicking real-world interfacing.

**Purpose:** Learn module communication, clock management, and digital-to-analog conversion within an SoC framework.

Functional modelling validates logic before synthesis, preventing errors that could propagate to the physical design.

---

## Setting Up the Repository

1. Navigate to your workspace:

```bash
cd ~/home/vboxuser/VLSI
```

2. Clone the source repository:

```bash
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC
ls
```

![modulesused](https://github.com/thaaroonesaec24-crypto/Week-2-RISC-V-Tape-Out/blob/main/images/modules%20used.png)

Cloning ensures all project files, modules, testbenches, and supporting libraries are available locally. This setup also enables version control and reproducibility.

---

## Directory Overview & Module Descriptions

Project layout:

```
VSDBabySoC/
├── src/
│   ├── include/
│   │   ├── sandpiper.vh
│   │   └── other header files...
│   ├── module/
│   │   ├── vsdbabysoc.v      # Top-level module integrating all components
│   │   ├── rvmyth.v          # RISC-V core module
│   │   ├── avsdpll.v         # PLL module
│   │   ├── avsddac.v         # DAC module
│   │   └── testbench.v       # Testbench for simulation
└── output/
└── compiled_tlv/         # Holds compiled intermediate files if needed
```

| Module           | Input Signals                              | Output Signals | Description                        |
| ---------------- | ------------------------------------------ | -------------- | ---------------------------------- |
| **avsddac.v**    | D[9:0], VREFH, VREFL                       | OUT            | Digital-to-analog conversion       |
| **avsdpll.v**    | REF, ENb_VCO, VCO_IN                       | CLK            | Clock generation and stabilization |
| **rvmyth.tlv**   | CLK, reset                                 | OUT[9:0]       | Behavioral RISC-V CPU core         |
| **vsdbabysoc.v** | reset, VCO_IN, ENb_CP, ENb_VCO, REF, VREFH | OUT            | Top-level SoC integration          |

Understanding module I/O and dependencies is important to ensure correct simulation and integration.

---

## Converting TL-Verilog CPU Core

RVMYTH is written in **TL-Verilog**, which simplifies RTL design and verification. It must be converted to standard Verilog for simulation and synthesis.

### Step 1: Install Python virtual environment tools

```bash
sudo apt update
sudo apt install python3-venv python3-pip
```

`python3-venv` isolates project-specific Python dependencies, while `pip` installs required packages.

### Step 2: Create and activate a virtual environment

```bash
cd ~/VLSI/VSDBabySoC/
python3 -m venv sp_env
source sp_env/bin/activate
```

This ensures all Python tools installed later do not interfere with system-wide Python packages.

### Step 3: Install SandPiper-SaaS

```bash
pip install pyyaml click sandpiper-saas
```

SandPiper-SaaS is required to convert TL-Verilog to synthesizable Verilog.

### Step 4: Convert `rvmyth.tlv` to Verilog

```bash
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```

The generated `rvmyth.v` can now be used in standard Verilog simulations.

---

## Module-Level RTL Simulation

Module-level simulation allows verification of each block independently before integrating the full SoC.

### DAC Module

```bash
iverilog -o /home/vboxuser/VLSI/VSDBabySoC/output/avsddac.vvp /home/vboxuser/VLSI/VSDBabySoC/src/module/avsddac.v /home/vboxuser/VLSI/VSDBabySoC/src/module/tb_avsddac.v
vvp avsddac.vvp
gtkwave tb_avsddac.vcd
```

![DAC Module RTL Simulation](https://github.com/thaaroonesaec24-crypto/Week-2-RISC-V-Tape-Out/blob/main/images/gtkwave%20avsddac.vvp.png)

| Input D | Extended Value | Analog OUT (V) |
| ------- | -------------- | -------------- |
| 3FE     | 1022           | 3.297          |

Simulation confirms DAC outputs match expected analog voltages, ensuring accurate conversion.

---

### PLL Module

```bash
iverilog -o avsdpll.vvp src/module/avsdpll.v tb_avsdpll.v
vvp avsdpll.vvp
gtkwave tb_pll.vcd
```

![PLL MODULe](https://github.com/thaaroonesaec24-crypto/Week-2-RISC-V-Tape-Out/blob/main/images/gtk%20wave%20avsdpll.v.png)

| Signal  | Description                 |
| ------- | --------------------------- |
| REF     | Reference clock input       |
| CLK     | PLL-generated clock, 8× REF |
| ENb_VCO | Clock enable                |

Simulation ensures stable clock generation, crucial for synchronous CPU operation.

---

### CPU Module

```bash
iverilog -o rvmyth.vvp -I src/include -I src/module src/module/rvmyth.v tb_rvmyth.v src/module/clk_gate.v
vvp rvmyth.vvp
gtkwave tb_rvmyth.vcd
```

![rvmyth.v module ](https://github.com/thaaroonesaec24-crypto/Week-2-RISC-V-Tape-Out/blob/main/images/gtkwave%20tb_rvmyth.vcd.png)

| Signal   | Role               |
| -------- | ------------------ |
| CLK      | Timing reference   |
| reset    | CPU initialization |
| OUT[9:0] | Data for DAC       |

CPU simulation verifies instruction execution and correctness of the data path.

---
Perfect! Let’s continue rewriting the **entire README** in the same explanatory style, keeping **all images and commands intact**, but adding contextual explanations naturally instead of labeling it “theory.”

---

## System-Level Pre-Synthesis Simulation

Running a system-level simulation allows you to **verify the integrated SoC design** before synthesis. This helps detect issues that may not appear in module-level testing.

```bash
vvp pre_synth_sim.vvp
gtkwave pre_synth_sim.vcd
```

![presynthesis simulation](https://github.com/thaaroonesaec24-crypto/Week-2-RISC-V-Tape-Out/blob/main/images/GTKwave%20-%20pre_synth_sim.png)

**Signals to Observe:**

| Signal         | Purpose                   |
| -------------- | ------------------------- |
| CLK            | CPU timing reference      |
| reset          | CPU initialization        |
| RV_TO_DAC[9:0] | Data path from CPU to DAC |
| OUT            | Analog output             |

This ensures **the CPU, DAC, and PLL interact correctly**, and data paths are behaving as expected.

---

## Monitoring and Signal Insights

Key signals to monitor during simulation:

| Signal         | Expected Behavior              |
| -------------- | ------------------------------ |
| CLK            | Continuous toggle              |
| reset          | Active at simulation start     |
| RV_TO_DAC[9:0] | Incrementing CPU output values |
| OUT            | Corresponding analog values    |

Observing these signals in GTKWave helps identify **timing mismatches or incorrect signal propagation** early.

---

## Gate-Level Synthesis Workflow

Gate-level synthesis transforms the RTL design into a **standard-cell-based netlist** suitable for ASIC or FPGA implementation.

### Step 1: Copy required include files

```bash
cp src/include/sp_verilog.vh .
cp src/include/sandpiper.vh .
cp src/include/sandpiper_gen.vh .
```

These files provide **macros, constants, and tool-generated definitions** needed during synthesis.

### Step 2: Run Yosys synthesis

```bash
yosys
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include src/module/rvmyth.v
synth -top vsdbabysoc
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

| Step         | Purpose                                                           |
| ------------ | ----------------------------------------------------------------- |
| strash       | Structural hashing to reduce redundant logic                      |
| scorr        | Sequential sweeping for redundancy removal                        |
| ifraig       | Incremental FRAIGing: logic equivalence checking and optimization |
| retime;{D}   | Moves registers across combinational logic to optimize timing     |
| strash       | Re-run structural hashing after retiming                          |
| dch,-f       | Delay-aware combinational optimization in fast mode               |
| map,-M,1,{D} | Map logic to gates minimizing area and retime-aware               |

### Step 3: Additional optimization commands

```bash
flatten
setundef -zero
clean -purge
rename -enumerate
```

| Command           | Purpose / Usage                                             |
| ----------------- | ----------------------------------------------------------- |
| flatten           | Converts hierarchical design to a single-level netlist      |
| setundef -zero    | Replaces all undefined (x) values with logical 0            |
| clean -purge      | Removes unused wires, cells, and modules                    |
| rename -enumerate | Ensures all internal wires/cells have unique numbered names |

### Step 4: Export synthesized netlist

```bash
write_verilog vsdbabysoc_synth.v
```

| Command                            | Purpose                                                                                                      |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `write_verilog vsdbabysoc_synth.v` | Saves the optimized gate-level netlist in Verilog format for post-synthesis simulation or further processing |

This netlist represents the **fully mapped, timing-optimized version of the design**.

---

## Post-Synthesis Simulation

Post-synthesis simulation verifies that the **optimized gate-level netlist behaves identically to the RTL design**.

```bash
iverilog -o /home/vboxuser/VLSI/VSDBabySoC/output/synth/vsdbabysoc.synth.v -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/vboxuser/VLSI/VSDBabySoC/src/include -I /home/vboxuser/VLSI/VSDBabySoC/src/module -I  /home/vboxuser/VLSI/VSDBabySoC/src/gls_model /home/vboxuser/VLSI/VSDBabySoC/src/module/testbench.v
vvp vsdbabysoc_synth.vvp
gtkwave post_synth_sim.vcd 
```

![post synthesis simulation](https://github.com/thaaroonesaec24-crypto/Week-2-RISC-V-Tape-Out/blob/main/images/postsyth%20vsdbaby.png)

This simulation **ensures timing and logic optimizations did not alter functional behavior**, which is critical before hardware implementation.

---

## Comparative Analysis: Before vs After Synthesis

| Feature       | RTL (Pre-Synth)   | Gate-Level (Post-Synth)  |
| ------------- | ----------------- | ------------------------ |
| Functionality | Verified          | Verified                 |
| Clock         | Behavioral        | Mapped to standard cells |
| Output        | Matches CPU logic | Matches CPU logic        |

The post-synthesis netlist maintains **functional integrity while optimizing for area and timing**, demonstrating successful mapping to standard cells.

---

## Concluding Notes

* BabySoC demonstrates **CPU → PLL → DAC integration** in a compact educational SoC.
* Step-by-step simulation validates **signal integrity, module communication, and data flow**.
* Pre- and post-synthesis verification ensures **design correctness under timing constraints**.
* Tables and waveform snapshots provide a **clear view of system behavior and verification status**.

**Tips:**

* Use `vsdbabysoc_synth.v` for post-synthesis simulation.
* Adjust testbenches as needed to match synthesized netlist ports.
* This guide blends **practical verification workflows with design exploration**, helping bridge conceptual understanding with implementation.

---


