**Company Name:** [Your Company] 
**Technical Support Email:** support@example.com 
**Phone:** +1-XXX-XXX-XXXX 
**Website:** www.yourcompany.com 

---
5. [Safety Systems](#5-safety-systems) 
6. [Electrical & Circuit Design](#6-electrical--circuit-design) 
7. [Troubleshooting Guide](#7-troubleshooting-guide) 
8. [Appendices](#8-appendices) 
9. [Contact & Support Information](#9-contact--support-information)

---

## 1. Project Overview

### Purpose
The purpose of this system is to control a heat exchanger used to heat milk to a specific, adjustable temperature setpoint. The controller ensures consistent temperature control, safety monitoring, and operational reliability.

### Scope
This cotnroller controls:
- Liquid temperature via heat exchanger
- Liquid temperature logging during transfer and heating
- Basic fault detection and alarm signaling

---

## 2. System Description

### General Description
The control box houses electronic components that monitor milk temperature and control the heating process via a circuilation pump on a heat exchanger and a transfer pump on the milk loop. The system uses multiple temperature sensors on the input and output of the milk lines as well as a sensor in the hot water loop to precisely control the output temperature of the milk. 

### Control Philosophy
- The speed of the circulation pump will be determined by a PID controller, informed by the input and output temperature of the milk as well as the temperature of the hot water loop.
- The speed of the transfer pump will be maximized while maintinaining temperature control of output to maximize the speed of transfer and the effeciency of the process.
- Multiple modes will be included to control the transfer or allow the operator to intervene
    - Automatic:
        - Set control switch to desired position(Keifer, cream seperation)
        - Set mode switch to automatic
        - Press start button
        - Observe pump speeds and monitor temperatures on the display
    - Manual:
        - Set the speed control knobs to 0
        - Set the mode selector to Manual
        - Press Start Button
        - Observe output temperature and manipulate speed control knobs to reach desired output temperature

### Major Components
- HMI computer: Raspberry pi 5 w/ SSD boot drive
- Logic Board: Arduino Uno
- Temperature Sensor: 5x PT100
- Enclosure: 12"x12"x6" IP-rated enclosure

---

## 3. Functional Specification

### Inputs
| Input        | Description                     | Range / Type     |
|--------------|---------------------------------|------------------|
| PT1/PT2  | Milk temperature input          |
| PT3/PT4 | Milk temperature output |
| PT5 | Boiler water temperature |
| Estop | Emergency Stop Button | Breaks hot and neutral from both motors |
| SW1 | System Power |
| SW2 | Mode Control Switch | Off, Automatic, Manual |
| SW3 | Product Control Switch | Keifer, Cream, Spare |
| POT1 | Transfer Pump Control | Speed control |
| POT2 | Circulation Pump Control | Speed Control |
| BT1 | Start Button |
| BT2 | Stop Button |

### Outputs
| Output       | Description                     | Type             |
|--------------|---------------------------------|------------------|
| Circulation Pump Power | Power supply | 110V |
| Circulation Pump Signal | Control Voltage | 0-10V Analog |
| Transfer Pump Power | Power Supply | 110V |
| Transfer Pump Signal | Control Voltage | 0-10V Analog |
| Beacon Control | Power for status beacon | 12V Digital |

### Internal Parts List
| Part | Description | Type |

| -----|-------|
| 12V Power Supply | Uno, Beacon | 5A |
| 10V Power Supply | Powers manual motor control signals | 1A |
| 5V Power Supply | Powers Pi | 5A |
| 7" HDMI Screen | Displays all temperature readings, system status, allows operator input |
| Raspberry pi | Handles user interaction, information display, logging |
| Arduino Uno | Communicates with amplifiers, controls signals to motors, reads data from estop, mode switches, performs all logic control |
| MAX31865 | Amplifies signal from PT100 sensors, DIN mounted |
| 10V PWM Generator | converts arduino signal voltage to 0-10V analog |
| Low voltage relays | Sends 12V Power to beacon from arduino control signals |
| 120V relays | Gates power to motors, controlled by estop and mode selectors |
| Latching Relay | Controls Start/Stop for manual mode |
| Optocoupler Module | Convert 24/12V signals to 5V for arduino |

### Control Logic Summary
1. System powers on and runs initialization check on temperature probes
2. System reads mode selector switches and control buttons to determine state
3. On start, system determiend setpoint temperature, runs transfer pump at 50%, continuously reads temperature from sensors and updates PID loop. 
4. After output temperature settles, controller increases speed of transfer pump until the speed of the circuilation pump is near 100% speed.

### Setpoint Adjustment
Setpoint for each product type can be configured via HMI, or config file

### Alarm & Fault Handling
#### Beacon Meaning
| Condition | Meaning | Effect |
| -- | -- | -- |
| Green Solid | IDLE | Operation as expected |
| Amber  | Warning, Operator Attention Required  | Operation continues |
| Red | Error, Estop, sensor error |Power removed from motors, estop condition |

#### Faults
| Condition           | Response                    |
|---------------------|-----------------------------|
| PT1/PT2 delta > 5 deg | Warning |
| PT1/PT2 delta > 10deg | Error |
| PT3/PT4 delta > 5 deg | Warning |
| PT3/PT4 delta > 10 deg | Error |
| PT5 < 110 or PT5 > 200 | Warning |
| PT1 circuit broken, PT2 nominal | Warning |
| PT2 circuit broken, PT1 nominal | Warning |
| PT1 circuit broken, PT2 circuit broken | Error |
| PT3 circuit broken, PT4 nominal | Warning |
| PT4 circuit broken, PT3 nominal | Warning |
| PT4 circuit broken, PT4 circuit broken | Error |
| Setpoint not reached after 15 seconds | warning |
| Setpoint not reached after 30 seconds | error |
| Setpoint Delta > 2deg | warning |
| Setpoing Delta > 5 deg | error |

---
## 4. General Operation Instructions

### Initial Setup
1. Mount and wire the box according to the provided schematic 
2. Connect power and verify grounding 
3. Connect sensor and motor cables to marked terminals

### Normal Operation
1. Power on system 
2. Ensure temperature sensor is in the milk flow path 
3. Observe system status LED or HMI 
4. Heater will operate automatically to maintain setpoint

### Setpoint Adjustment
If applicable, access HMI or internal DIP switch to adjust desired milk temperature

## 5. Safety Systems

- **Emergency Stop (E-Stop):** Immediately cuts all outputs 
- **Overtemperature Cutoff:** Prevents overheating of milk 
- **Sensor Fault Detection:** Disables heating if input is invalid
