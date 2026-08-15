# 📋 Bill of Materials (BOM)

This directory contains the manufacturing Bill of Materials for the **Electabuzz Unplugged Line Follower Car**.

---

## 📑 Component Breakdown

The table below is derived directly from the project schematics and manufacturing documents ([`Bill_of_Materials.docx`](Bill_of_Materials.docx) / [`BOM.pdf`](BOM.pdf)):

| Item # | Designator / Symbol | Component Description | Value / Model | Footprint / Package | Qty | Functional Role |
|:---:|:---|:---|:---|:---|:---:|:---|
| 1 | `U8` | Microcontroller Module | ESP32-DEVKIT-V1 | `ESP32-DEVKIT-V1:MODULE_ESP32_DEVKIT_V1` | 1 | Primary MCU — 240 MHz dual-core, WiFi/BT, motor control |
| 2 | `U2` | Camera / Vision Module | ESP32-CAM (OV2640) | `Display:OLED-128O064D` | 1 | Real-time image capture & visual feedback |
| 3 | `U1` | Dual H-Bridge Motor Driver | TB6612FNG | `Package_SO:SSOP-24_5.3x8.2mm_P0.65mm` | 1 | Dual DC motor speed and direction driver |
| 4 | `M1`, `M2` | DC Drive Motors | MotorDC 12V | `Connector / Wire Pad` | 2 | Left and right propulsion gear motors |
| 5 | `U6` | Satellite Navigation | NEO-6M-GPS | `NEO-6M-GPS:XCVR_NEO-6M-GPS` | 1 | Telemetry & positioning via UART2 |
| 6 | `U3`, `U9` | Reflective IR Optical Sensors | TSSP58038 / IR array | `OptoDevice:Vishay_MINICAST-3Pin` | 2+ | Line detection and course navigation |
| 7 | `SC1` | Photovoltaic Harvester | Solar_Cell (6V / 1W) | `Connector / Wire Pad` | 1 | Auxiliary solar charging source |
| 8 | `BT1`, `BT2` | Li-ion Battery Holders | BK-18650-PC2 | `BK-18650-PC2:BAT_BK-18650-PC2` | 2 | Primary 18650 Li-ion battery storage |
| 9 | `U7` | Li-ion Charge Controller | TP4056 (with protection) | `TP4056:SOP127P600X175-9N` | 1 | Single-cell Li-ion charging & protection IC |
| 10 | `U5` | Step-Down Buck Converter | LM2596 | `LM2596:TO263-5` | 1 | Regulates battery rail down to stable 5V logic supply |
| 11 | `U4` | Graphic Display Module | DM-OLED096-636 | `DM-OLED096-636:MODULE_DM-OLED096-636` | 1 | 0.96" SSD1306 I²C OLED diagnostic screen |

---

## 📁 Source Documents

- [`Bill_of_Materials.docx`](Bill_of_Materials.docx) — Original Word document specification
- [`BOM.pdf`](BOM.pdf) — Printable PDF specification
