# Enabling-SDC-swithces-in-Yosys
 
This repo is a step-by-step journey of my [Hardware Design Program offered by VLSI System Design](https://www.vlsisystemdesign.com/hdp/). The title of this repo is the ultimate outcome of this journey. 

## Step-1: A hawk-eye view write-up of the project
The following PDF introduces the problem statement, and briefs on the goal that this work aims at.

[Enabling_SDC_Switches_in_Yosys.pdf](https://github.com/stativeboss/Enabling-SDC-swithces-in-Yosys/files/8939670/Enabling_SDC_Switches_in_Yosys.pdf)

## Step-2: Installing the tools and shortlisting the SDC commands

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

