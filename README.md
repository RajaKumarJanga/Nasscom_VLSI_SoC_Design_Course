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
  <em>Figure 1: OpenLane_design_flow </em>
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
  <em>Figure 2: OpenLane_commands </em>
</p>

## Command to run synthesis
```bash
run_synthesis
```
After synthesis, we calculate the flop ratio for a sanity check.

## What is flop ratio?
It is the ratio of D Flip-Flops to the total number of cells.
```
Flop ratio = (value of dxftp/total number of cells) * 100
           = (1613/15762)*100 = 10.233%
```
<p align="center">
  <img src="images/day1/flop_ratio.png" width="1000">
  <br>
  <em>Figure 3: Number of dxftp cells & toal number of cells </em>
</p>

<a id="day-2"></a>
## Day 2 - Chip Floorplanning, Library Cells and Standard Cell Placement


## Chip Floorplanning — Core Area and Utilization

Floorplanning determines the **size, shape, and organization of the chip's core area** and establishes where major design components will be placed.

Two important parameters are:

- **Utilization Factor** = Area Occupied by Netlist / Total Core Area
  - A utilization factor of approximately **0.5–0.6** is commonly used to leave sufficient space for buffers, routing, and other physical design requirements.

- **Aspect Ratio** = Core Height / Core Width
  - An aspect ratio of **1** represents a square core, while other values result in a rectangular core.

---

## Pre-Placed Cells and Decoupling Capacitors

**Pre-placed cells**, such as memories, PLLs, and complex IP blocks, are positioned and fixed before automated placement begins.

Their locations are determined based on factors such as **connectivity and power requirements**.

**Decoupling capacitors (decaps)** are placed near these cells to act as **local charge reservoirs**. They help compensate for voltage fluctuations caused by switching activity and provide a more stable power supply to nearby circuits.

---

### Power Planning — Mesh and Ring

Power planning ensures reliable distribution of **VDD and VSS** throughout the chip.

A typical power distribution network consists of:

- **Power Rings** — Surround the core and provide a strong VDD/VSS supply path.
- **Power Mesh** — Distributes power across the core using horizontal and vertical metal layers.

Together, the power ring and mesh provide nearby power connections to standard cells, helping reduce **IR drop** and **electromigration (EM) risk**.

---

### Pin Placement and Logical Cell Blockage

Input and output pins are placed along the **chip boundary**, with their locations determined primarily by design connectivity.

Pins are positioned close to the logic they communicate with whenever possible to improve routing efficiency.

The region between the **core boundary and die boundary** is typically restricted from standard-cell placement. This placement blockage reserves space for I/O-related structures and prevents automated placement from using areas intended for boundary-level resources.

### Lab - Floorplan and Placement

## Command to run floorplan
```bash
run_floorplan
```
<p align="center">
  <img src="images/day2/floorplan_command.png" width="1000">
  <br>
  <em>Figure 4: Running floorplan  </em>
</p>

## What is Die Area?
It is the entire chip area.

<p align="center">
  <img src="images/day2/Die_Area.png" width="1000">
  <br>
  <em>Figure 5: Die_Area  </em>
</p>

After this completes, we can inspect the DEF file that was generated:

```bash
cd results/floorplan/
less picorv32a.def
```

## Command to view floorplan in Magic

```bash
magic -T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech \
  lef read ../../tmp/merged.nom.lef \
  def read picorv32a.def &
```

<p align="center">
  <img src="images/day2/floorplan.png" width="1000">
  <br>
  <em>Figure 6: Floorplan layout  </em>
</p>

<p align="center">
  <img src="images/day2/zoom_floorplan.png" width="1000">
  <br>
  <em>Figure 7: Zoomed version of floorplan layout  </em>
</p>

<p align="center">
  <img src="images/day2/floorplan_equidispins.png" width="1000">
  <br>
  <em>Figure 8: IO pins equidistance view  </em>
</p>

<p align="center">
  <img src="images/day2/standardcells.png" width="1000">
  <br>
  <em>Figure 9: Standard cells  </em>
</p>

## Command to run placement

```bash
run_placement
```

## Command to view placement in Magic

```bash
magic -T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech \
  lef read ../../tmp/merged.nom.lef \
  def read picorv32a.def &
```

<p align="center">
  <img src="images/day2/placement_command.png" width="1000">
  <br>
  <em>Figure 10: Running placement </em>
</p>

<p align="center">
  <img src="images/day2/placement.png" width="1000">
  <br>
  <em>Figure 11: Placement layout </em>
</p>

<p align="center">
  <img src="images/day2/Zoom_standcellplaceview.png" width="1000">
  <br>
  <em>Figure 12: Zoomed version of Placement layout showing standard cells </em>
</p>

---

<a id="day-3"></a>
## Day 3 - Design and Characterization of Standard Cells using Magic and ngspice


### CMOS Inverter — SPICE Deck

To characterize a **CMOS standard cell**, a SPICE netlist is created to describe the electrical behavior of the circuit.

The SPICE deck typically includes:

- **PMOS and NMOS transistor models**
- **Transistor dimensions (W/L ratios)**
- **Supply voltage (VDD)**
- **Input stimulus**
- **Output load capacitance**

The SPICE simulation transient response evaluates the cell's timing characteristics.

#### Key Timing Parameters

- **Rise Time** — Time taken for the output voltage to transition from **20% to 80%** of its final value.
- **Fall Time** — Time taken for the output voltage to transition from **80% to 20%** of its final value.
- **Propagation Delay** — Time difference between the **50% transition point of the input** and the corresponding **50% transition point of the output**.

---

### 16-Mask CMOS Fabrication Process — Brief Overview

