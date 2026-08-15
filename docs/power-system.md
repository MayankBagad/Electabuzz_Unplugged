# Power Distribution Architecture

This document describes the dual-domain power architecture designed for the **Electabuzz** line-follower robot.

---

## 1. Power Distribution Overview

The robot implements two separate power supply domains to isolate sensitive microcontroller logic from high-current motor transients:

```
[ Domain 1: Logic & Telemetry Power ]
Solar Panel (6V) ──[1N5819 Diode]──► TP4056 (IN+)
                                          │
2× 18650 Li-ion Cells (~3.7V) ──────► TP4056 (BAT+)
                                          │
                                    TP4056 (OUT+) (~3.7V - 4.2V)
                                          │
                                          ▼
                                ┌───────────────────┐
                                │ LM2596 Buck Conv. │──► 5.0V Regulated Bus
                                └─────────┬─────────┘
                                          │
                 ┌────────────────────────┴────────────────────────┐
                 ▼                                                 ▼
         ESP32-DEVKIT (VIN)                                ESP32-CAM (5V Pin)
                 │
      (Onboard 3.3V LDO)
                 │
                 ▼
 3.3V Logic Bus ──► IR Sensors, SSD1306 OLED, NEO-6M GPS, TB6612 (VCC Pin)


[ Domain 2: High-Current Motor Power ]
External 12V Power Source ────────► TB6612 (VM Pin) ──────► 2× 12V DC Motors
```

---

## 2. Domain Breakdown

### Domain 1: Logic and Sensors (3.7V - 5.0V - 3.3V)
1. **Primary Energy Storage**: Two 18650 Li-ion battery cells connected in parallel (~3.7V nominal, 4.2V fully charged).
2. **Battery Management**: A TP4056 charge controller manages cell charging and protects against over-discharge.
3. **Auxiliary Energy Harvesting**: A 6V / 1W solar panel connects to the TP4056 input through a 1N5819 Schottky diode (preventing reverse current).
4. **5.0V Bus Regulation**: An LM2596 step-down switching regulator provides 5.0V DC to the ESP32 `VIN` pin and the ESP32-CAM `5V` input pin.
5. **3.3V Subsystem Bus**: The onboard LDO on the ESP32 DevKit steps down 5.0V to 3.3V to power the IR sensor array, SSD1306 OLED display, NEO-6M GPS module, and the TB6612FNG logic supply pin (`VCC`).

### Domain 2: Motor Actuation (12V)
1. **Separation Rationale**: The 18650 single-cell parallel arrangement outputs ~3.7V to 4.2V, which is insufficient to drive 12V DC gear motors. An external 12V DC power source connects to the TB6612FNG `VM` pin.
2. **Noise Immunity**: Operating the motor supply independently prevents inductive voltage spikes and stall-current sags from causing brownout resets on the ESP32 microcontroller.
3. **Common Reference**: The logic ground and motor ground share a single common ground connection on the PCB.
