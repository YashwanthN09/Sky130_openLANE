# Sky130_OpenLANE
# Description : 

This repository is the collection of steps required to convert RTL netlist to GDSII form, using an automated/interactive flow called OpenLANE. This flow includes usage of various tools like Yosys, Magic, OpenROAD, SPEF-Extractor etc., along with custom made scripts for flow optimization and easeness. The goal of OpenLANE is to produce clean GDSII without any human intervention. Currently, OpenLANE is tuned for Skywater 130nm open-source PDK and the same is used in this process, explained below.

More information on OpenLANE and setup installation can be found here https://github.com/efabless/openlane

# RTL to GDSII or Physical Design
Physical Design is a very crucial and time taking portion of Chip Design life cycle, involving tidious processes like synthesis, place and route, clock tree synthesis etc. It start's with  the code of a design in VHDL or Verilog, simulating the coded design, followed by design synthesis and optimization, then running equivalency checks at different stages and then synthesizing the design, floorplan, place-and-route the synthesized netlist while meeting timing, and finally, writing out a GDSII file.


As said earlier, OpenLANE is an automated flow and we can also use it interactively, as in we can perform the flow step by step as below.

1) Synthesis
2) Floorplan
3) Placement
4) CTS
5) Power Distribution Network
6) Routing
7) DRC & LVS sign off

First things first, perform the installation steps and setup the terminal.

Assuming 'docker' step is finished, (mentioned in the above github link)


      % ./flow.tcl -design <design_name>
 
 will run the entire steps and gives us the final output. 
 
 But, if we want to understand the steps happening and to fine tune the results as per our needs, we can run the same steps interactives, i.e., one after the other ensuring our requirements are met(like design metrics). For interactive flow,
 
    % ./flow.tcl -interactive
 
 this triggers the tools and we can confirm this by seeing the symbol as shown
 
       %
 
 Now, we need to import the package required using the below command
 
    % package require openlane 0.9
 
 After this, the needs the design name which we are interested in, and also give a tag(default tag - time_stamp) to it, to store and identify the results, reports, logs etc related to that particular design in that particular run. 
 
    % prep -design <design_name> -tag <tag_name>
 
 tags can be found in 
 
    % /designs/<design_name>/runs/
 
 All the configurations can be found in 
 
     % designs/<design_name>/config.tcl 
 
 and can be configured as per requirement. These variables can be changed using tcl command 'set' and we can confirm using 'echo' and the list of variable with which we can tune our analysis is provided in the read_me file.
 
 one we have set all our variables, we can start the process using the tcl proc's which are already predefined in scripts folder
 
 # Synthesis 
    % run_synthesis
 
 This performs the synthesis of the netlist and gives the first hand report of the design including the basic details like count of instances, cells, wires and the timing details.
 
 # Floorplan
    % run_floorplan
 
 This performs the floorplan of the synthesised netlist, giving us a floorplan.def file. By this time we have configured many parameters like
 
Die Area
Core Area
Core Utilization
Aspect Ratio

by using the variables avaiable. (described in read_me file in main directory)

We can actually see the floorplan made and locate the IO pins placed using magic tool

    % magic -T <.tech file path> lef read <.lef file path(in tmp folder)> def read <.def file path>

Satisfied with the plan, next step is placement.

# Placement

    % run_placement

This performs the placement of the cells, giving us the updated def file with extension, placement.def This evaluates the design, checks the legality of the design and converges the placement to the best possible form.

We can actually see the placement made using magic tool

    % magic -T <.tech file path> lef read <.lef file path(in tmp folder)> def read <placement.def file path>

The beauty of the flow is its ebility to make modifications on the fly. In case we need to modify any cell or any part of the design or any parameter of the design, we just need to update the respective variable and re-run the steps. In case we need to improve the performance of any particular cell we can always use ngspice tool to analysis the performance by doing DC analysis or Transient analysis using the spice tool. Thereafter we can write a new verilog file from the tool itself and update the netlist. In this case we need to re-run the floorplan and placement stages as we are modifying the netlist after placement. 

Once we are good to go with the placement, next step is Clock Tree synthesis.

# CTS

This is performed using the below command.

    % run_cts

Once, it finishes the analysis, it generates the timing report and lists out the timing violations, if any. And the best part is it shows Hold and Setup violations seperately. If there are any setup violations, we can improve the timing numbers by increasing the buffer strength of huge fanout buffers, using openSTA tool. This can be done in two ways, either in OpenLANE flow using OpenRoad or outside OpenLANE flow using OpenSTA tool. 

# Power Distribution Network

    % gen_pdn

Generating power distribution network is the next step.

# Routing

    % run_routing

After power distribution network is generated, routing is to be done, where it is done in 2 steps. 1) Global routing and 2) Detailed routing.

Here we get an option to change the quality of the result (QoR) levels from 0 - 14, where a QoR of 14 gives us the best routing solution with 0 DRC violations, but consuming more memory and time for the run to finish.

# SPEF Extraction

Once routing is finished, we need to extract the parasitics using SPEFEXTRACTOR, which is outside the OpenLANE flow. For this following command is to be used from the SPEFEXTRACTOR directory.

    % python.3 main.py <lef path> <routing.def path>
  
main.py does the work and gives us teh spef file
  

# GDSII

Once all steps are finished, we need to sign-off doing post-route STA and generate GDSII files using

    % run_magic

Done!

We are done with our Physical Design steps of our chip.

#

Many Thanks to Mr. Kunal Ghosh and his team.
#

Useful links,

https://github.com/efabless/openlane

https://github.com/nickson-jose/openlane_build_script

https://github.com/nickson-jose/vsdstdcelldesign

https://github.com/kunalg123
 
