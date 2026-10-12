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

---

## LTE Cat-1 Integration
The LTE modem introduced its own power behavior, logic voltage domain, SIM interface and RF path.  
Integrating these correctly was therefore a combination of **digital interfacing, power integrity and RF design**.

```mermaid
flowchart LR

    MCU["STM32<br/>CONTROL CORE"]:::mcu
    SHIFT{{"3.3 V ↔ 1.8 V<br/>LEVEL DOMAIN"}}:::shift
    LTE(["EC200U<br/>LTE CAT-1"]):::lte
    RF(["50 Ω<br/>RF PATH"]):::rf
    NET(("CELLULAR<br/>NETWORK")):::network

    PWR[("3.8 V<br/>POWER")]:::power
    SIM[("USIM<br/>IDENTITY")]:::sim

    MCU -->|"UART / AT"| SHIFT
    SHIFT --> LTE
    LTE --> RF
    RF --> NET

    PWR -->|"energy"| LTE
    SIM -->|"network access"| LTE

    classDef mcu fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:2px;
    classDef shift fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef lte fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:4px;
    classDef rf fill:#450A0A,stroke:#FB7185,color:#FFFFFF,stroke-width:2px;
    classDef network fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:3px;
    classDef power fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef sim fill:#422006,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;
```

### Power Became Part of the Communication Design

The EC200U operates around **3.8 V** and its supply must remain stable while the modem changes operating state and transmits over the cellular network.  
This influenced the power path width and the physical position of the modem power network.

<p align="center">
  <img src="../media/documentation/lte-power-star-routing.png" width="50%">
  <br><br>
  <b>Cellular Modem Power Reference</b>
  <br>
  <sub>
    LTE reference topology showing local energy storage, high-frequency
    decoupling and separate VBAT_RF / VBAT_BB paths.
  </sub>
</p>

The manufacturer reference also separates the supply toward **VBAT_RF** and **VBAT_BB**, with local bulk and high-frequency capacitors placed close to the module.

For EdgeGuard, this translated into a short, low-impedance modem supply path with local bulk capacitance and decoupling close to the cellular section.

---

### Two Logic Domains Had to Meet

The STM32 operates in a **3.3 V logic environment**, while the EC200U UART belongs to a **1.8 V domain**.

Instead of connecting both devices directly, a dedicated level-translation stage was placed between the controller and modem.

<p align="center">
  <img src="../media/documentation/lte-uart-level-shifting.png" width="50%">
  <br><br>
  <b>UART Level-Translation Reference</b>
  <br>
  <sub>
    Manufacturer reference showing the interface between the modem's
    1.8 V UART domain and an external MCU logic domain.
  </sub>
</p>

This interface provides the communication path used for **AT commands, modem control and data exchange**.

---

### Network Access Starts at the SIM Interface

The cellular link also depends on a reliable USIM interface.

The SIM signals were therefore treated as an external interface with attention to local filtering, protection and routing between the modem and SIM holder.

<p align="center">
  <img src="../media/documentation/lte-usim-interface.png" width="50%">
  <br><br>
  <b>USIM Interface Reference</b>
  <br>
  <sub>
    Reference architecture for SIM power, clock, reset and data signals,
    including filtering and protection close to the connector.
  </sub>
</p>

The EC200U supports both **1.8 V and 3.0 V USIM operation**, allowing the modem to manage the SIM supply according to the inserted card.

---

### The Last Centimeters Became RF Design

The EC200U main antenna interface is specified for **50 Ω characteristic impedance**, so the antenna connection was treated as a controlled-impedance RF path.

<p align="center">
  <img src="../media/documentation/lte-main-antenna-matching.png" width="62%">
  <br><br>
  <b>Main Antenna Matching Reference</b>
  <br>
  <sub>
    LTE reference showing the reserved π-matching network between
    ANT_MAIN and the external antenna connection.
  </sub>
</p>

This influenced trace geometry, ground continuity, component placement and the distance between the modem and antenna connection.


<br>

<p align="center">
  <sub><i>Reference: LTE Series Hardware Design and Reference Design documentation.</i></sub>
</p>

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
<br>

<h3 align="center">GNSS Design References</h3>

<p align="center">
  <sub>
    Manufacturer recommendations used as references for the antenna interface,
    impedance control and RF routing strategy.
  </sub>
</p>

<br>

<p align="center">
  <img src="../media/documentation/gnss-reference-circuit.png" width="40%">
  <br><br>
  <b>GNSS Antenna Reference Circuit</b>
  <br>
  <sub>
    Recommended antenna interface and matching network for the GNSS RF path.
  </sub>
</p>

<br>

