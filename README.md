## 1. Project Overview

### Purpose
The system is designed to control a heat exchanger for heating milk to a specific, adjustable temperature setpoint. The controller ensures consistent temperature control, safety monitoring, and operational reliability.

### Scope
This controller is responsible for:
- Controlling milk temperature based on product type
- Manual operation of the system in the event of software malfunction
- Logging liquid temperature during transfer and heating
- Basic fault detection and alarm signaling

---

## 2. System Description

### General Description
The control box contains electronic components that monitor milk temperature and regulate heating through a circulation pump on the heat exchanger and a transfer pump in the milk line. Multiple temperature sensors are used on the input and output of the milk lines, as well as a sensor in the hot water loop to precisely control heat transfer.

### Control Methodology
- **Circulation Pump:** Speed is determined by a PID controller, informed by input/output milk temperatures and the hot water loop temperature.
- **Transfer Pump:** Speed is maximized while maintaining output temperature control to optimize transfer speed and process efficiency.
- **Operating Modes:**
  - **Automatic:**
    1. Set control switch to desired position (e.g., Keifer, cream separation).
    2. Set mode switch to Automatic.
    3. Press the Start button.
    4. Observe pump speeds and monitor temperatures on the display.
  - **Manual:**
    1. Set speed control knobs to 0.
    2. Set mode selector to Manual.
    3. Press the Start button.
    4. Observe output temperature and adjust speed control knobs to reach the desired output temperature.

### Major Components
- **HMI Computer:** Raspberry Pi 5 w/ SSD boot drive  
- **Logic Board:** Arduino Uno  
- **Temperature Sensors:** 5x PT100  
- **Enclosure:** 12"x12"x6" IP-rated enclosure

---

## 3. General Operation Instructions

### Initial Setup
1. Mount and wire the box according to the provided schematic.
2. Connect power and verify grounding.
3. Connect sensor and motor cables to marked terminals.

### Automatic Operation
1. Power on the system
2. Set mode to automatic
3. Set product type to desired product
4. Press start button

### Manual Operation
1. Power on the system
2. Set mode to manual
3. Set speed knobs to 0%
4. Press start
5. Use control knobs to acheive desired output temperature
   

### Setpoint Adjustment 
Setpoints for each product type can be configured via HMI or the configuration file.

---

## 4. Functional Specification

### Inputs

| Input       | Description              | Range / Type                  |
| ----------- | ------------------------ | ----------------------------- |
| **PT1/PT2** | Milk temperature input   | -                             |
| **PT3/PT4** | Milk temperature output  | -                             |
| **PT5**     | Boiler water temperature | -                             |
| **Estop**   | Emergency Stop Button    | Breaks power from both motors |
| **SW1**     | System Power             | -                             |
| **SW2**     | Mode Control Switch      | Off, Automatic, Manual        |
| **SW3**     | Product Control Switch   | Keifer, Cream, Spare          |
| **POT1**    | Transfer Pump Control    | Speed control                 |
| **POT2**    | Circulation Pump Control | Speed control                 |
| **BT1**     | Start Button             | -                             |
| **BT2**     | Stop Button              | -                             |

### Outputs

| Output                      | Description             | Type         |
| --------------------------- | ----------------------- | ------------ |
| **Circulation Pump Power**  | Power supply            | 110V         |
| **Circulation Pump Signal** | Control Voltage         | 0-10V Analog |
| **Transfer Pump Power**     | Power supply            | 110V         |
| **Transfer Pump Signal**    | Control Voltage         | 0-10V Analog |
| **Beacon Control**          | Power for status beacon | 12V Digital  |

### Internal Parts List  

| Part                   | Description                                                   | Type |
| ---------------------- | ------------------------------------------------------------- | ---- |
| **12V Power Supply**   | Powers Uno, Beacon                                            | 5A   |
| **10V Power Supply**   | Powers manual motor control signals                           | 1A   |
| **5V Power Supply**    | Powers Raspberry Pi                                           | 5A   |
| **7" HDMI Screen**     | Displays temperatures, system status                          | -    |
| **Raspberry Pi**       | User interaction, information display, logging                | -    |
| **Arduino Uno**        | Controls motor signals, reads data, logic                     | -    |
| **MAX31865**           | Amplifies signal from PT100 sensors                           | -    |
| **10V PWM Generator**  | Converts Arduino signals to analog (0-10V)                    | -    |
| **Low Voltage Relays** | Sends 12V Power to beacon                                     | -    |
| **120V Relays**        | Gates power to motors, controlled by Estop and mode selectors | -    |
| **Optocoupler Module** | Converts signals to 5V for Arduino                            | -    |

### Control Logic Summary
1. System powers on and runs initialization checks on temperature probes.
2. Reads mode selector switches and control buttons to determine state.
3. On start, system sets the temperature setpoint and runs the transfer pump at 50%. The PID loop continuously reads temperatures.
4. Once output temperature stabilizes, the system increases the transfer pump speed until the circulation pump reaches nearly 100% speed.

### Alarm & Fault Handling

#### Beacon Indicators  

| Condition       | Meaning                     | Effect                                     |
| --------------- | --------------------------- | ------------------------------------------ |
| **Green Solid** | IDLE                        | Operation as expected                      |
| **Amber**       | Warning, Attention Required | Operation continues                        |
| **Red**         | Error (Estop, sensor error) | Power removed from motors, Estop condition |

#### Faults

| Condition                                 | Response |
| ----------------------------------------- | -------- |
| **PT1/PT2 delta > 5°C**                   | Warning  |
| **PT1/PT2 delta > 10°C**                  | Error    |
| **PT3/PT4 delta > 5°C**                   | Warning  |
| **PT3/PT4 delta > 10°C**                  | Error    |
| **PT5 < 110 or PT5 > 200**                | Warning  |
| **PT1 circuit broken, PT2 nominal**       | Warning  |
| **PT2 circuit broken, PT1 nominal**       | Warning  |
| **Both PT1 & PT2 circuits broken**        | Error    |
| **PT3 circuit broken, PT4 nominal**       | Warning  |
| **PT4 circuit broken, PT3 nominal**       | Warning  |
| **Both PT3 & PT4 circuits broken**        | Error    |
| **Setpoint not reached after 15 seconds** | Warning  |
| **Setpoint not reached after 30 seconds** | Error    |
| **Setpoint Delta > 2°C**                  | Warning  |
| **Setpoint Delta > 5°C**                  | Error    |

---

