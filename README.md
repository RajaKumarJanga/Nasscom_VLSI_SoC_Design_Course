# Digital VLSI SoC Design and Planning — RTL to GDSII
This repository documents the transformation of the **PicoRV32A RISC-V core** from synthesizable RTL into a fabrication-ready GDSII layout. It combines physical-design theory, reproducible OpenLane labs, custom-cell characterization, timing analysis, and practical debugging into one continuous engineering narrative.

## Why this repository is different

Physical design is easier to understand when every concept is connected to an action and every action leaves evidence. Each stage in this repository therefore follows four questions:

| Lens | Question |
|---|---|
| **Concept** | What physical-design problem are we solving? |
| **Action** | Which command or tool performs the work? |
| **Evidence** | Which report, waveform, DEF, LEF, or layout proves the result? |
| **Insight** | What should an engineer verify before moving forward? |

The goal is not merely to complete the flow, but to understand how early design decisions affect area, timing, power, congestion, and manufacturability later.

---

## The silicon journey

```mermaid
flowchart TD
    A["RTL design"] --> B["Logic synthesis"]
    B --> C["Floorplan"]
    C --> D["Placement"]
    D --> E["Clock tree synthesis"]
    E --> F["Power grid and routing"]
    F --> G["Parasitic extraction and STA"]
    G --> H["DRC, LVS and GDSII"]
```

| Stage | Primary question | Main artifact |
|---|---|---|
| RTL | Does the logic describe the intended behavior? | Verilog source |
| Synthesis | Can the logic be mapped to real cells? | Gate-level netlist |
| Floorplan | Can the design fit and connect efficiently? | Floorplan DEF |
| Placement | Where should each standard cell be located? | Placement DEF |
| CTS | Can the clock reach sequential cells with controlled skew? | CTS netlist/DEF |
| Routing | Can every net be physically connected? | Routed DEF |
| Sign-off | Is the design timed, legal, and manufacturable? | Reports and GDSII |

---

## Workshop roadmap

- [Day 1 - Introduction to Open-Source ASIC Design, OpenLane and Sky130 PDK](#day-1) 

- [Day 2 - Chip Floorplanning, Library Cells and Standard Cell Placement](#day-2)

- [Day 3 - Design and Characterization of Standard Cells using Magic and ngspice](#day-3)

- [Day 4 - Pre-Layout Timing Analysis and Clock Tree Synthesis](#day-4)

- [Day 5 - Final RTL to GDSII Flow: Power Distribution, Routing and Post-Route Timing Analysis](#day-5)

- [Tools & Environment](#tools-environment)
  
- [Key Learnings](#key-learnings)
 
- [Acknowledgements](#acknowledgements)

- [References](#references)

---

## Toolchain

| Tool | Role in the flow |
|---|---|
| **OpenLANE** | Orchestrates the RTL-to-GDSII flow |
| **Yosys** | RTL synthesis |
| **ABC** | Technology mapping and logic optimization |
| **OpenROAD** | Floorplanning, placement, CTS, PDN, and routing |
| **Magic** | Layout viewing, extraction, and DRC |
| **ngspice** | Transistor-level simulation |
| **OpenSTA** | Static timing analysis |
| **Netgen** | Layout-versus-schematic comparison |
| **SKY130A PDK** | Process rules, device models, and cell libraries |

---

<a id="day-1"></a>
## Day 1 - Introduction to Open-Source ASIC Design, OpenLane and Sky130 PDK

## What is inside a chip?

An integrated circuit contains a silicon **die**, a central **core** for logic, and an I/O region that connects internal signals to the package. The core may contain:

- **Standard cells** — reusable logic gates and sequential elements.
- **Macros** — larger predesigned blocks such as memories.
- **Foundry IP** — process-dependent blocks such as SRAMs, PLLs, and analog structures.

At RTL, the design expresses behavior and register transfers. Synthesis converts that description into interconnected cells selected from a technology library. This gate-level netlist bridges logical design and physical implementation.

## Why does an open PDK matter?

A Process Design Kit connects circuit design to a real manufacturing process. It provides design rules, device models, layer definitions, extraction data, and characterized standard-cell libraries. The SKY130 PDK enables an end-to-end educational ASIC flow without proprietary design data.


The OpenLane framework integrates multiple open-source EDA tools into a single automated ASIC implementation flow.
<p align="center">
  <img src="images/day1/OpenLane_design_flow.png" width="900">
  <br>
  <em>Figure 1: OpenLane_design_flow.png </em>
</p>

## Lab — Running OpenLane for picorv32a

```bash
cd /home/vscode/Desktop/OpenLane
make mount
./flow.tcl -interactive
package require openlane 1.0.2
```
<p align="center">
  <img src="images/day1/OpenLane_commands.png" width="1000">
  <br>
  <em>Figure 1: OpenLane_commands.png </em>
</p>


```bash
run_synthesis
```

## What is flop ratio?
It is the ratio of standard cells to the total number of cells.
```
Flop ratio = (value of dxftp/total number of cells) * 100
           = (1613/15762)*100 = 10.233%
```
<p align="center">
  <img src="images/day1/flop_ratio.png" width="1000">
  <br>
  <em>Figure 1: dxftp & toal number of cells.png </em>
</p>
<a id="day-2"></a>


## Day 2 - Chip Floorplanning, Library Cells and Standard Cell Placement

```bash
run_floorplan
```

```bash
run_placement
```
Day 2 content goes here.

---

<a id="day-3"></a>
## Day 3 - Design and Characterization of Standard Cells using Magic and ngspice

Day 3

---

<a id="day-4"></a>
## Day 4 - Pre-Layout Timing Analysis and Clock Tree Synthesis

Day 4

--- 

<a id="day-5"></a>
## Day 5 - Final RTL to GDSII Flow: Power Distribution, Routing and Post-Route Timing Analysis

Day 5

--- 

<a id="tools-environment"></a>
## Tools & Environment

--- 

<a id="key-learnings"></a>
## Key Learnings

--- 

<a id="acknowledgements"></a>
## Acknowledgements

--- 

<a id="references"></a>
## References