<table>
<tr>
<td align="center" width="50%">
<b>50 Ω GNSS Interface</b><br><br>
<img src="../media/documentation/gnss-50ohm-interface.png" width="95%"><br><br>
<sub>The ANT_GNSS interface is specified with a characteristic impedance of 50 Ω.</sub>
</td>
<td align="center" width="50%">
<b>4-Layer RF Routing Reference</b><br><br>
<img src="../media/documentation/rf-4layer-cpw-reference.png" width="95%"><br><br>
<sub>Coplanar-waveguide geometry used as a reference for controlled-impedance RF routing.</sub>
</td>
</tr>
</table>

<br>

<p align="center">
  <img src="../media/documentation/gnss-antenna-requirements.png" width="52%">
  <br><br>
  <b>GNSS Antenna Requirements</b>
  <br>
  <sub>
    Manufacturer requirements covering operating frequency, VSWR, gain and antenna characteristics.
  </sub>
</p>

<br>

<p align="center">
  <sub><i>Source: Quectel Hardware Design documentation.</i></sub>
</p>

---

## Fault & Relay-State Monitoring

A key function of the board is monitoring the electrical state of external relay and fault signals.

Because these signals come from the field, they cannot be connected directly to the STM32.  
The input stage therefore conditions and protects the signal before it reaches the MCU.

```mermaid
flowchart LR

    FIELD["FIELD<br/>RELAY SIGNAL"]:::field
    PROT["INPUT<br/>PROTECTION"]:::protect
    FILTER["FILTERING &<br/>POLARIZATION"]:::filter
    BUFFER["SCHMITT<br/>BUFFER"]:::buffer
    MCU["STM32<br/>DIGITAL INPUT"]:::mcu
    LOGIC["FAULT / STATE<br/>PROCESSING"]:::logic

    FIELD --> PROT
    PROT --> FILTER
    FILTER --> BUFFER
    BUFFER --> MCU
    MCU --> LOGIC

    classDef field fill:#450A0A,stroke:#FB7185,color:#FFFFFF,stroke-width:2px;
    classDef protect fill:#431407,stroke:#FB923C,color:#FFFFFF,stroke-width:2px;
    classDef filter fill:#422006,stroke:#FACC15,color:#FFFFFF,stroke-width:2px;
    classDef buffer fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef mcu fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:3px;
    classDef logic fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:2px;
```

### Design Considerations

- Protection of the MCU from external field signals
- Input filtering to reduce noise and false transitions
- Defined logic states through signal polarization
- Clean digital transitions before the STM32 input
- Reliable detection of relay and fault states
- Separation between field-side signals and digital processing

A Schmitt-trigger buffer was used to provide clean logic transitions before the signal reached the STM32.

> **The goal was not only to detect a relay state, but to make sure the MCU receives a clean and reliable digital signal.**

---
<br>

<h3 align="center">Logic Level Translation Reference</h3>

<p align="center">
  <sub>
    The SN74LVC1T45 was used as a reference for communication between logic domains operating at different voltage levels.
  </sub>
</p>

<br>

<table>
<tr>
<td align="center" width="50%">
<b>Level-Shifting Application</b><br><br>
<img src="../media/documentation/level-shifter-application.png" width="95%"><br><br>
<sub>Typical unidirectional translation between two logic-voltage domains.</sub>
</td>
<td align="center" width="50%">
<b>Voltage Translation Waveform</b><br><br>
<img src="../media/documentation/level-shifter-waveform.png" width="95%"><br><br>
<sub>Example waveform showing logic translation from 1.8 V to 5 V.</sub>
</td>
</tr>
</table>

<br>

<p align="center">
  <sub><i>Source: Texas Instruments — SN74LVC1T45 Datasheet.</i></sub>
</p>

---

## Local Storage & EEPROM

Local non-volatile memory was included so that important information could remain available even when communication with the remote platform was temporarily unavailable.

The STM32 can use the EEPROM to store information such as system parameters, events or diagnostic data.

```mermaid
flowchart LR

    MCU["STM32<br/>CONTROL CORE"]:::mcu
    EEPROM["NON-VOLATILE<br/>EEPROM"]:::mem
    CONFIG["SYSTEM<br/>PARAMETERS"]:::data
    EVENTS["EVENT / FAULT<br/>RECORDS"]:::fault
    REMOTE["REMOTE<br/>COMMUNICATION"]:::cloud

    CONFIG --> MCU
    MCU <--> EEPROM
    EVENTS --> EEPROM
    MCU <--> REMOTE

    classDef mcu fill:#172554,stroke:#60A5FA,color:#FFFFFF,stroke-width:3px;
    classDef mem fill:#052E16,stroke:#4ADE80,color:#FFFFFF,stroke-width:3px;
    classDef data fill:#083344,stroke:#22D3EE,color:#FFFFFF,stroke-width:2px;
    classDef fault fill:#450A0A,stroke:#FB7185,color:#FFFFFF,stroke-width:2px;
    classDef cloud fill:#2E1065,stroke:#C084FC,color:#FFFFFF,stroke-width:2px;
```