CMOS fabrication involves a sequence of masking, deposition, oxidation, implantation, and etching steps to create the transistors and their interconnections on a silicon substrate.

A simplified overview of the process is:

1. **Substrate Selection**  
   - Begin with a **p-type, high-resistivity silicon substrate**.

2. **Active Region Formation**  
   - Define the active regions using **field oxidation** and a **Si₃N₄ mask**.

3. **N-Well and P-Well Formation**  
   - Create the required wells using **ion implantation**.

4. **Gate Oxide Formation**  
   - Grow a thin layer of **SiO₂** to form the gate oxide.

5. **Polysilicon Gate Formation**  
   - Deposit and pattern **polysilicon** to create the transistor gates.

6. **Source and Drain Formation**  
   - Form the source and drain regions using implantation techniques such as **LDD and halo implantation**.

7. **Contact and Metal Formation**  
   - Create contacts and deposit metal layers to electrically connect the transistors.

8. **Final Passivation**  
   - Apply a protective **passivation layer** over the completed chip to protect the circuitry from contamination and physical damage.


## Lab — Cloning and Characterizing a Custom Inverter Cell

### Cloning the Standard Cell Repository

```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

```
magic -T sky130A.tech sky130_inv.mag &
```


<p align="center">
  <img src="images/day3/vsdstdcell_files.png" width="1000">
  <br>
  <em>Figure 13: Files inside vsdstdcell </em>
</p>

<p align="center">
  <img src="images/day3/inverter_layout.png" width="1000">
  <br>
  <em>Figure 14: Custom_Inverter layout </em>
</p>

<p align="center">
  <img src="images/day3/spiceext_commands.png" width="1000">
  <br>
  <em>Figure 15: Commands to extract a SPICE file </em>
</p>

<p align="center">
  <img src="images/day3/extr_spice.png" width="1000">
  <br>
  <em>Figure 16: Extracted SPICE file </em>
</p>

<p align="center">
  <img src="images/day3/update_spice.png" width="1000">
  <br>
  <em>Figure 17: Updated SPICE file </em>
</p>

### Command to run ngspice simulation

```
ngspice sky130_inv.spice
```

```
plot y vs time a
```

<p align="center">
  <img src="images/day3/ngspice_commands.png" width="1000">
  <br>
  <em>Figure 18: Commands to start ngspice </em>
</p>

<p align="center">
  <img src="images/day3/ngspice_plot.png" width="1000">
  <br>
  <em>Figure 19: Plot y vs. time a </em>
</p>

<p align="center">
  <img src="images/day3/rise_trns_20per.png" width="1000">
  <br>
  <em>Figure 20: 20% of the maximum value @ Rising transition </em>
</p>

<p align="center">
    <img src="images/day3/rise_trns_20per_plot.png" width="1000">
    <br>
    <em>Figure 21: Waveform </em> 
</p>

<p align="center">
  <img src="images/day3/rise_trns_80per.png" width="1000">
  <br>
  <em>Figure 22: 80% of the maximum value @ Rising transition </em>
</p>

<p align="center">
    <img src="images/day3/rise_trns_80perplot.png" width="1000">
    <br>
    <em>Figure 23: Waveform </em> 
</p>

From the waveform, the rising transition time is as follows: 

```bash
Rise transition time = Time taken for output to rise to 80% - Time taken for output to rise to 20%
                     = 2.245798ns - 2.24579ns = 0.064ns
```

<p align="center">
  <img src="images/day3/fall_trns_80per.png" width="1000">
  <br>
  <em>Figure 23: 80% of the maximum value @ Falling transition </em>
</p>

<p align="center">
    <img src="images/day3/fall_trns_80perplot.png" width="1000">
    <br>
    <em>Figure 24: Waveform </em> 
</p>

<p align="center">
  <img src="images/day3/fall_trns_20per.png" width="1000">
  <br>
  <em>Figure 24: 20% of the maximum value @ Falling transition </em>
</p>

<p align="center">
    <img src="images/day3/fall_trns_20perplot.png" width="1000">
    <br>
    <em>Figure 25: Waveform </em> 
</p>

From the waveform, the falling transition time is as follows: 

```bash
Fall transition time = Time taken for output to fall to 20% - Time taken for output to fall to 80%
                     = 8.05211ns - 8.0951ns = 0.04ns
```

Cell raise Delay(Propagation delay):
<p align="center">
  <img src="images/day3/cell_rise_delay.png" width="1000">
  <br>
  <em>Figure 25: 50% of the input value & 50% of rising output </em>
</p>

<p align="center">
    <img src="images/day3/cell_rise_delay_plot.png" width="1000">
    <br>
    <em>Figure 25: Cell rise delay waveform </em> 
</p>

From the waveform, the cell rise delay is as follows: 

```bash
Cell rise delay(propagation delay)  = Time taken for output to rise to 50% - Time taken for input to reach 50% of its value
                     = 6.21018ns - 6.14965ns = 0.061ns
```


Cell fall Delay(Propagation delay):
<p align="center">
  <img src="images/day3/cell_fall_delay.png" width="1000">
  <br>
  <em>Figure 25: 50% of the input value & 50% of falling output </em>
</p>

<p align="center">
    <img src="images/day3/cell_fall_delay_plot.png" width="1000">
    <br>
    <em>Figure 25: Cell fall delay waveform </em> 
</p>

From the waveform, the cell fall delay is as follows: 

```bash
Cell fall delay(propagation delay)  = Time taken for output to fall to 50% - Time taken for input to reach 50% of its value
                     = 8.07728ns - 8.05014ns = 0.02714ns
```







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


