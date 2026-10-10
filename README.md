<div align="center">

# EDGEGUARD

### Industrial Edge Monitoring Platform

**From field information to industrial supervision and remote connectivity —  
through embedded hardware, communication, multilayer PCB design, and real manufacturing.**

<br>

<img src="media/pcb-3d/edgeguard-hero.png" width="100%">

</div>

---

## Overview

EdgeGuard was developed during my PFA internship as an embedded monitoring and communication platform for transformer cabinets.

The objective was to bring several functions together on a single industrial electronic board: local processing, field communication, remote connectivity, monitoring, data storage, and diagnostics.

The system is built around an **STM32-based core** and combines:

- Isolated **RS-485 / Modbus RTU**
- **LTE Cat-1** cellular connectivity
- **GNSS** capability
- Local data storage
- Fault and relay-state monitoring
- USB and diagnostic interfaces
- Dedicated power and protection circuits

The project went from the first system requirements to a complete multilayer PCB and finally to a real manufactured board.

---

# System Architecture

```mermaid
flowchart LR

    FIELD["Field Signals<br/>Sensors · Relay States"]

    INPUT["Protection &<br/>Signal Conditioning"]

    MCU["STM32<br/>Embedded Core"]

    RS485["Isolated<br/>RS-485"]

    SCADA["Industrial<br/>SCADA"]

    LTE["LTE Cat-1<br/>Modem"]

    CLOUD["Remote<br/>Platform"]

    GNSS["GNSS"]

    MEMORY["Local<br/>Storage"]

    DEBUG["USB / Debug<br/>Interfaces"]

    FIELD --> INPUT
    INPUT --> MCU

    MCU <--> RS485
    RS485 <--> SCADA

    MCU <--> LTE
    LTE <--> CLOUD

    GNSS --> MCU
    MCU <--> MEMORY
    DEBUG <--> MCU
```
---
## From Intent to Copper

```mermaid
flowchart TD
    A["Requirements"] --> B["System Architecture"]
    B --> C["Component Selection"]
    C --> D["Schematic Design"]
    D --> E["Power & Isolation Strategy"]
    E --> F["PCB Placement"]
    F --> G["Routing"]
    G --> H["Electrical & Mechanical Checks"]
    H --> I["Manufacturing"]

    classDef input fill:#0B2533,stroke:#38BDF8,color:#FFFFFF,stroke-width:2px;
    classDef design fill:#172554,stroke:#818CF8,color:#FFFFFF,stroke-width:2px;
    classDef pcb fill:#18392B,stroke:#34D399,color:#FFFFFF,stroke-width:2px;
    classDef verify fill:#422006,stroke:#FBBF24,color:#FFFFFF,stroke-width:2px;
    classDef final fill:#3F1722,stroke:#FB7185,color:#FFFFFF,stroke-width:3px;

    class A input;
    class B,C,D,E design;
    class F,G pcb;
    class H verify;
    class I final;
```

Every stage influenced the next one.

A communication decision affected isolation.  
A modem decision affected the power architecture.  
The power architecture affected placement.  
Placement affected routing.

And eventually, all of these decisions shaped the final board.

---

## The Board Has Multiple Worlds

One of the main challenges was making several very different electrical functions coexist on the same PCB.

```mermaid
flowchart TB

    MCU["STM32<br/>Embedded Core"]:::core

    POWER["Power<br/>Distribution"]:::power
    RS485["Isolated<br/>RS-485"]:::industrial
    LTE["LTE Cat-1<br/>Connectivity"]:::wireless
    GNSS["GNSS<br/>Positioning"]:::wireless
    MEMORY["Local<br/>Storage"]:::memory
    MONITOR["Fault & Relay<br/>Monitoring"]:::monitor
    DEBUG["USB / Debug<br/>Interfaces"]:::debug

    POWER --> MCU
    RS485 <--> MCU
    LTE <--> MCU
    GNSS --> MCU
    MEMORY <--> MCU
    MONITOR --> MCU
    DEBUG <--> MCU

    classDef core fill:#172554,stroke:#818CF8,color:#FFFFFF,stroke-width:3px;
    classDef power fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef industrial fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef wireless fill:#312E81,stroke:#A78BFA,color:#FFFFFF,stroke-width:2px;
    classDef memory fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
    classDef monitor fill:#450A0A,stroke:#F87171,color:#FFFFFF,stroke-width:2px;
    classDef debug fill:#27272A,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;
```

The board brings together several different electrical environments:

- **Embedded processing** — the STM32 is the central controller of the system.
- **Industrial communication** — isolated RS-485 provides reliable Modbus RTU communication.
- **Cellular connectivity** — LTE Cat-1 enables remote communication.
- **GNSS** — provides positioning capability when required.
- **Local storage** — keeps important information available locally.
- **Monitoring** — handles fault and relay-state information.
- **Power** — supplies sections with very different electrical requirements.
- **Debug interfaces** — support programming, testing and diagnostics.

The real challenge was not adding these functions individually, but making all of them work reliably together on the same board.

---

---

## Decisions That Shaped the Board

The final PCB was shaped by a few key engineering decisions. Each one solved a different constraint of the system.