### Design Considerations

- Non-volatile storage for information that must survive a power cycle
- Communication between the STM32 and external memory
- Controlled read and write operations
- Avoiding unnecessary memory writes
- Keeping local information available during communication loss
- Possibility of storing configuration and diagnostic information

> **Local storage gives the system a memory even when the network is unavailable.**

---
<br>

<h3 align="center">EEPROM Read / Write References</h3>

<p align="center">
  <sub>
    Reference transactions used to understand page-oriented storage and sequential data access.
  </sub>
</p>

<br>

<table>
<tr>
<td align="center" width="50%">
<b>64-Byte Page Write</b><br><br>
<img src="../media/documentation/eeprom-page-write.png" width="95%"><br><br>
<sub>Page-write sequence used to store multiple bytes in a single EEPROM write operation.</sub>
</td>
<td align="center" width="50%">
<b>Sequential Read</b><br><br>
<img src="../media/documentation/eeprom-sequential-read.png" width="95%"><br><br>
<sub>Sequential access allows consecutive memory locations to be read efficiently.</sub>
</td>
</tr>
</table>

<br>

<p align="center">
  <sub><i>Source: Microchip — 24AA256 / 24LC256 / 24FC256 Datasheet.</i></sub>
</p>


## Signal Integrity & Controlled Impedance

HyperLynx was used to reproduce the physical PCB stackup and study the electromagnetic behavior of critical communication traces.

The objective was not only to calculate impedance values, but also to visualize how trace geometry, reference planes and spacing influence field distribution and coupling.

### PCB Stackup Model

The physical **JLC04161H-3313** stackup used in the PCB design was reproduced in HyperLynx.

The model included:

- **4 copper layers**
- **35 µm** outer copper
- **15.2 µm** inner copper
- **99.4 µm** prepreg between TOP and GND
- **1.265 mm** FR-4 core
- **L2 GND** as the primary reference plane for top layer controlled impedance traces
- Total modeled thickness of approximately **1.584 mm**

<p align="center">
  <img src="../media/documentation/hyperlynx-stackup-jlc04161h-3313.png" width="55%">
  <br><br>
  <b>JLC04161H-3313 Stackup Model</b>
  <br>
  <sub>Physical 4-layer PCB construction reproduced in HyperLynx.</sub>
</p>

---

### 50 Ω Trace Coupling Analysis

The 50 Ω top layer traces were evaluated using their real controlled impedance geometry.

The main trace parameters were:

- Trace width: **169.63 µm**
- Reference plane: **L2 GND**
- TOP-to-GND dielectric thickness: **99.4 µm**
- Dielectric constant: approximately **4.1**
- Actual trace separation: **6 mm**

The field solver shows that, with the real 6 mm spacing, most of the electromagnetic field remains concentrated around each trace and its nearby ground reference.

<p align="center">
  <img src="../media/documentation/hyperlynx-field-coupling-6mm.png" width="55%">
  <br><br>
  <b>Field Distribution — 6 mm Trace Spacing</b>
  <br>
  <sub>Field solver visualization of the controlled-impedance traces using the actual PCB spacing.</sub>
</p>

To understand the influence of spacing, the same geometry was also evaluated with a reduced separation between the traces.

<p align="center">
  <img src="../media/documentation/hyperlynx-field-coupling-reduced-spacing.png" width="55%">
  <br><br>
  <b>Field Distribution — Reduced Trace Spacing</b>
  <br>
  <sub>Comparison case showing stronger electromagnetic interaction when the traces are brought closer together.</sub>
</p>

The comparison illustrates an important routing principle:

> **Increasing the separation between adjacent controlled-impedance traces reduces electromagnetic interaction and helps limit crosstalk.**

---

### RS485 Differential Impedance Validation

The RS485 A/B pair was also reproduced using its actual routing geometry.

The simulated differential pair used:

- Trace width: **66.82 µm**
- Trace gap: **127 µm**
- Coupled length: approximately **27 mm**
- Routing layer: **TOP**
- Reference plane: **L2 GND**
- Target differential impedance: **120 Ω**

Altium calculated approximately **119.95 Ω**, while the HyperLynx field solver produced approximately **117.3 Ω**.

This corresponds to only about **2.25% deviation** from the 120 Ω target.

<p align="center">
  <img src="../media/documentation/hyperlynx-rs485-differential-field.png" width="55%">
  <br><br>
  <b>RS-485 Differential Field Distribution</b>
  <br>
  <sub>Electromagnetic field distribution of the A/B pair referenced to the L2 ground plane.</sub>
</p>

Unlike the widely separated 50 Ω traces, electromagnetic interaction between the RS485 conductors is intentional and contributes to the differential behavior of the pair.

> **The simulation confirmed that the implemented RS485 geometry remained close to the 120 Ω differential impedance target.**



