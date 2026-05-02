# Modular Sorting and Palletizing System

> A multi-station industrial automation cell integrating PLCs, collaborative and industrial robotics, machine vision, pneumatics, and SCADA — designed, programmed, and commissioned as a modular learning platform for industrial automation education.

---

## My Role: PLC / Controls / Robotics Engineer

I served as the **controls and robotics engineer** on this project, and my responsibilities extended to include the full scope of system integration from system CAD design, I/O, creating educational guides, and network architecture. My work included:

### PLC Programming & System Orchestration
- Designed and implemented the **complete PLC control system** on Allen-Bradley CompactLogix using Studio 5000
- Architected a **state machine** to orchestrate all stations — cube dispensing, robotic pick-and-place, color inspection, sorting, palletizing, and pallet exchange — as a single coordinated production cell
- Developed **fault handling and recovery logic** including timeout watchdogs, sensor validation, and safe state transitions to ensure consistent, repeatable operation
- Implemented **recipe management logic** allowing operators to dynamically assign color mappings through the HMI without modifying PLC code

### Robot Programming & Integration
- Programmed **Epson SCARA robot** using SPEL+ for high-speed cube transfer from a queue conveyor to the identification conveyor with precise color sensor presentation positioning
- Integrated **two Universal Robots UR3 collaborative robots** — one for color-based sorting to pallets or reject conveyor, and one for vision-triggered pallet exchange between rotary table, input conveyor, and exit conveyor
- Designed the **robot-PLC handshaking architecture** for all three robots, managing busy/done signaling, command sequencing, and fault pass-through across vendor boundaries

### Sensor Integration & Extended I/O
- Configured and calibrated the **Banner QCM50-K5D60-Q8 color sensor** for four-color discrete output detection (red, white, blue, black)
- Integrated **photoelectric sensors** (diffuse, retroreflective, through-beam) for cube presence detection, conveyor positioning, and bin-level sorting triggers
- Configured **inductive proximity sensors** for binary-encoded tray identification (3 sensors → 8 unique tray IDs)
- Set up **EtherNet/IP remote I/O** modules and managed device-level network addressing and diagnostics

### Pneumatic & Conveyor Control
- Developed PLC logic for **pneumatic cylinder sequencing** (cube dispensing, pick-and-place actuation, air eject nozzles)
- Implemented **conveyor start/stop logic** with VFD speed control, position-based sorting triggers, and timer-based reject run-off for the final bin
- Tuned **nozzle pulse timing** and conveyor speeds to achieve reliable ejection at each bin position

### Safety & Commissioning
- Designed **emergency stop circuits** and safety interlocks across all stations
- Implemented **safe state logic** ensuring all motion stops and outputs de-energize on fault or E-stop
- Led system **commissioning, I/O checkout, and integration testing** across all stations

---

## System Overview

This system is a **complete automated production cell** that sorts colored cubes by color and palletizes them for downstream processing. It was built as a modular educational platform to provide hands-on experience with real industrial equipment.

### Process Flow

```
  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ Station 1│    │ Station 2│    │ Station 3│    │ Station 4│    │ Station 5│    │ Station 6│
  │   Cube   │───▶│ Pneumatic│───▶│  SCARA   │───▶│  Color   │───▶│   UR3    │───▶│  Vision  │
  │   Feed   │    │ Pick &   │    │  Robot   │    │  Sensor  │    │ Sorting  │    │  Pallet  │
  │          │    │  Place   │    │ Transfer │    │  ID      │    │  Robot   │    │  Mgmt    │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
   Pneumatic       Vacuum          Epson SCARA     QCM50 Color     UR3 Cobot       Cognex Vision
   Dispenser       End Effector    SPEL+ AOIs      4 Discrete      Sort/Reject     + UR3 Cobot
   Diffuse PE      Multi-Axis      Queue Conv      Red/White       Rotary Table    Pallet Swap
   Sensor          Actuation       ID Conveyor     Blue/Black      Reject Conv     In/Out Conv
```

### Station Descriptions

**Station 1 — Cube Dispensing:**
A pneumatic cylinder pushes cubes one at a time from a gravity-fed dispenser. A diffuse photoelectric sensor confirms a cube is in position and ready for pickup.

**Station 2 — Pneumatic Pick & Place:**
A multi-axis pneumatic actuator with a vacuum suction end effector picks cubes from the feed station and places them on a queue conveyor for robotic handling.

