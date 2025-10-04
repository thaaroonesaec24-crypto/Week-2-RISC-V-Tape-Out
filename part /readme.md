
Perfect 👍 — here’s a **professionally styled and detailed Markdown version** of your

## 🧠 *“BabySoC Functional Overview”*

with a short intro, clear structure, and only the main topics in the Table of Contents.

---

# 🧩 BabySoC Functional Overview

### 📘 About

This document provides a functional overview of the **VSDBabySoC**, a compact open-source **RISC-V–based System on Chip (SoC)**. It explains the design objectives, architecture, and functionality of its key components — **RVMYTH**, **PLL**, and **DAC** — as well as their roles in system integration and simulation.

---

## 📑 Table of Contents

1. [Introduction to VSDBabySoC](#introduction-to-vsdbabysoc)
2. [System on Chip (SoC) Overview](#system-on-chip-soc-overview)
3. [Core Components](#core-components)

   * [RVMYTH Microprocessor](#rvmyth-microprocessor)
   * [Phase-Locked Loop (PLL)](#phase-locked-loop-pll)
   * [Digital-to-Analog Converter (DAC)](#digital-to-analog-converter-dac)
4. [Functional Flow](#functional-flow)
5. [Simulation and Verification](#simulation-and-verification)
6. [Applications and Learning Scope](#applications-and-learning-scope)
7. [Summary](#summary)

---

## 1️⃣ Introduction to VSDBabySoC

The **VSDBabySoC** is a small yet powerful SoC designed for educational and research purposes.
Its goal is to **test three open-source IP cores** — a RISC-V CPU, PLL, and DAC — and analyze their behavior when integrated into a single silicon platform.

**Key Highlights**

* Integrates digital and analog modules on the same chip
* Built on **Sky130 technology** (open-source PDK)
* Demonstrates real-time **digital-to-analog interfacing**
* Serves as a learning platform for **open-source hardware design**

---

## 2️⃣ System on Chip (SoC) Overview

A **System on Chip (SoC)** combines multiple functional components on a single silicon die.
These components, called **IP cores**, include processors, memory blocks, communication interfaces, and analog peripherals.

| **Type of IP Core** | **Example**             | **Function**                              |
| ------------------- | ----------------------- | ----------------------------------------- |
| Digital             | CPU, Memory, Controller | Logical processing and control            |
| Analog              | DAC, ADC, PLL           | Signal conversion and timing control      |
| Mixed-Signal        | Sensor Interface        | Bridge between analog and digital domains |

In the BabySoC, digital processing (RVMYTH) is paired with analog timing (PLL) and output generation (DAC).

---

## 3️⃣ Core Components

### 🧮 RVMYTH Microprocessor

* **Type:** 32-bit RISC-V educational core
* **Function:** Executes instructions and processes data
* **Special Register:** `r17` — stores digital values for DAC conversion
* **Language:** Written in **TL-Verilog**, later converted to Verilog using *SandPiper-SaaS*
* **Output Role:** Continuously updates data sent to DAC

**Example:**
When RVMYTH runs a program that generates a sawtooth pattern, it stores incremental values in register `r17`.
These values are forwarded to the DAC, which produces a corresponding analog waveform.

---

### ⏱️ Phase-Locked Loop (PLL)

* **Function:** Generates a stable, high-frequency clock signal
* **Multiplier:** 8× — means output clock = 8 × input clock
* **Role:** Synchronizes system timing for consistent RVMYTH and DAC operation

**Example:**
If the input clock = 10 MHz → PLL output = 80 MHz
This ensures high-speed operation and synchronization among all modules.

---

### 🎚️ Digital-to-Analog Converter (DAC)

* **Resolution:** 10-bit
* **Function:** Converts digital signals from RVMYTH into analog voltages
* **Output:** Continuous analog waveform (audio/video compatible)

**Example:**
If `r17` = 512 (binary `10 0000 0000`), DAC outputs an analog voltage at ~50% of its full-scale range.

---

## 4️⃣ Functional Flow

The internal functional flow of BabySoC can be summarized as follows:

1. **PLL** generates a stable clock signal.
2. **RVMYTH** executes program instructions, generating data patterns.
3. Data stored in **r17** register is sent to **DAC**.
4. **DAC** converts this digital data into an analog voltage.
5. The **output signal (OUT)** is then available for external analog devices.

| **Stage** | **Component** | **Function**                 |
| --------- | ------------- | ---------------------------- |
| 1         | PLL           | Clock generation             |
| 2         | RVMYTH        | Digital computation          |
| 3         | DAC           | Digital-to-Analog conversion |
| 4         | Output        | Analog signal generation     |

---

## 5️⃣ Simulation and Verification

The BabySoC design can be simulated using **Icarus Verilog** and **GTKWave**.

### 🧩 Pre-Synthesis Simulation

* Tests the logical correctness of the RTL code
* Run using `iverilog` before synthesis
* Outputs `.vcd` file for waveform viewing

**Command Example:**

```bash
iverilog -o output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM -I src/include -I src/module src/module/testbench.v
./output/pre_synth_sim/pre_synth_sim.out
```

### 📊 GTKWave Visualization

Open the generated VCD file:

```bash
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```

**Signals to Observe**

| **Signal Name**  | **Source** | **Purpose**          |
| ---------------- | ---------- | -------------------- |
| `CLK`            | PLL        | Clock input          |
| `reset`          | External   | Reset control        |
| `RV_TO_DAC[9:0]` | RVMYTH     | Digital data for DAC |
| `OUT`            | DAC        | Analog output        |

---

## 6️⃣ Applications and Learning Scope

* Ideal for **students** learning SoC design and verification
* Helps understand **digital–analog integration**
* Provides a **hands-on RISC-V** CPU implementation
* Demonstrates **clock synchronization** via PLL
* Teaches **waveform interpretation** using GTKWave

---

## 7️⃣ Summary

The **VSDBabySoC** is a foundational educational project demonstrating the integration of digital (RVMYTH), analog (DAC), and clocking (PLL) systems on a single chip.
It provides a practical and open-source environment for exploring **SoC design, simulation, and verification** — bridging the gap between **theory and real hardware behavior**.

---

Would you like me to add a **“Project Directory Overview”** section (with folder and file structure in a clean table)?
It’ll make the document look more complete for GitHub.
