# System Architecture & Interface Design

This document details the hardware interconnections, communication buses, and pin mappings implemented on the **Electabuzz** line-follower PCB.

---

## 1. System Block Diagram

```
                              ┌───────────────────┐
                              │  6V Solar Panel   │
                              └─────────┬─────────┘
                                        │ (1N5819 Diode)
                                        ▼
 ┌──────────────────────┐     ┌───────────────────┐
 │ 2× 18650 Li-ion      │◄───►│  TP4056 Charger   │
 │ Battery (Parallel)   │     │ (Protection IC)   │
 └──────────────────────┘     └─────────┬─────────┘
                                        │ ~3.7V - 4.2V VBAT
                                        ▼
                              ┌───────────────────┐
                              │ LM2596 Buck Conv. │──► 5V Main Rail (ESP32 VIN, CAM 5V)
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

## 2. Hardware Pinout Mapping Table

The table below reflects the exact schematic wiring in [`hardware/pcb/team_electrabuzz.kicad_sch`](../hardware/pcb/team_electrabuzz.kicad_sch):

| MCU Pin | Peripheral | Peripheral Pin | Signal Type | Hardware Description |
|:---:|:---:|:---:|:---:|:---|
| **GPIO 25** | TB6612FNG | `AIN1` | Digital Output | Motor A Direction Control 1 |
| **GPIO 26** | TB6612FNG | `AIN2` | Digital Output | Motor A Direction Control 2 |
| **GPIO 27** | TB6612FNG | `PWMA` | PWM Output | Motor A Speed Control |
| **GPIO 14** | TB6612FNG | `BIN1` | Digital Output | Motor B Direction Control 1 |
| **GPIO 12** | TB6612FNG | `BIN2` | Digital Output | Motor B Direction Control 2 |
| **GPIO 13** | TB6612FNG | `PWMB` | PWM Output | Motor B Speed Control |
| **GPIO 32** | TB6612FNG | `STBY` | Digital Output | Driver Standby Enable ($10\text{ k}\Omega$ pull-up) |
| **GPIO 21** | SSD1306 OLED | `SDA` | I²C Data | Display Data Line ($4.7\text{ k}\Omega$ pull-up) |
| **GPIO 22** | SSD1306 OLED | `SCL` | I²C Clock | Display Clock Line ($4.7\text{ k}\Omega$ pull-up) |
| **GPIO 16** | NEO-6M GPS | `TX` | UART RX2 | Serial GPS telemetry input |
| **GPIO 17** | NEO-6M GPS | `RX` | UART TX2 | Serial configuration output |
| **GPIO 34** | IR Sensor 1 | `OUT` | Digital Input | Leftmost tracking sensor (Input only) |
| **GPIO 35** | IR Sensor 2 | `OUT` | Digital Input | Mid-left tracking sensor (Input only) |
| **GPIO 36** | IR Sensor 3 | `OUT` | Digital Input | Center tracking sensor (`SENSOR_VP`) |
| **GPIO 39** | IR Sensor 4 | `OUT` | Digital Input | Mid-right tracking sensor (`SENSOR_VN`) |
| **GPIO 33** | IR Sensor 5 | `OUT` | Digital Input | Rightmost tracking sensor |
| **GPIO 1** | ESP32-CAM | `U0RXD` | UART TX0 | Inter-module communication |
| **GPIO 3** | ESP32-CAM | `U0TXD` | UART RX0 | Inter-module communication |

---

## 3. Subsystem Interface Provisions

### Motor Driving Interface (TB6612FNG)
- **Logic Rail**: Driven by the 3.3V bus from the ESP32.
- **Motor Power Rail**: Driven by the external 12V bus connected to `VM`.
- **Control Strategy**: 6 logic lines (2 direction inputs per motor + PWM speed) and 1 global standby line.

### Optical Tracking Interface (5× IR Sensors)
- The PCB accommodates a 5-channel reflective optical sensor arrangement.
- Sensor outputs connect directly to dedicated digital input pins on the ESP32 (`GPIO 33`, `34`, `35`, `36`, `39`).

### Serial Communication Subsystems
- **I²C Bus**: Shared between the ESP32 and the SSD1306 OLED display (`0x3C` I²C address) on `GPIO 21` (SDA) and `GPIO 22` (SCL).
- **UART2**: Dedicated to the NEO-6M GPS module for NMEA sentence reception (`GPIO 16` RX2, `GPIO 17` TX2).
- **UART0**: Hardware serial header linking the ESP32 DevKit to the ESP32-CAM module.
