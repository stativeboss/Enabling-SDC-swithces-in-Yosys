# Enabling-SDC-swithces-in-Yosys
 
This repo is my step-by-step journey through the ten-week [Hardware Design Program offered by VLSI System Design](https://www.vlsisystemdesign.com/hdp/). The title of this repo is the ultimate outcome of this journey. 

## Step-1: A hawk-eye view write-up of the project
Inherently, Yosys is capable of performing the following: Process a synthesizable Verilog code, check for equivalence, and map to standard cell libraries. The framework as such does not consider timing, power, or area constraints. This work aims to enable the Synopsys Design Constraints (SDCs) in Yosys.

Though proprietary resources offer 'personalised' support, are more secure and powerful, they are 'black boxes' which are expected to work as plug-and-play. Open resources on the other hand, are more customizable, reliable, and cheaper. As the market grows, there will be an obvious progress in the proprietary resources. If the open source resources could catch up with the industry needs, it'd give the designers a flexibility to choose an open source resource or a proprietary resource, depending on the need and application. In the VLSI industry, many open source resources have been developed to cater the Application Specific Integrated Circuit(ASIC), as well as the Field Programmable Gate Array(FPGA) design flows. 'Yosys', a logic synthesis framework, is one such resource that has gained popularity. More about Yosys is discussed in the next section.

During the synthesis phase, the designer might want to set certain expectations (or rather requirements) on the design and timing characteristics. These expectations are called 'Constraints'. 'Synopsys Design Constraints' (SDC) is a widely used format by many proprietary tools to describe these constraints. It'd be useful to run SDC or mimick similar constraints on Yosys.

![image](https://user-images.githubusercontent.com/14873110/174593593-0a0af9b9-8309-4a1d-b57e-ccb37e6132c0.png)

In the above figure, a. Shows typical ASIC design flow while b. Shows the proposal

## Step-2: Installing the tools

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

## Step 2.a: Static Timing Analisys course from Udemy