```mermaid
flowchart LR

    ISO["ISOLATION<br/>Separate industrial<br/>and logic domains"]:::cyan

    POWER["POWER<br/>Handle cellular modem<br/>current peaks"]:::orange

    RF["RF<br/>Protect antenna<br/>performance"]:::purple

    BOARD["EDGEGUARD<br/>MULTILAYER PCB"]:::core

    GND["GROUNDING<br/>Control return paths<br/>and reduce noise"]:::green

    PROT["PROTECTION<br/>Protect sensitive<br/>external interfaces"]:::red

    DEBUG["DIAGNOSTICS<br/>Support testing<br/>and maintenance"]:::yellow

    ISO --> BOARD
    POWER --> BOARD
    RF --> BOARD

    BOARD --> GND
    BOARD --> PROT
    BOARD --> DEBUG

    classDef core fill:#111827,stroke:#38BDF8,color:#FFFFFF,stroke-width:4px;
    classDef cyan fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef orange fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef purple fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;
    classDef green fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
    classDef red fill:#450A0A,stroke:#FB7185,color:#FFFFFF,stroke-width:2px;
    classDef yellow fill:#422006,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;
```

### Why these choices mattered

| Decision | Impact on the design |
|---|---|
| **Isolated RS-485** | Kept the industrial communication side separated from the MCU domain |
| **Dedicated modem power path** | Helped support the high current demand of cellular communication |
| **RF-aware placement** | Kept sensitive RF paths short and protected antenna areas |
| **Multilayer architecture** | Improved grounding, routing and power distribution |
| **Protection circuits** | Added protection between the board and the external environment |
| **Structured grounding** | Helped different electrical sections coexist more reliably |
| **Diagnostic interfaces** | Made programming, testing and troubleshooting easier |

> The board was not shaped by one big decision.  
> It was shaped by many small decisions that had to work together.

---
# Engineering Constraints Map

```mermaid
mindmap
  root((EdgeGuard))
    Power
      Multiple voltage domains
      Cellular current peaks
      Decoupling
      Protection
    Communication
      RS485
      Modbus RTU
      LTE Cat-1
      GNSS
    PCB Design
      Multilayer stackup
      Placement
      Routing
      Ground strategy
    Isolation
      Isolated communication
      Domain separation
    RF
      Antenna routing
      Keep-out areas
      Grounding
    Validation
      Electrical checks
      Mechanical checks
      DRC
      Manufacturing review
```
---

## PCB Design

This is where the system architecture became a physical board.

The design was implemented as a **4-layer PCB**, with the placement and routing organized around power, communication, isolation, RF and mechanical constraints.

<p align="center">
  <img src="media/pcb-3d/edgeguard-render.png" width="88%">
</p>

### Design Focus

| ⚡ Power Integrity | 🔌 Industrial Interfaces | 📡 RF & Connectivity | 🧩 PCB Integration |
|:---:|:---:|:---:|:---:|
| Stable power distribution | Isolated RS-485 | LTE Cat-1 integration | Functional placement |
| Local decoupling | Modbus communication | GNSS capability | Multilayer routing |
| Cellular current peaks | Interface protection | Antenna keep-out | Ground strategy |
| Protection stages | Domain separation | Short RF paths | Mechanical checks |

<br>

### From Placement to Routing

```mermaid
flowchart LR

    A["FUNCTIONAL<br/>ZONING"]:::blue
    B["COMPONENT<br/>PLACEMENT"]:::purple
    C["POWER & GROUND<br/>PLANNING"]:::orange
    D["SIGNAL<br/>ROUTING"]:::green
    E["DRC &<br/>VALIDATION"]:::yellow

    A --> B --> C --> D --> E

    classDef blue fill:#082F49,stroke:#38BDF8,color:#FFFFFF,stroke-width:2px;
    classDef purple fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;
    classDef orange fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef green fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
    classDef yellow fill:#422006,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;
```

The goal was not simply to connect every component.

The goal was to make the board **electrically reliable, physically organized and ready to manufacture**.

---

## From Screen to Board

One of the most rewarding parts of the project was seeing the design move from the screen into the real world.

```mermaid
flowchart LR

    A["SYSTEM<br/>ARCHITECTURE"]:::blue
    B["SCHEMATIC<br/>DESIGN"]:::purple
    C["PCB<br/>LAYOUT"]:::cyan
    D["3D<br/>VALIDATION"]:::orange
    E["MANUFACTURING<br/>FILES"]:::yellow
    F["PHYSICAL<br/>PCB"]:::green

    A --> B --> C --> D --> E --> F

    classDef blue fill:#082F49,stroke:#38BDF8,color:#FFFFFF,stroke-width:2px;
    classDef purple fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;
    classDef cyan fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef orange fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef yellow fill:#422006,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;
    classDef green fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:3px;
```

<p align="center">
  <img src="media/manufactured-board/board-top.jpg" width="72%">
</p>

Once the PCB was manufactured, every earlier design decision became physical.

Connector positions, footprints, dimensions, clearances and component placement were no longer just elements on a screen.

They became part of the real system.

> **The moment the design leaves the screen, every detail starts to matter.**

---

    MCU <--> LTE
    LTE <--> CLOUD

    GNSS --> MCU
    MCU <--> MEMORY
    DEBUG <--> MCU
