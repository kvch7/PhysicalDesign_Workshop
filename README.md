# PhysicalDesign_Workshop
A tour to the world of Physical Design using open-sourced EDA tools

## **Introduction:** 

This workshop basically walks us through the various steps involved in the Physical Design Flow of an SOC design, also introducing  the various open-sourced EDA tools involved in each step. It unfolds with the introduction of IC design terminologies, SOC based designs and moves forwards touching the various phases of PD flow starting from synthesis to routing and DRCs. 

## **1. Introduction to IC Design terminology and RISC-V based picoSoc:**
### 1.1 IC design terminologies: 
* **Package:** The package that is taken as an example is the QFN-48 and its size is 7mmx7mm. The QFN here, stands for Quad Flat No-leads and 48 is the total number of pins in the package.
* **Chip:** The chip mainly sits at the centre of the package. The pins of the package are connected to the chip and thus information gets transmitted to and fro from the chip and outer world.
* **Pads:** One of the components of the Chip through which information gets exchanged.
* **Die:** It is the overall area of the chip that gets manufactured on silicon wafer.
* **Core:** Its the area where all the digital logic of the chip is placed.

![](wrkshp_img/ic_term.PNG)


### 1.2 RISC-V based SOC: 
Each chip has its core, the core considered here is of RISC-V based Soc. 
RISC-V is an instruction set architecture, that helps communicating with the computer. There are many flavours of RISC-V here we will be using rv32, 32 bit instruction set. The  riscv implementation used is picorv32. The picorv32 is open-sourced and is implemented by Clifford Wolf. The picorv32 is used as a component in both picoSoc and the RavenSoc. The difference between the Raven and the pico Soc is the picoSoc is targetted towards the embedded systems and the components such as SRAM is external in RAvenSoc. 

![](wrkshp_img/picorv32.PNG)

We will be using the picorv32 design for the various stages the Physical Design Flow. 

## 2. Introduction to Open-Source EDA Tools:
### 2.1 Tools used in this workshop:
* yosys: synthesis
* graywolf: placement
* qrouter: routing
* magic: layout tool
* ngspice: circuit simultaion tool
* Opentimer and OpenSTA: STA tool

### 2.2 Introduction to Qflow:
Qflow is a complete tool chain for synthesizing digital circuits starting from verilog source and ending in physical layout for a specific target fabrication process. This tool has a GUI interface that helps connect as well as simplify the process of running synthesis, place and route also generating a specific log file for each step performed. 