**Station 3 — SCARA Robot Transfer:**
An Epson SCARA robot picks cubes from the queue conveyor and presents them to the color sensor on an identification conveyor. 

**Station 4 — Color Identification:**
A Banner QCM50 color sensor identifies each cube as red, white, blue, or black. The PLC decodes the four discrete sensor outputs and determines the cube's destination based on the active recipe. Black cubes are always classified as rejects.

**Station 5 — UR3 Sorting:**
A UR3 collaborative robot picks cubes from the identification conveyor and places them either on a pallet (positioned on a rotary table) or on a reject conveyor, based on the color classification.

**Station 6 — Vision-Based Pallet Management:**
A machine vision system monitors pallets on the rotary table for fill status. When a full pallet is detected, a second UR3 removes it to an exit conveyor and loads an empty pallet from an input conveyor — all without operator intervention.

---

## Control Architecture

### PLC State Machine

The system operates on a state machine that coordinates all stations through a single production cycle:

| State | Name | Description |
|-------|------|-------------|
| 0 | **Idle** | SPEL initialized, waiting for tray and cubes |
| 1 | **Ready** | Tray present, cubes detected, recipe valid, awaiting start |
| 2 | **Pick** | Verify cube present, activate SPEL pick AOI |
| 3 | **Color Detection** | Cube at sensor, read and decode color value |
| 4 | **Moving to Conveyor** | Determine target bin, place cube on conveyor |
| 5 | **Sorting** | Conveyor runs to target bin, eject or run-off |

After sorting, the system loops back to **State 2** for the next cube, or returns to **State 1** when no cubes remain on the tray.

### Fault Handling

The system includes comprehensive fault detection and recovery:

- **One Shot** on every state transition
- **Cube presence verification** before and after each operation
- **Color sensor validation** (no-match detection)
- **Robot fault pass-through** from all three robots to PLC
- **Conveyor feedback monitoring** (commanded vs. actual)
- **Safe state transitions** — all motion stops on fault, operator reset required

---

## Industrial Technologies Used

This platform integrates a wide array of industrial systems and components:

### Controls & Programming
- Allen-Bradley CompactLogix PLC (Studio 5000)
- Ladder logic and structured text
- State machine design

### Robotics
- Epson SCARA robot — SPEL+ programming, 7 custom AOIs
- Universal Robots UR3 (×2) — sorting and pallet handling
- Robot-PLC handshaking across multiple vendors

### Sensors
- Banner QCM50-K5D60-Q8 color sensor (4-color discrete output)
- Photoelectric sensors (diffuse, retroreflective, through-beam)
- Inductive proximity sensors (binary tray identification)
- Pressure and vacuum switches

### Pneumatics
- Pneumatic cylinders and actuators
- Solenoid valve manifolds
- Vacuum generators and suction cups
- Air preparation units (FRL)

### Material Handling
- Belt and roller conveyors (VFD controlled)
- Rotary table with indexing
- Gravity-fed dispenser
- Pneumatic air eject nozzles

### Communication Protocols
- EtherNet/IP (PLC to I/O, PLC to robots)
- OPC-UA (Ignition SCADA integration)
- Modbus TCP/IP (device communication)
- TCP/IP sockets (vision system)

### Safety
- Hardwired emergency stop circuits
- Collaborative robot force limiting
- Safety-rated I/O and interlocks
- Light curtains and area monitoring

---

## System Specifications

| Parameter | Value |
|-----------|-------|
| Cube Colors | Red, White, Blue, Black (reject) |
| Cubes per Tray | 3 |
| Tray ID Encoding | 3-bit binary (8 unique IDs) |
| Color Sensor | Banner QCM50-K5D60-Q8, 4 discrete outputs |
| Color Detection Time | <10ms |
| Robots | 1× Epson SCARA, 2× UR3 |
| PLC Platform | Allen-Bradley CompactLogix |
| SCADA Platform | Inductive Automation Ignition |


---

## Educational Value

This system was designed as a **modular learning platform** for students entering the industrial automation field. It provides hands-on experience with the same equipment, protocols, and programming tools used in professional manufacturing environments.

The breadth of integrated technologies — spanning PLCs, three robots across two vendors, machine vision, pneumatics, multiple sensor types, conveyors, and SCADA — gives students real-world exposure to the complexity of modern automated systems and the integration challenges that come with multi-vendor, multi-protocol production cells.

Every component in this system is an industrial-grade product used in actual manufacturing facilities, ensuring that skills developed on this platform transfer directly to professional automation engineering roles.

---
