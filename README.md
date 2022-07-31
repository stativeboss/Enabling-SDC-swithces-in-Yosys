# Enabling-SDC-swithces-in-Yosys
 
This repo is my step-by-step journey through the ten-week [Hardware Design Program offered by VLSI System Design](https://www.vlsisystemdesign.com/hdp/). The title of this repo is the ultimate outcome of this journey. 

## Step-1: A hawk-eye view write-up of the project
Inherently, Yosys is capable of performing the following: Process a synthesizable Verilog code, check for equivalence, and map to standard cell libraries. The framework as such does not consider timing, power, or area constraints. This work aims to enable the Synopsys Design Constraints (SDCs) in Yosys.

Though proprietary resources offer 'personalised' support, are more secure and powerful, they are 'black boxes' which are expected to work as plug-and-play. Open resources on the other hand, are more customizable, reliable, and cheaper. As the market grows, there will be an obvious progress in the proprietary resources. If the open source resources could catch up with the industry needs, it'd give the designers a flexibility to choose an open source resource or a proprietary resource, depending on the need and application. In the VLSI industry, many open source resources have been developed to cater the Application Specific Integrated Circuit(ASIC), as well as the Field Programmable Gate Array(FPGA) design flows. 'Yosys', a logic synthesis framework, is one such resource that has gained popularity. More about Yosys is discussed in the next section.

During the synthesis phase, the designer might want to set certain expectations (or rather requirements) on the design and timing characteristics. These expectations are called 'Constraints'. 'Synopsys Design Constraints' (SDC) is a widely used format by many proprietary tools to describe these constraints. It'd be useful to run SDC or mimick similar constraints on Yosys.

![image](https://user-images.githubusercontent.com/14873110/174593593-0a0af9b9-8309-4a1d-b57e-ccb37e6132c0.png)

In the above figure, a. Shows typical ASIC design flow while b. Shows the proposal

## Step-2: Tool installation and understanding the timing theory

### Installation of tools

For installation of tools, the [free course](https://www.udemy.com/course/vsd-a-complete-guide-to-install-open-source-eda-tools/) by Kunal Ghosh on Udemy was referred.

The course suggests the following steps:
1. Install Oracle VM.
2. Download any Linux distro (ISO file).
3. Install the downloaded distro on VM.
4. Clone the git https://github.com/kunalg123/vsdflow. This is a flow that downloads all required tools and files on the VM.

Unfortunately, the timing analysis tool [OpenTimer](https://github.com/OpenTimer/OpenTimer) wasn't installed by the flow and had to be manually installed.

The following issues were faced during different stages of the manual installation. Solutions to all these problems is indicated after quoting the problems:
1. cmake couldn't run successfully due to the 'C compiler identification unknown' error.

![image](https://user-images.githubusercontent.com/14873110/174555188-bfce9953-ce79-4595-905a-8809334cf5fa.png)

2. The gcc was installed and yet the 'command not found' error was prompted.

![image](https://user-images.githubusercontent.com/14873110/174554942-bf5da5d5-4476-4000-8148-34c1bd56065a.png)

3. The 'make' command prompted an error that the altStackMem from doctest.h file (present in the vsdflow/opentimer directory) had a non-constant declaration inside it (upon opening the file, it was found that the altStackMem array was declared as [4 * SIGSTKSZ]. Apparently, SIGSTKSZ is used to allocate an array dynamically).

The following are the solutions in-line:
1. For cmake and gcc issues, build-essential, cmake, make, and gcc packages were autoremoved and re-installed. 
2. For doctest.h issue, the 4 * SIGSTKSZ from the array was replaced by 32.

![image](https://user-images.githubusercontent.com/14873110/174568536-9ce1079c-3df9-4dd9-9a16-535759c909db.png)

### Testing the essential tools
1. OpenTimer: [ref](https://github.com/OpenTimer/OpenTimer)

![image](https://user-images.githubusercontent.com/14873110/174590328-e499ae67-4220-4615-81f4-00bb5385b488.png)

The following command lines are used to generate the timing report for the above circuit
```
ot> cd example/simple
ot> read_celllib osu018_stdcells.lib
ot> read_verilog simple.v   
ot> read_sdc simple.sdc
ot> report_timing      # report the most critical path
```
![image](https://user-images.githubusercontent.com/14873110/174589416-8d53ee1d-6059-498c-934d-530a8dcbb937.png)

2. Yosys: [ref]()

The following command lines are used to check whether Yosys is working properly:
```
>>git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git //cloning git that has .v examples
>>cd sky130RTLDesignAndSynthesisWorkshop/verilog_files
>>yosys
>>read_verilog good_mux.v
>>synth -top good_mux
>>show
```
![image](https://user-images.githubusercontent.com/14873110/175656292-24d9e495-3fdb-4d37-ae60-93368bc2b1db.png)

The good_mux.v contains the following code:

![image](https://user-images.githubusercontent.com/14873110/175656808-e1b41ef3-deae-4098-a9eb-c7b7a66e3b1a.png)

### Static Timing Analisys using OpenTimer

#### The basics of STA

*Timing path* : A timing path is the path between A and B. 'A' can be either a clk pin or an input port while 'B' can be either D-pin of FF or an output-port.

![image](https://user-images.githubusercontent.com/14873110/179394947-9a773f4a-a0ae-4e80-b713-bde97edbd80d.png)

In the above figure, paths 1->2, 1->2->3->4->5, 6->3->4, 7->5 are the four possible timing paths.

*Arrival time* : The time required for the signal to start from A and reach B. It may be noted that arrival time is calculated at B only. Also, B may have multiple arrival times depending on whether it has multiple A's 

(for example the above figure has two arrival times at 5 i.e, 1->2->3->4->5 and 7->5).

*Required time* : This is the time that defines the constraints. Consider you wish to catch a train at 5:45pm. You arrive at the station at 5:35pm. Here the required time is 5:45pm and the arrival time is 5:35pm.

Now the constraint could also have a lower limit. In the above train example, let us say the railway staff released a notice saying that the passengers will NOT be allowed into the railway station half an hour prior to their respective train time. In this case, you have to reach the station between 5:15pm and 5:45pm.

The difference between arrival time and required time is called _slack_. Since required time can have max and min limits, slack also can be min_slack or max_slack. 

In the train example, min_slack would be 20min (5:35 - 5:15), while the max_slack would be 10min (5:45 - 5:35). These slacks would give us the _set-up time_ (max_slack) and _hold time_ (min slack) constraints. It my be noted that negative slack (min or max) is not desirable.

The below figure will now be used to introduce the types of Set-up/Hold analysis:

![image](https://user-images.githubusercontent.com/14873110/181148213-672040aa-6865-417f-ad39-6fc90fcec565.png)


path 6->4 is reg2reg </br>
path 1->2 is in2reg </br>
path 5->out is reg2out </br>
path 1->out is in2out </br>
path 10->11 is clock gating </br>

There could be another path from clk_2 to the reset pin of the capture flop through some combinational circuit. This path is called _recover/removal_ path.</br>

paths 10->14 and 6->15 data-to-data </br>

In the above figure, FF_clk_gating and FF are connected to a latch. The idea is to 'borrow' or 'give' time to these flipflops (FF_clk_gating may _borrow_ while the latch may _give_ to FF) in case the timing is not met. This is possible because flipflops are edge triggered while latches are level triggered (so even if we 'borrow' some time from latch, the latch would still function fine!). This concept creates two more timing analysis:</br>

path 11->17 time borrow</br>
path 16->19 time given</br>

Another aspect to be considered while analysing the timing is the slew. If slew is too sharp, it'd increase the short circuit power and if it too blunt, it'd increase the opening time. Therefore there is a range within which the slew should fall. Slew can be analysed for data signals and clock signals.

Besides these analyses, there are also load analysis (min/max fanout and min/max capacitance) and clock analysis(skew and pulse width). 

Consider the below figure:

![image](https://user-images.githubusercontent.com/14873110/181353364-4f577fef-7c43-44ca-95bb-02b3f9af34e8.png)


The timing graph for the above circuit would be as below:

![image](https://user-images.githubusercontent.com/14873110/181354029-da0d0815-d571-4f80-82fc-8a0e5b9b9269.png)

Let's say the posedge of the clock reaches the _source_ node at 0. Arrival time at any pin 'Pi' is defined as the time taken for the signal to reach that pin after the 0 instant. Moving from source side towards the output (Y) side, the Actual Arrival Time (AAT) at any node would be the sum of all delays until that particular node. Similarly, the Required Arrival Time (RAT) at a prticular node would be obtained by subtracting all delays till that node when moving from output towards the source (RAT at the output would be given and we need to subtract the delays from this value as we move from Y to the source). One node may have more than one AAT or RAT (for example, AAT at P5 could be 1.1 or 1.3 time units) and we'd be choosing one of these values depending on our analysis approach (generally we try to design for worst case). This is where the concept of slack comes into picture. Slack is defined as RAT-AAT. It is evident (from the expression of slack) that the slack at every node should be positive to say that the timing is met.

Note: If the analysis is made by tracing the worst timing path of a timing graph, it is called Graph Based Analysis (GBA) and if the analysis is made based on the path that the signal would take on actual silicon, it's called Path Based Analysis (PBA). 




![image](https://user-images.githubusercontent.com/14873110/177349666-aa5e80d4-6059-4afc-b357-3885c07372cb.png)

The following is contained in my_netlist.v:

![image](https://user-images.githubusercontent.com/14873110/178023569-d8c2903a-e358-4fd0-a993-c62bb76e8378.png)


![image](https://user-images.githubusercontent.com/14873110/177344348-2fcb6a30-dcfe-4f5e-9c97-96db703e3efa.png)
![image](https://user-images.githubusercontent.com/14873110/177345217-a3711916-7920-4139-9c59-991218790dff.png)

### Reg2reg analysis:

The following figure shows the timing constraints that were imposed and reflected by OpenTimer in the above figure:
![image](https://user-images.githubusercontent.com/14873110/177346154-3063f7bf-b36c-4dc7-9b69-66896404e5b7.png)

![image](https://user-images.githubusercontent.com/14873110/181865171-3f66a742-f362-4ffb-9b79-1b0cc4015572.png)

The below figure briefs on what above constraints mean:

![image](https://user-images.githubusercontent.com/14873110/182025804-cdd27af6-8803-4469-bbd9-ef2a181a2be0.png)


my_netlist.v is modified as below for slack compuattion studies and ECO:

```
module my_module (
in,
clk,
out
);

// primary inputs
input in;
input clk;

// primary outputs
output out;

// wires
wire n1;
wire n2;
wire n3;
wire n4;
wire n5;
wire n6;
wire in;
wire clk;
wire out;
wire c1;
wire c2;
wire c3;
wire c4;
wire c4_1;
wire c4_2;
wire c4_3;
wire c4_4;

// cells
my_dff_xsize80 f1 (.d(in), .ck(c4), .q(n1));
my_inv_xsize1 u3 (.a(n1), .o(n2));
my_inv_xsize2 u4 (.a(n2), .o(n3));
my_nand2_xsize1 u6 ( .a(n1), .b(n3), .o(n4));
my_nand4_xsize1 u5 ( .a(n3), .b(n2), .o(n5));
my_nor2_xsize1 u7 ( .a(n4), .b(n5), .o(n6) );
my_dff_xsize80 f2 ( .d(n6), .ck(c2), .q(out) );

//clock path
my_inv_xsize1 u8 (.a(clk), .o(c1));
my_inv_xsize1 u9 (.a(c1), .o(c2));
my_inv_xsize1 u10 (.a(c2), .o(c3));
my_inv_xsize1 u11 (.a(c3), .o(c4_1));
my_inv_xsize1 u12 (.a(c4_1), .o(c4_2));
my_inv_xsize1 u13 (.a(c4_2), .o(c4_3));
my_inv_xsize1 u14 (.a(c4_3), .o(c4_4));
my_inv_xsize1 u15 (.a(c4_4), .o(c4));

endmodule

```

The circuit that we are trying to analyse with the above netlist would be as follows:

![image](https://user-images.githubusercontent.com/14873110/181865680-83ed760c-bf6a-4a39-80de-7bcee5674163.png)

The timing report is as follows:

![image](https://user-images.githubusercontent.com/14873110/181865835-e2462ce3-b974-4094-84f7-f92c386fc2ff.png)

### Interface analysis

*Case-1 : c2q and combinational delay is known*

With the following timing constraints:

![image](https://user-images.githubusercontent.com/14873110/182027201-90330905-95c3-4b6e-9de5-12ad9a8fbfd9.png)


The following the timing report:

![image](https://user-images.githubusercontent.com/14873110/182027172-a4793a06-ac4b-4707-830f-38b4a25cdcca.png)