For detailed information regarding the Qflow please refer [here](http://opencircuitdesign.com/qflow/) 

## 3. Synthesis 
During the synthesis phase, the .v file that is being received from the front-end design is converted to a gate level netlist. The synthesis phase is a combination of a number of steps transtation + logic optimization + technology mapping => gate level netlist
### 3.1 Lab Instances: 
We will be using the qflow gui to navigate through the phases of the PD.

Command to run gui: **qlow gui &**

#### 3.1.1 Preparation:
In the gui, we do the following settings to prep our design for the synthesis phase.
* technology : 180nm
* verilog source file: picorv32.v
* module name : picorv32

![](wrkshp_img/synthesis_prep.PNG)

After the prep stage we run the synthesis phase, this generates a log file, a snapshot of which is attached. The snapshot shows the statistics, the total number of cells used, total DFFs. We can find the **total logic used** in the design by **total no. of DFFs used/total number of cells**  

![](wrkshp_img/synth_report.PNG)

## 4. Floorplaning Considerations
From the netlist generated from the synthesis phase, the following aspects are considered: 
* Width and height of core and die:
* Preplaced cells
* Decoupling capacitors
* Power Planing
* Pin Placement
* Cell Blockage
After considering all this aspects the design gets ready for the Placement and routing phase.

### 4.1 Lab Instances:
For the floorplaning labs, 
* We are placing the clock and the resetn pin at the bottom of the picorv32 design. Steps involved in the qflow gui are as follows:
 * click on **arrange pins --> auto group --> apply**
 * click on **arrange pins --> create group my_pins --> drag the pins clk and resetn to my_pins section --> check the fixed box and bottom box --> apply**

 ![](wrkshp_img/pin_placement.PNG)

* set the utilization factor as 0.70 and aspect ratio 1 indicating that we are considering a square shape and run the placement flow. 

 ![](wrkshp_img/floorplan_lab.PNG)

## 5. Placement
In this phase:
* Initially, the gates of the netlist are binded to the standard cells and are placed on the floorplan that was designed considering the parameters mentioned in the previous section
* Optimizing placement, here the repeaters/buffers are inserted based on the estimation of wire length and capacitance.

## 6. Introduction to standard cell libraries
Libraries are common across all the stages of the physical design flow. 
The standard cells included in the library includes
* different functionality
* varying sizes
* different threshold voltages
* different drive strength etc.
### 6.1 Cell Design Flow:
Design of a single cell of the library has the following Flow:
* INPUTS: 
  * PDKs: 
    * DRC and LVS rules: these information is provided by the foundries
    * Spice model Parameters
    * user defined spec: e.g cell height, width, supply voltage, pin locations
* Design Steps: 
  * Circuit design: here we implement the function using the spice simulation 
  * Layout design: euler's path and stick diagram concepts are utilized for layouts 
  * characterization: 
* Outputs:
  * CDL
  * extracted spice netlist
  * timing,noise, power .libs
  
### 6.2 Lab Instances:
**Experiment 1:** We did ngspice simultaions of a inverter for both dc and transient analysis
* commands to run ngspice: 
  
  <div class="bg-blue-light mb-2">
  
  ngspice inv.spice
  
  ngspice 1 -> run
  
  ngspice 1 -> setplot dc1
  
  ngspice 1 -> plot out
  
* We did ngspice simulations for pmos size 0.5 and 0.75 and observed the shift in the Vm i.e the intersection point of the blue and red lines.

![](wrkshp_img/inv_dc_0.5.PNG "for pmos size 0.5")           ![](wrkshp_img/ngspice_invdc_75.PNG "for pmos size 0.75")

* Similarly we did ngspice simulation for transient analysis of a inverter and calculated rise delay for pmos width 0.75

![](wrkshp_img/rise_delay_calc.PNG)

**Experiment 2:** We did this lab for understanding the layout concepts, ran spice simulation for both pre layout spice and post layout spice and observed the difference in the PULSE WIDTH. Also the area consumed by the layout. 
* Steps/commands involved:
  * cd ngspice_labs
  * magic -T min2.tech fn_postlayout.mag &
  * select the area of the layout and type **"box"** in the tkcon window. this steps gives the area covered in the layout in microns
  * extract the parasitics of the layout using the command **"extract all"** in the tkcon window.
  * convert the extracted files to spice using **"ext2spice"** this creates a fn_postlayout.spice file in the directory.
  * In the pre_layout.spice files change the width and length of pmos according to the scaling factor in the post layout file and copy the simulations commands from pre to the post layout files. Now run spice simulations for both the files and observed the change in their pulse width. 

![](wrkshp_img/Area_layout.PNG)
![](wrkshp_img/para_extractPNG.PNG)

* In this image the left is pre-layout spice simulation and right is the post layout simulation:
![](simu.PNG)


## 7. Timing Analysis using OpenTimer: 
* Introduced to the concept of delay tables. The delay tables contain the delay of a cell w.r.t input slew and output load.
* Set up and hold time w.r.t ideal and real clocks. In the case of real clocks, the concept of clock jitter and clock skew comes to the picture.
* Slack is met when the condition data required - data arrival time is positive i.e. no timing violation.
### 7.1 Lab instances: 
For timing analysis Opentimer tool is used. Steps involved
* the tool reads a std_cell.lib which is mentioned in the configuration file. A snapshot of this library is shown:
![](stdcell_lib.PNG)

* create sdc file: here a clock constraint is given to the analysis tool to perform the timing analysis
![](wrkshp_img/sdc.PNG)

* set the sta configuration file: 
![](wrkshp_img/sta_confg.PNG)

* run the sta tool: command used **sta prelayout_sta.conf**
* check the set-up slack : the set up report generated is shown below, it contains all the information such as DAT, DRT, library time, delay in each cell 
![](wrkshp_img/slack.PNG)

* check hold time slack: command used: **report_checks -path_delay min -digits 4**
![](wrkshp_img/hold_check.PNG)

## 8. Routing and DRC: 
### 8.1 Routing:
* routing means finding the best possible way to connect two components/cells.
* most of the routing tools are based on the Lee's Algorithm. For more info on the algorithm please refer [here](https://www.vlsisystemdesign.com/maze-routing-lees-algorithm/) 
* Routes with minimum amount of bends is prefered by the tool
### 8.2 DRC (Design Rule Check):
* Wires are made using optical lithography techniques so, most of the design rules comes from the lithography point of view. 
* Some of DRC rules for a pair of Wires: 
  * Wire pitch
  * Wire Space
  * Wire Width
* Physical wires have resistance and capacitance and these information are extracted and represented in the form of a file format called **SPEF**
### 8.3 Lab instances:
In this lab, the routing stage is performed using the qflow, and the frequency of prelayout and postlayout is compared.  
* Command used :
  * **qflow route picorv32**
  * **qflow sta picorv32**
  * **qflow backanno picorv32**

* Qrouter run and log file: 
![](wrkshp_img/route_commnd.PNG)
![](wrkshp_img/qroute_pico.PNG)
![](qroute_log.PNG)

* Prelayout freq check: For this purpose we look into the **log/sta.log** file 
![](wrkshp_img/prelayout_freq.PNG)

* Post layout freq check: For this purpose we look into the **log/post_sta.log** file
![](wrkshp_img/post_sta.PNG)

* **Observation:** Post layout we noticed that the design got slower comapred to the  pre layout the possible reason is that the addition of parasitics in the post layout. 

## Acknowledgement
* Kunal Ghosh, Co-founder (VSD Corp. Pvt. Ltd)



