# 🚗 Electabuzz: Line Follower Robot Hardware Platform

> **A custom hardware design, PCB layout, sensor interface, and power system package for a line-following mobile robot platform.**

Developed by **Team Electrabuzz** for *UNPLUGGED — A 24-Hour Hardware Hackathon* organized by DJSCE × IETE-ISF × DJS MicroMinds VLSI Club.

[![KiCad Version](https://img.shields.io/badge/KiCad-v9.0-blue?style=flat-square&logo=kicad)](https://kicad.org/)
[![Simulation](https://img.shields.io/badge/Simulation-LTSpice-red?style=flat-square)](https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html)
[![MCU](https://img.shields.io/badge/MCU-ESP32%20WROOM--32-green?style=flat-square&logo=espressif)](https://www.espressif.com/)

---

## 📸 Project Overview

![3D PCB Render](docs/assets/images/3d-pcb-render.jpeg)

---

## ⚡ Key Engineering Deliverables

- **Custom Form-Factor PCB**: 2-layer FR4 PCB designed in KiCad 9.0 to fit mechanical chassis boundaries, with separate ground returns for logic and motor circuits.
- **Integrated Sensor & Actuator Interface**: Onboard routing for an ESP32 microcontroller, 5-channel IR tracking array, TB6612FNG dual DC motor driver, 0.96" I²C OLED display, NEO-6M GPS, and ESP32-CAM module.
- **Dual-Domain Power Architecture**: Solar-assisted Li-ion battery charging (TP4056 + 2× 18650 cells) with LM2596 buck regulation for 5V/3.3V logic, alongside an isolated 12V motor supply rail.
- **Wide-Range Isolated Power Front-End (Simulation)**: LTSpice SPICE simulation of a $\pm 12\text{V}$ to $\pm 50\text{V}$ DC polarity-independent isolated flyback converter with logic-level polarity telemetry.
- **Design Verification**: Checked with KiCad Electrical Rules Check (ERC) and Design Rules Check (DRC).

---

## 🎯 Problem & Engineering Approach

### The Problem
Line-follower robots require clean sensor signal routing, reliable motor current delivery, and stable logic power. Connecting high-draw DC motors directly to the same unregulated battery rail as the microcontroller often causes voltage dips, electrical noise, and unwanted MCU resets.

### The Engineering Solution
**Electabuzz** addresses these requirements at the board level:
1. **Isolated Power Rails**: Logic circuitry (ESP32, sensors, OLED) is powered via an LM2596 buck regulator from 18650 Li-ion cells, while motors are driven through a dedicated 12V rail on the TB6612FNG driver.
2. **Consolidated Board Layout**: Integrates power management, compute, telemetry, and motor switching onto a single PCB shaped to match chassis mounting holes.
3. **Transient-Tolerant Power Simulation**: A supplementary flyback converter stage was modeled to evaluate polarity-independent operation and surge clamping under wide DC input ranges.

---

## 🏗 System Architecture

```
                              ┌───────────────────┐
                              │  6V Solar Panel   │
                              └─────────┬─────────┘
                                        │ (1N5819 Schottky Diode)
                                        ▼
 ┌──────────────────────┐     ┌───────────────────┐
 │ 2× 18650 Li-ion      │◄───►│  TP4056 Charger   │
 │ Battery (Parallel)   │     │ (Protection IC)   │
 └──────────────────────┘     └─────────┬─────────┘
                                        │ ~3.7V - 4.2V VBAT
                                        ▼
                              ┌───────────────────┐
                              │ LM2596 Buck Conv. │──► 5.0V Logic Rail (ESP32 VIN, CAM 5V)
                              └─────────┬─────────┘
                                        │
           ┌────────────────────────────┴────────────────────────────┐
           │                                                         │
           ▼                                                         ▼
 ┌───────────────────┐                                     ┌───────────────────┐
 │ ESP32-DEVKIT-V1   │◄────────────── UART0 ──────────────►│ ESP32-CAM (OV2640)│
 │ (Main Controller) │                                     └───────────────────┘
 └───┬───┬───┬───┬───┘
     │   │   │   │
     │   │   │   └─────────── I²C (GPIO 21/22) ───────────► 0.96" SSD1306 OLED (0x3C)
     │   │   └─────────────── UART2 (GPIO 16/17) ─────────► NEO-6M GPS Module
     │   └─────────────────── GPIO Inputs ────────────────► 5× IR Sensor Array
     │
     └─────────────────────── PWM / Direction ────────────► TB6612FNG Motor Driver
                                                                     │
                    (Dedicated 12V Motor Rail) ──────────────────────┼──► 2× 12V DC Motors
```

---

## 🔌 Hardware Interfaces & Pin Mapping

| ESP32 Pin | Connected Device | Pin / Function | Signal Type | Description |
|:---:|:---:|:---:|:---:|:---|
| **GPIO 25** | TB6612FNG | `AIN1` | Digital Out | Motor A Direction 1 |
| **GPIO 26** | TB6612FNG | `AIN2` | Digital Out | Motor A Direction 2 |
| **GPIO 27** | TB6612FNG | `PWMA` | PWM Out | Motor A Speed Control |
| **GPIO 14** | TB6612FNG | `BIN1` | Digital Out | Motor B Direction 1 |
| **GPIO 12** | TB6612FNG | `BIN2` | Digital Out | Motor B Direction 2 |
| **GPIO 13** | TB6612FNG | `PWMB` | PWM Out | Motor B Speed Control |
| **GPIO 32** | TB6612FNG | `STBY` | Digital Out | Driver Enable / Standby ($10\text{ k}\Omega$ pull-up) |
| **GPIO 21** | SSD1306 OLED | `SDA` | I²C Data | Display I²C Data line ($4.7\text{ k}\Omega$ pull-up) |
| **GPIO 22** | SSD1306 OLED | `SCL` | I²C Clock | Display I²C Clock line ($4.7\text{ k}\Omega$ pull-up) |
| **GPIO 16** | NEO-6M GPS | `TX` | UART RX2 | Serial GPS telemetry input |
| **GPIO 17** | NEO-6M GPS | `RX` | UART TX2 | Serial configuration output |
| **GPIO 34** | IR Sensor 1 | `OUT` | Digital In | Leftmost line sensor |
| **GPIO 35** | IR Sensor 2 | `OUT` | Digital In | Mid-left line sensor |
| **GPIO 36** | IR Sensor 3 | `OUT` | Digital In | Center line sensor |
| **GPIO 39** | IR Sensor 4 | `OUT` | Digital In | Mid-right line sensor |
| **GPIO 33** | IR Sensor 5 | `OUT` | Digital In | Rightmost line sensor |
| **GPIO 1** | ESP32-CAM | `U0RXD` | UART TX0 | Inter-module serial link |
| **GPIO 3** | ESP32-CAM | `U0TXD` | UART RX0 | Inter-module serial link |

> 📖 **Full Details**: For bus specifications and pinout notes, see [Architecture Documentation](docs/architecture.md).

---

## 🖥 PCB Design & Power Architecture

| Schematic Capture | Copper Layer Routing (Top & Bottom) |
|:---:|:---:|
| [![Schematic Overview](docs/assets/images/schematic-overview.jpeg)](docs/pcb-design.md) | [![Front & Back Copper](docs/assets/images/pcb-front-copper.jpeg)](docs/pcb-design.md) |

### Power System Overview
- **Logic Domain (5V / 3.3V)**: Two 18650 Li-ion cells in parallel (~3.7V nominal) charged via a TP4056 module (with auxiliary 6V solar charging), regulated to 5.0V through an LM2596 buck converter for the ESP32 and ESP32-CAM.
- **Motor Domain (12V)**: An external 12V power source powers the TB6612FNG `VM` pin, keeping motor noise and current surges off the microcontroller supply rail.

> 📖 **Full Details**: For layer stackups, see [PCB Design](docs/pcb-design.md). For power tree analysis, see [Power System Documentation](docs/power-system.md).

---

## 🔬 LTSpice Simulation (Bonus Challenge)

As part of the hackathon's "Brownie Points" challenge, a **polarity-independent isolated flyback power supply** was designed and simulated in LTSpice.

![LTSpice Simulation Waveform](simulation/assets/images/ltspice-flyback-output.jpeg)

> [!NOTE]
> **Source Disclosure**: The original hackathon repository included the simulation output waveform and schematic notes as an image (`ltspice-flyback-output.jpeg`). The standalone SPICE netlist files ([`polarity_protection_flyback.cir`](simulation/polarity_protection_flyback.cir) and [`polarity_protection_flyback.net`](simulation/polarity_protection_flyback.net)) were **reconstructed from the complete netlist text preserved in that original simulation output**, allowing the circuit to be opened and executed in standard SPICE simulators.

### Circuit Features
- **Wide Input Range ($\pm 12\text{V}$ to $\pm 50\text{V}$)**: Full-bridge Schottky rectifier guarantees positive internal voltage regardless of wire orientation.
- **Galvanic Isolation**: $100\text{kHz}$ flyback transformer ($1:1.2$ turns ratio) provides electrical isolation between primary input and output.
- **Regulated $+12.0\text{V}$ Output**: $12.7\text{V}$ Zener reference and NPN emitter follower deliver a regulated $12\text{V}$ output rail.
- **Polarity Telemetry**: Hardware comparator outputs a $0\text{V}$ (Normal) or $5\text{V}$ (Reversed) logic signal to indicate input polarity.

> 📖 **Full Details**: For stage-by-stage equations and netlists, see [Simulation Documentation](simulation/README.md).

---

## 📁 Repository Structure

```
Electabuzz_Unplugged/
├── .gitignore                              # KiCad, LTSpice, CAD, and OS ignore rules
├── README.md                               # Project showcase and overview
│
├── docs/                                  # Technical documentation
│   ├── architecture.md                    # System block diagrams & pin assignments
│   ├── hardware.md                        # Component specifications & mounting
│   ├── pcb-design.md                      # KiCad layout, stackup & ERC/DRC details
│   ├── power-system.md                    # Power tree & domain separation analysis
│   ├── simulation.md                      # Simulation challenge summary
│   ├── bom/                               # Manufacturing Bill of Materials
│   │   ├── README.md                      # Verified component list
│   │   ├── Bill_of_Materials.docx         # Original DOCX BOM
│   │   └── BOM.pdf                        # Original PDF BOM
│   └── assets/images/                     # Design screenshots and renders
│
├── hardware/
│   ├── pcb/                               # KiCad 9.0 PCB project files
│   │   ├── team_electrabuzz.kicad_pro     # Project master file
│   │   ├── team_electrabuzz.kicad_sch     # System schematic
│   │   └── team_electrabuzz.kicad_pcb     # PCB layout
│   └── cad/models/                        # 3D mechanical STEP models
│       ├── BK-18650-PC2--3DModel-STEP-269445.STEP
│       ├── DM-OLED096-636--3DModel-STEP-56544.STEP
│       └── MOTORJGB37-520.STEP
│
└── simulation/                            # SPICE simulation files
    ├── README.md                          # Theoretical breakdown & circuit equations
    ├── polarity_protection_flyback.cir   # Reconstructed SPICE netlist
    ├── polarity_protection_flyback.net   # LTSpice netlist format
    └── assets/images/                     # Simulation waveform plots
```

---

## 🔍 Project Scope & Future Roadmap

### Included in this Repository
- Complete KiCad 9.0 schematic, PCB layout, and design rule check reports.
- 3D STEP mechanical models for key onboard components.
- Manufacturing Bill of Materials (BOM) in Markdown, PDF, and DOCX formats.
- Reconstructed SPICE simulation netlists and theoretical documentation for the isolated power front-end.

### Future Implementation Scope (Firmware & Software)
- Microcontroller firmware for sensor reading and motor PWM generation.
- Closed-loop line tracking algorithms (e.g., PID steering control).
- Display driver routines for real-time status output on the OLED.
- ESP32-CAM firmware integration for visual data capture.

---

## 🚀 Quickstart

### Opening the PCB in KiCad
1. Install [KiCad 9.0 or higher](https://kicad.org/download/).
2. Clone this repository:
   ```bash
   git clone https://github.com/MayankBagad/Electabuzz_Unplugged.git
   ```
3. Open [`hardware/pcb/team_electrabuzz.kicad_pro`](hardware/pcb/team_electrabuzz.kicad_pro) in KiCad.

### Running the SPICE Simulation
1. Install [LTSpice](https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html) or [Ngspice](https://ngspice.sourceforge.io/).
2. Open [`simulation/polarity_protection_flyback.cir`](simulation/polarity_protection_flyback.cir).
3. Run transient analysis (`.tran 0 20m 0 1u uic`).

---

## 👥 Team & Acknowledgments

- **Team Electrabuzz** — Hardware & PCB Design, Power Architecture
- **Event**: *UNPLUGGED — 24-Hour Hardware Hackathon*
- **Organizers**: DJSCE × IETE-ISF × DJS MicroMinds VLSI Club

---

## 📄 License Notice

> [!NOTE]
> An open-source license has not yet been selected for this repository. Recommended options:
> - **Hardware Designs**: [CERN-OHL-P-2.0](https://ohwr.org/cern_ohl_p_v2.txt) (Permissive) or [CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/)
> - **Software / Netlists**: [MIT License](https://opensource.org/licenses/MIT) or [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0)
