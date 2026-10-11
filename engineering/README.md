# Engineering Design Notes

This section focuses on the technical decisions that shaped EdgeGuard.

The goal is not to reproduce the complete production design, but to explain the main engineering challenges, trade-offs and validation steps behind the hardware.

---

## System Design Philosophy

EdgeGuard combines several electrical environments on the same board:

- Industrial communication
- Cellular connectivity
- GNSS
- Local storage
- Fault monitoring
- Power conversion

```mermaid
flowchart TB

    CORE["STM32<br/>CONTROL CORE"]:::core

    POWER["POWER<br/>ARCHITECTURE"]:::power
    RS485["ISOLATED<br/>RS-485"]:::comm
    LTE["LTE CAT-1<br/>MODEM"]:::rf
    GNSS["GNSS"]:::rf
    STORE["LOCAL<br/>STORAGE"]:::memory
    MON["FAULT<br/>Circuitry "]:::monitor

    POWER --> CORE
    RS485 <--> CORE
    LTE <--> CORE
    GNSS --> CORE
    STORE <--> CORE
    MON --> CORE

    classDef core fill:#172554,stroke:#818CF8,color:#FFFFFF,stroke-width:3px;
    classDef power fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef comm fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef rf fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;
    classDef memory fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
    classDef monitor fill:#450A0A,stroke:#FB7185,color:#FFFFFF,stroke-width:2px;
```

---

---

## Power Architecture

The board had to supply several sections with different voltage and current requirements.

Instead of treating power as a single rail, the design was organized around several electrical domains:

- **5 V** — main intermediate supply
- **3.3 V** — MCU and digital logic
- **5 V ISO** — isolated communication domain
- **1.8 V** — low-voltage interface domain

The cellular section also required special attention because of its short high-current peaks during network activity.

```mermaid
flowchart LR

    INPUT["POWER<br/>INPUT"]:::input
    PROT["INPUT<br/>PROTECTION"]:::protect
    PWR["POWER<br/>DISTRIBUTION"]:::main

    V5["5 V<br/>DOMAIN"]:::v5
    V33["3.3 V<br/>DOMAIN"]:::v33
    V18["1.8 V<br/>DOMAIN"]:::v18
    ISO["5 V ISO<br/>ISOLATED DOMAIN"]:::iso
    LTE["CELLULAR<br/>POWER PATH"]:::lte

    INPUT --> PROT --> PWR

    PWR --> V5
    V5 --> V33
    V5 --> V18
    V5 --> ISO
    PWR --> LTE

    classDef input fill:#111827,stroke:#38BDF8,color:#FFFFFF,stroke-width:2px;
    classDef protect fill:#450A0A,stroke:#FB7185,color:#FFFFFF,stroke-width:2px;
    classDef main fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:3px;

    classDef v5 fill:#422006,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;
    classDef v33 fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:2px;
    classDef v18 fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;
    classDef iso fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef lte fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
```

The main design points were:

- Keeping the different voltage domains clearly organized
- Providing local decoupling close to the loads
- Supplying enough transient current for the cellular modem
- Keeping the isolated supply separated from the main logic domain
- Minimizing high-current power paths
- Protecting the board at the power entry

  ---

## Industrial Communication & Isolation

The industrial communication interface is based on **RS-485 / Modbus RTU** and is electrically isolated from the main STM32 domain.

The isolation helps protect the logic side of the board from disturbances coming from the industrial environment.

```mermaid
flowchart LR

    MCU["STM32<br/>3.3 V DOMAIN"]:::logic

    ISO["ISO1410<br/>ISOLATED RS-485"]:::iso

    BUS["RS-485 BUS<br/>"]:::bus

    FIELD["INDUSTRIAL<br/>EQUIPMENT"]:::field

    PWR["5 V ISO<br/>SUPPLY"]:::power

    MCU <--> ISO
    PWR --> ISO
    ISO <--> BUS
    BUS <--> FIELD

    classDef logic fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:2px;
    classDef iso fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:3px;
    classDef bus fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;
    classDef field fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
    classDef power fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
```

> **Isolation was treated as both an electrical and a physical PCB constraint.**

---

## LTE Cat-1 & RF Integration

The cellular section gives EdgeGuard remote connectivity through an **LTE Cat-1 modem**.

Compared with the MCU and low-power logic, the modem introduced two important challenges:

- High transient current demand
- RF routing and antenna integration

```mermaid
flowchart LR

    MCU["STM32<br/>CONTROL CORE"]:::mcu
    MODEM["LTE CAT-1<br/>MODEM"]:::lte
    SIM["SIM / USIM<br/>INTERFACE"]:::sim
    ANT["LTE<br/>ANTENNA"]:::rf
    CLOUD["REMOTE<br/>PLATFORM"]:::cloud
    PWR["DEDICATED<br/>POWER PATH"]:::power

    MCU <--> MODEM
    SIM --> MODEM
    PWR --> MODEM
    MODEM --> ANT
    ANT <--> CLOUD

    classDef mcu fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:2px;
    classDef lte fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:3px;
    classDef sim fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef rf fill:#4C1D95,stroke:#D8B4FE,color:#FFFFFF,stroke-width:2px;
    classDef cloud fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
    classDef power fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
```

### Power Considerations

The modem can generate short current peaks during transmission, so the power path was designed with:

- Bulk capacitance close to the modem
- Local high-frequency decoupling
- Short and wide power paths
- Protection at the supply entry
- Careful return-current paths

### RF Considerations

The RF section required additional PCB constraints:

- Controlled **50 Ω antenna routing**
- Short RF traces
- Antenna keep-out area
- Continuous ground reference
- Separation from noisy power and digital sections
- Careful placement of the modem and antenna connector

> **The cellular section influenced both the power architecture and the physical PCB layout.**

---
---

## GNSS Integration

GNSS capability was integrated to allow the system to provide positioning information when needed.

The GNSS section had to be treated carefully because RF signals are sensitive to routing, grounding and nearby noise sources.

```mermaid
flowchart LR

    GNSS["GNSS<br/>RECEIVER"]:::gnss
    MCU["STM32<br/>CONTROL CORE"]:::mcu
    ANT["GNSS<br/>ANTENNA"]:::rf
    PWR["CLEAN<br/>POWER"]:::power
    DATA["POSITION<br/>DATA"]:::data

    ANT --> GNSS
    PWR --> GNSS
    GNSS --> MCU
    MCU --> DATA

    classDef gnss fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:3px;
    classDef mcu fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:2px;
    classDef rf fill:#4C1D95,stroke:#D8B4FE,color:#FFFFFF,stroke-width:2px;
    classDef power fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef data fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
```

### Design Considerations

- Short RF path between the GNSS section and antenna
- Continuous ground reference
- Separation from noisy digital and power circuits
- Careful antenna placement and keep-out
- Local decoupling close to the GNSS circuitry
- Reliable communication with the STM32

> **With GNSS, placement and routing become part of the RF design, not just the PCB layout.**

---
