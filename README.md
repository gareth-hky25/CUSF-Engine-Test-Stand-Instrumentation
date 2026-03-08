# 🚀 CUSF Static Fire Test Stand — Avionics

**Hardware, firmware, and documentation for the electronics that control and instrument the Cambridge University Spaceflight static fire test stand.**

This repo contains documents for CUSF engine test stand. The electronics are split across three circuit boards:

| **Master Board** | Comms bridge between the laptop and the rest of the system. Receives commands over USB/serial/Ethernet and relays them to the other boards. |
| **Servo + Solenoid Board**  Drives up to 6 solenoid valves (MPQ6610 half-bridge ICs) and up to 4 servo-actuated ball valves with position feedback. The muscle of the system. |
| **Instrumentation Board** Reads pressure transducers, thermocouples, and a load cell for thrust measurement. The eyes and ears of the system. |

A laptop running ground-station software sends commands to the Master Board, which forwards them to the Servo + Solenoid Board (to open/close valves) and polls the Instrumentation Board (to read sensor data back). A hardware interlock chain ensures that no valve can be actuated unless all safety conditions are met.

## Board Details

### Master Board

The Master Board is the bridge between the operator's laptop and the test stand hardware. It receives high-level commands over USB and translates them into bus transactions to the other boards.

**Responsibilities:**
- Parse and validate commands from the ground station software
- Forward valve commands to the Servo + Solenoid Board
- Poll the Instrumentation Board for sensor readings and relay data back to the laptop
- Manage the system state machine (idle → armed → firing → safing)
- Assert/de-assert the hardware interlock arm signal
- Log all commands, telemetry, and events with timestamps

---

### Servo + Solenoid Board

This board directly controls all actuated valves on the test stand. It combines solenoid driving (for fast on/off valves) and servo driving (for proportional ball valves) onto a single PCB. Uses an ESP32 microcontroller to control servos and solenoids. 

#### Solenoid Section

Each of the 6 solenoid channels uses an **MPS MPQ6610** half-bridge power driver. This single IC replaces what would otherwise be a high-side MOSFET, low-side MOSFET, gate driver, bootstrap circuit, and current-sense shunt — all in an 8-pin SOIC package.

**Why the MPQ6610:**
- 4V–55V input range covers our 24V bus with margin
- 3A output current with internal 100mΩ (HS) / 120mΩ (LS) on-resistance
- Built-in current sensing — sources 100µA per amp of load current on the ISET pin, no external shunt needed
- Cycle-by-cycle current regulation (constant off-time PWM) set by one external resistor
- Over-current protection, over-temp shutdown, UVLO, and open-load detection
- AEC-Q100 Grade 1 qualified (automotive reliability)
- Accepts 3.3V/5V logic directly on EN and IN pins


#### Servo Section

Up to 4 servos
 - PWM signal from ESP32

### Instrumentation Board

This board is the sensor interface. It conditions and digitises analog signals from the test stand sensors and makes the data available to the Master Board over the inter-board bus.

## Repository Structure

```
cusf-static-fire/
│
├── hardware/
│   ├── master-board/
│   │   ├── kicad/              # Schematic + PCB project
│   │
│   ├── servo-solenoid-board/
│   │   ├── kicad/
│   │
│   ├── instrumentation-board/
│   │   ├── kicad/

│   │
│   └── lib/                    # Shared KiCad symbols + footprints
│       ├── cusf-symbols.kicad_sym
│       └── cusf-footprints.pretty/
│
├── firmware/
│   ├── master/
│   ├── servo/
│   ├── solenoid/
│   ├── instrumentation/
│
├── ground-station/             # Laptop software (GUI + logging) #Development hasn't started
│
├── datasheets/
│   ├── MPQ6610.pdf
│   └── ...
│
├── docs/
│   ├── design.docx   
│   └── ...
│
└── README.md                  
