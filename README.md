# 🚗 Line Follower Car — UNPLUGGED 24-Hour Hardware Hackathon

> **Event:** UNPLUGGED — A 24-Hour Hardware Hackathon
> **Organised by:** DJSCE × IETE-ISF × DJS MicroMinds VLSI Club
> **Round:** Round II — PCB Design + CAD Integration

---

## 📌 Table of Contents

- [Problem Statement](#problem-statement)
- [System Overview](#system-overview)
- [Component List (BOM)](#component-list-bom)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [PCB Design](#pcb-design)
- [CAD Model](#cad-model)
- [Brownie Points — LTSpice Simulation](#brownie-points--ltspice-simulation)
- [Repository Structure](#repository-structure)
- [Tools & Software Used](#tools--software-used)
- [How to Build](#how-to-build)
- [Team](#team)

---

## Problem Statement

Design and develop a high-efficiency schematic and PCB layout using **KiCad** within the given time frame to serve as the **central control system for a line follower car**.

The PCB must:
- Fit within the exact mechanical boundaries of the provided chassis reference
- Interface all provided components with proper power distribution and signal integrity
- Pass Electrical Rule Check (ERC) and Design Rule Check (DRC)
- Be accompanied by a complete **CAD model** demonstrating structural, accessibility, and thermal considerations

---

## System Overview

```
Solar Panel
    │
    ▼ (1N5819 blocking diode)
TP4056 Li-ion Charger ◄──── 18650 Battery Cell
    │
    ▼
LM2596 Buck Converter (→ 5V)
    │
    ├──► ESP32 WROOM-32 (main MCU)
    │         │
    │         ├── I²C  ──────► 0.96" OLED Display (SSD1306)
    │         ├── UART2 ──────► NEO-6M GPS Module
    │         ├── GPIO (PWM) ─► TB6612FNG Motor Driver ──► 2× 12V DC Gear Motors
    │         ├── GPIO (IN) ──► IR Sensor Array (×5)
    │         └── UART0 ──────► ESP32-CAM (OV2640)
    │
    └──► ESP32-CAM (5V direct)
```

---

## Component List (BOM)

| Sr. No. | Component | Model / Spec | Qty | Role |
|---------|-----------|--------------|-----|------|
| 1 | Microcontroller | ESP32 WROOM-32 | 1 | Main MCU — WiFi, BT, dual-core 240 MHz |
| 2 | Camera Module | ESP32-CAM (OV2640) | 1 | 2MP visual feedback / logging |
| 3 | Motor Driver | TB6612FNG | 1 | Dual H-bridge for 2 DC motors |
| 4 | DC Gear Motor | 12V DC Gear Motor | 2 | Left & right drive wheels |
| 5 | GPS Module | NEO-6M | 1 | Position tracking via UART2 |
| 6 | IR Sensor Module | Generic reflective IR | 5 | 5-channel line detection array |
| 7 | Solar Panel | 6V / 1W | 1 | Battery charging via TP4056 |
| 8 | Battery Holder | 18650 Li-ion cell holder | 1 | Primary energy storage |
| 9 | Charger IC | TP4056 (with protection) | 1 | Li-ion charge controller |
| 10 | Buck Converter | LM2596 (adjustable) | 1 | Step-down to 5V for logic |
| 11 | OLED Display | 0.96" SSD1306 I²C | 1 | Status display (speed, GPS, sensor state) |
| — | Schottky Diode | 1N5819 | 1 | Solar reverse-current protection |
| — | Pull-up Resistors | 4.7 kΩ | 2 | I²C bus (SDA + SCL) |
| — | Pull-up Resistor | 10 kΩ | 1 | TB6612 STBY pin |

---

## Circuit & Pin Connections

### ESP32 → TB6612FNG Motor Driver

| ESP32 Pin | TB6612 Pin | Signal Type | Notes |
|-----------|------------|-------------|-------|
| GPIO25 | AIN1 | Digital | Motor A direction 1 |
| GPIO26 | AIN2 | Digital | Motor A direction 2 |
| GPIO27 | PWMA | PWM (LEDC) | Motor A speed |
| GPIO14 | BIN1 | Digital | Motor B direction 1 |
| GPIO12 | BIN2 | Digital | Motor B direction 2 |
| GPIO13 | PWMB | PWM (LEDC) | Motor B speed |
| GPIO32 | STBY | Digital | HIGH = active; 10 kΩ pull-up |
| 3.3V | VCC | Power | Logic supply |
| GND | GND | Power | Common ground |
| 12V bus | VM | Power | Motor supply (max 15V) |

**TB6612 Outputs → Motors**

| TB6612 Pin | Motor Terminal |
|------------|----------------|
| AO1 | Left motor (+) |
| AO2 | Left motor (−) |
| BO1 | Right motor (+) |
| BO2 | Right motor (−) |

---

### IR Sensor Array → ESP32

| IR Sensor | ESP32 Pin | Notes |
|-----------|-----------|-------|
| Sensor 1 OUT | GPIO34 (input only) | Leftmost — no internal pull-up |
| Sensor 2 OUT | GPIO35 (input only) | Left-centre |
| Sensor 3 OUT | GPIO36 (VP) | Centre |
| Sensor 4 OUT | GPIO39 (VN) | Right-centre |
| Sensor 5 OUT | GPIO33 | Rightmost |
| VCC (all) | 3.3V rail | 3.3V supply |
| GND (all) | GND | Common ground |

---

### NEO-6M GPS → ESP32 (UART2)

| NEO-6M Pin | ESP32 Pin | Notes |
|------------|-----------|-------|
| TX | GPIO16 (RX2) | GPS → ESP32 (9600 baud default) |
| RX | GPIO17 (TX2) | ESP32 → GPS (config, optional) |
| VCC | 3.3V | 45 mA typical |
| GND | GND | Common ground |

---

### 0.96″ OLED SSD1306 → ESP32 (I²C)

| OLED Pin | ESP32 Pin | Notes |
|----------|-----------|-------|
| SCL | GPIO22 | 4.7 kΩ pull-up to 3.3V |
| SDA | GPIO21 | 4.7 kΩ pull-up to 3.3V |
| VCC | 3.3V | I²C address: 0x3C |
| GND | GND | Common ground |

---

### Power Chain

```
Solar Panel (+) ──[1N5819]──► TP4056 IN+
Solar Panel (−) ────────────► TP4056 IN−
18650 (+) ──────────────────► TP4056 BAT+
18650 (−) ──────────────────► TP4056 BAT−
TP4056 OUT+ (~4.2V) ────────► LM2596 IN+
LM2596 OUT+ (5V) ───────────► ESP32 VIN
LM2596 OUT+ (5V) ───────────► ESP32-CAM 5V
ESP32 3.3V (out) ───────────► IR Array VCC, OLED VCC, NEO-6M VCC, TB6612 VCC
12V supply ─────────────────► TB6612 VM  (separate motor rail)
```

> ⚠️ **Important:** The 18650 single cell outputs ~4.2V at full charge — insufficient for 12V motors. A **separate 12V supply or 3S Li-ion pack** is required for the motor rail. The 18650 + TP4056 powers logic only.

---

### ESP32-CAM

| ESP32-CAM Pin | Connection | Notes |
|---------------|------------|-------|
| 5V | LM2596 5V output | Up to 310 mA peak |
| GND | Common GND | — |
| U0TXD (GPIO1) | ESP32 GPIO3 (RX0) | Optional UART logging |
| U0RXD (GPIO3) | ESP32 GPIO1 (TX0) | Optional control |
| GPIO0 | GND (to flash) | Pull LOW to program; float for run |
| IO4 | Optional GPIO | Onboard flash LED control |

---

## PCB Design

All KiCad project files are in the `/pcb/` directory.

### Deliverables

- [ ] Complete PCB layout screenshot
- [ ] Front copper layer (F.Cu)
- [ ] Back copper layer (B.Cu)
- [ ] 3D rendered view of PCB
- [ ] Schematic diagram
- [ ] ERC (Electrical Rule Check) — 0 errors
- [ ] DRC (Design Rule Check) — 0 errors

### Design Constraints

- PCB dimensions match chassis reference (functionally equivalent in size and mounting)
- Compact layout with clean signal routing
- Separate ground pours for logic and motor sections (star ground at single tie point)
- Decoupling capacitors (100 nF) on VCC pins of ESP32, TB6612, and sensors
- Thermal consideration: TB6612 pad exposed for heat dissipation

---

## CAD Model

CAD files are in the `/cad/` directory.

### Deliverables

- [ ] Rendered images of enclosure and full assembly
- [ ] PCB integrated within CAD model
- [ ] Structural considerations — mounting holes aligned to chassis
- [ ] Accessibility considerations — USB/programming port accessible without disassembly
- [ ] Thermal considerations — motor driver ventilation slots, battery compartment venting

---

## Brownie Points — LTSpice Simulation

**Task:** Design a polarity-independent input interface using LTSpice capable of:
- Accepting DC input: **12V to 50V with unknown polarity**
- Generating a regulated **+12V DC output**
- Producing a **polarity-indicating digital signal** (0V = normal, 5V = reversed)
- Electrical isolation between input and output stages
- Safe operation under both normal and reverse polarity

Simulation files are in the `/simulation/` directory.

### Deliverables

- [ ] LTSpice schematic screenshots
- [ ] Simulation waveform screenshots (normal polarity)
- [ ] Simulation waveform screenshots (reversed polarity)
- [ ] Output voltage verification (stable +12V)
- [ ] Polarity indicator signal verification (0V / 5V)

---

## Repository Structure

```
unplugged-line-follower/
│
├── README.md
│
├── pcb/
│   ├── line_follower.kicad_pro
│   ├── line_follower.kicad_sch       # Schematic
│   ├── line_follower.kicad_pcb       # PCB layout
│   ├── fp-lib-table                  # Footprint libraries
│   ├── sym-lib-table                 # Symbol libraries
│   ├── gerbers/                      # Fabrication files
│   │   ├── line_follower-F_Cu.gbr
│   │   ├── line_follower-B_Cu.gbr
│   │   ├── line_follower-Edge_Cuts.gbr
│   │   └── ...
│   └── screenshots/
│       ├── schematic.png
│       ├── pcb_layout.png
│       ├── front_copper.png
│       ├── back_copper.png
│       └── 3d_render.png
│
├── cad/
│   ├── assembly.step                 # Full assembly STEP file
│   ├── enclosure.f3d                 # Fusion 360 / FreeCAD source
│   └── renders/
│       ├── top_view.png
│       ├── side_view.png
│       └── exploded_view.png
│
├── simulation/
│   ├── polarity_protection.asc       # LTSpice schematic
│   ├── polarity_protection.net       # Netlist
│   └── screenshots/
│       ├── schematic.png
│       ├── normal_polarity_waveform.png
│       └── reverse_polarity_waveform.png
│
├── bom/
│   └── BOM.csv                       # Bill of Materials
│
├── firmware/                         # (optional) Reference code
│   └── main.ino
│
└── docs/
    └── chassis_reference.pdf
```

---

## Tools & Software Used

| Tool | Purpose |
|------|---------|
| KiCad 7.x / 8.x | Schematic capture and PCB layout |
| Git + GitHub | Version control and submission |

---



### Generating Gerbers

In KiCad PCB Editor: `File → Fabrication Outputs → Gerbers`

---

## Team

| Name | Role |
|------|------|
| — | PCB Design (KiCad) |

---

*Submitted for UNPLUGGED — A 24-Hour Hardware Hackathon | DJSCE × IETE-ISF × DJS MicroMinds VLSI Club*
