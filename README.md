<p align="center">
  <img src="media/banner/edgeguard-banner.png" width="100%" alt="EdgeGuard Industrial Edge Monitoring Platform">
</p>

---

# Overview

EdgeGuard was developed during my PFA internship as an industrial electrical fault monitoring and remote supervision unit for power distribution systems.

The objective was to bring several functions together on a single industrial electronic board: local processing, field communication, remote connectivity, monitoring, data storage, and diagnostics.

The system is built around an **STM32-based core** and combines:

- Isolated **RS-485 / Modbus RTU**
- **LTE Cat-1** cellular connectivity
- **GNSS** capability
- Local data storage
- Fault and relay-state monitoring
- USB and diagnostic interfaces
- Dedicated power and protection circuits

<div align="center">

<br>

<img src="media/pcb-3d/edgeguard-hero.png" width="70%">

</div>

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
flowchart TB

    PCB(["EDGEGUARD<br/><b>MULTILAYER PCB</b>"]):::pcb

    subgraph ROBUST["ELECTRICAL ROBUSTNESS"]
        direction LR
        ISO["ISOLATION<br/><small>Separate field and logic domains</small>"]:::iso
        PROT["PROTECTION<br/><small>Protect external interfaces</small>"]:::prot
    end

    subgraph INTEGRITY["SIGNAL & POWER INTEGRITY"]
        direction LR
        POWER["POWER<br/><small>Handle modem current demand</small>"]:::power
        GND["GROUNDING<br/><small>Control return paths</small>"]:::gnd
        RF["RF<br/><small>Preserve antenna performance</small>"]:::rf
    end

    subgraph SERVICE["VALIDATION & SERVICEABILITY"]
        direction LR
        DIAG["DIAGNOSTICS<br/><small>Enable test and maintenance</small>"]:::diag
    end

    ROBUST ==> PCB
    INTEGRITY ==> PCB
    SERVICE ==> PCB

    classDef pcb fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:4px;

    classDef iso fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef prot fill:#450A0A,stroke:#FB7185,color:#FFFFFF,stroke-width:2px;

    classDef power fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef gnd fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
    classDef rf fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;

    classDef diag fill:#422006,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;

    style ROBUST fill:#0D1117,stroke:#22D3EE,stroke-width:1px
    style INTEGRITY fill:#0D1117,stroke:#C084FC,stroke-width:1px
    style SERVICE fill:#0D1117,stroke:#FACC15,stroke-width:1px
```

### Why these choices mattered

| Decision | Impact on the design |
|---|---|
| **Isolated RS-485** | Kept the industrial communication side separated from the MCU domain |
| **Dedicated modem power path** | Helped support the high current demand of cellular communication |
| **RF placement** | Kept sensitive RF paths short and protected antenna areas |
| **Multilayer architecture** | Improved grounding, routing and power distribution |
| **Protection circuits** | Added protection between the board and the external environment |
| **Structured grounding** | Helped different electrical sections coexist more reliably |
| **Diagnostic interfaces** | Made programming, testing and troubleshooting easier |


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

<table>
  <tr>
    <td align="center">
      <img src="media/pcb-3d/top-layer.png" width="100%">
      <br>
      <b>Top Layer</b>
    </td>
    <td align="center">
      <img src="media/pcb-3d/bottom-layer.png" width="100%">
      <br>
      <b>Bottom Layer</b>
    </td>
  </tr>
</table>
<p align="center">
  <img src="media/pcb-3d/edgeguard-3d.png" width="55%">
  <br>
  <b>3D Assembly View</b>
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
---

## Technologies & Tools

<p align="center">
  <b>Hardware & Processing</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/STM32-03234B?style=for-the-badge&logo=stmicroelectronics&logoColor=white">
  <img src="https://img.shields.io/badge/ARM_Cortex--M-0091BD?style=for-the-badge&logo=arm&logoColor=white">
  <img src="https://img.shields.io/badge/EEPROM-334155?style=for-the-badge">
</p>


<p align="center">
  <b>Industrial & Remote Communication</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/RS--485-0891B2?style=for-the-badge">
  <img src="https://img.shields.io/badge/Modbus_RTU-0E7490?style=for-the-badge">
  <img src="https://img.shields.io/badge/LTE_Cat--1-7C3AED?style=for-the-badge">
  <img src="https://img.shields.io/badge/GNSS-6D28D9?style=for-the-badge">
  <img src="https://img.shields.io/badge/USB-2563EB?style=for-the-badge&logo=usb&logoColor=white">
</p>

<p align="center">
  <b>Simulation & Development</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/LTspice-B91C1C?style=for-the-badge">
  <img src="https://img.shields.io/badge/HyperLynx-2563EB?style=for-the-badge">
  <img src="https://img.shields.io/badge/STM32CubeMX-03234B?style=for-the-badge&logo=stmicroelectronics&logoColor=white">
  <img src="https://img.shields.io/badge/STM32CubeIDE-0F4C81?style=for-the-badge&logo=stmicroelectronics&logoColor=white">
</p>

<p align="center">
  <b>PCB & Hardware Design</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Altium_Designer-A5915F?style=for-the-badge&logo=altiumdesigner&logoColor=white">
  <img src="https://img.shields.io/badge/JLCPCB tools-166534?style=for-the-badge">
  <img src="https://img.shields.io/badge/RF_Routing-9333EA?style=for-the-badge">
  <img src="https://img.shields.io/badge/Power_Integrity-EA580C?style=for-the-badge">
  <img src="https://img.shields.io/badge/Isolation-DC2626?style=for-the-badge">
</p>

<p align="center">
  <b>Engineering Workflow</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/System_Architecture-0284C7?style=for-the-badge">
  <img src="https://img.shields.io/badge/Schematic_Design-4F46E5?style=for-the-badge">
  <img src="https://img.shields.io/badge/PCB_Layout-059669?style=for-the-badge">
  <img src="https://img.shields.io/badge/DRC-F59E0B?style=for-the-badge">
  <img src="https://img.shields.io/badge/Hardware_Validation-B91C1C?style=for-the-badge">
</p>

---

---

## What Is Intentionally Not Included

This repository is meant to present the **project process, architecture and design decisions** behind the project.

Some production material is intentionally not published.

| Not Included | Reason |
|---|---|
| **Complete Schematics** | Production design details are kept private |
| **Gerber Files** | Manufacturing files are not part of the public repository |
| **Complete BOM** | Component sourcing and production details are excluded |
| **Production Firmware** | Only sanitized examples may be shared in the future |
| **Internal Configuration Data** | Project specific configuration is kept private |

<br>

> **The goal of this repository is to share the engineering journey without exposing sensitive production material.**

---


