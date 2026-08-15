# Hardware Components & Specifications

This document describes the physical components, modules, and mechanical hardware integrated into the **Electabuzz** line follower platform.

---

## 1. Primary Components & Specifications

| Component | Schematic Ref | Footprint / Package | Specifications | Functional Role |
|---|---|---|---|---|
| **ESP32-DEVKIT-V1** | `U8` | `MODULE_ESP32_DEVKIT_V1` | Dual-core Tensilica Xtensa 32-bit LX6 @ 240 MHz, 520 KB SRAM | Central processing module |
| **ESP32-CAM** | `U2` | `Display:OLED-128O064D` (Header) | OV2640 2MP Camera, onboard flash LED | Visual capture module |
| **TB6612FNG** | `U1` | `Package_SO:SSOP-24_5.3x8.2mm_P0.65mm` | Dual H-bridge, MOSFET-based, 1.2A cont. / 3.2A peak per channel | DC motor driver |
| **12V DC Gear Motors** | `M1`, `M2` | Wire terminals | 12V rated DC gear motors | Drive propulsion |
| **NEO-6M GPS** | `U6` | `XCVR_NEO-6M-GPS` | UART output, 9600 baud default, ceramic patch antenna | Position telemetry module |
| **IR Reflective Sensors** | `U3`, `U9` | `OptoDevice:Vishay_MINICAST-3Pin` | Reflective infrared emitter/receiver pairs | Track line detection |
| **18650 Battery Holders** | `BT1`, `BT2` | `BK-18650-PC2:BAT_BK-18650-PC2` | 2× 18650 Li-ion holder (parallel layout) | Primary onboard logic energy storage |
| **TP4056 Charger IC** | `U7` | `SOP127P600X175-9N` | Constant-Current / Constant-Voltage linear charging IC | Li-ion charge management |
| **LM2596 Buck Regulator** | `U5` | `LM2596:TO263-5` | Step-down switching regulator (3A rated) | 5V logic supply regulation |
| **SSD1306 OLED Display** | `U4` | `MODULE_DM-OLED096-636` | 0.96" Monochrome 128×64 pixels, I²C interface | Status display |
| **Solar Cell** | `SC1` | Wire pads | 6V / 1W nominal output | Auxiliary trickle charge source |

---

## 2. Mechanical Integration & 3D Models

The mechanical models for the board layout are stored in the [`hardware/cad/models/`](../hardware/cad/models/) directory:
- **`BK-18650-PC2--3DModel-STEP-269445.STEP`**: 18650 battery holder mechanical envelope.
- **`DM-OLED096-636--3DModel-STEP-56544.STEP`**: 0.96" OLED display module outline.
- **`MOTORJGB37-520.STEP`**: 12V DC gear motor reference model.

For manufacturing part numbers and detailed BOM documentation, refer to the [Bill of Materials](bom/README.md).
