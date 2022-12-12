# CMPE283-Assignment-2 & 3 <br>

Student Name : Bhavya Hegde <br>
email : bhavya.hegde@sjsu.edu


## Modify the CPUID emulation code in KVM to report back additional information when special CPUID leaf nodes are requested. 

## Assignment 2

 ### Implementation of CPUID leaf nodes 0x4FFFFFFC and 0x4FFFFFFD <hr style="border:2px solid gray">

 
 ### Question 2: 
 
 #### Steps used to complete this assignment : 


Following are the development steps I have followed to develop the solution for assignment-2:  Adding leaf nodes 0x4ffffffc and 0x4ffffffd to the CPUID emulation code in KVM.

* Fork the Linux Kernel  https://github.com/torvalds/linux.git  

* Checkout the linux kernel code from my github repository 
                       git clone https://github.com/hegdebhavya/linux.git 

 The new kernel code that is checked-out here is **6.1.0-rc6+**

![1_Linux_repo_Checkout](https://user-images.githubusercontent.com/85700971/205465167-2a3b5eb1-6a2f-4c2b-b249-a8bfbdc8ad08.png)


 The current Linux Kernel version is as below,

![2_Current_Liux_Version](https://user-images.githubusercontent.com/85700971/205465173-c2a1e955-7a6d-4165-b528-a1751448835f.png)




* We now install all the libraries and tools we would need to build the new Linux Kernel.

```
sudo apt install gcc make bison flex pkg-config 
sudo apt-get install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools g++ libelf-dev

```

* Next we make the config file for our kernel build 

```
make xconfig
```

* I have selected the default options for xconfig. Once this command is executed we start the make process
 ``` 
 make
 ```

* Once the make is complete, 
```
sudo make modules_install
sudo make install
```
* Next, we configure the grub loader, to check the exact kernel version name you can navigate to the /lib/modules directory and get the version, with this you can run commands below, 


```

sudo update-initramfs -c -k 6.1.0-rc6+
sudo update-grub

```

![3_GRUB_config](https://user-images.githubusercontent.com/85700971/205465182-5cfcefa6-3828-4c55-884c-dbe554d54d67.png)




* Next, we reboot the system and run **uname -r** to check the new linux kernel version

![8_Linux_Kernel_New_version](https://user-images.githubusercontent.com/85700971/205470726-bf50c73c-5bbe-4e6c-872c-ad25e5df5504.png)



We observe that the Linux version has been upgraded to **6.1.0-rc6+** from the previous **5.15.0-53-generic**.

  *  Next, I have modified the files /linux/arch/x86/kvm/cpuid.c and /linux/arch/x86/kvm/vmx/vmx.c to add the required variables and logic to add support for cpuid leaf nodes 0x4ffffffc and 0x4ffffffd. These changes are committed to the current GitHub repository and commits can be seen [here](https://github.com/hegdebhavya/linux/commit/b0f1c18a54b1e92549034ab453affeccfb367bf0)

* After the files are modified we build the modules again by running following commands

```
make modules
```
![10_makemodule](https://user-images.githubusercontent.com/85700971/205470767-bc6913fa-17de-4c68-8b07-55f699e6bfe9.png)

* Once make completes please install the modules using command below,
```
make INSTALL_MOD_STRIP=1 modules_install
```

* We then reload the kvm modules using following commands
```
sudo rmmod kvm_intel
sudo rmmod kvm
sudo modprobe kvm_intel
sudo modprobe kvm
```

![9_modulereload](https://user-images.githubusercontent.com/85700971/205470806-5e1e4330-1c56-4991-9b67-5644c654af18.png)


* To test the cpuid modifications we will now install virt-manager and run a 32-bit Ubuntu Virtual machine
```
sudo apt install virt-manager
```

* Once virt-manager is installed we can launch the virtual machine manager by running command **virt-manager** ;  create a new 32-bit VM in my case I have tested with Ubuntu 

![4_innervm_specs](https://user-images.githubusercontent.com/85700971/205465206-057c6326-3cc9-4653-9743-cfff4e069701.png)

 We can see the 32-bit inner VM in the screenshot below,

![5_Screenshot_innervm](https://user-images.githubusercontent.com/85700971/205465239-e1420982-da06-4241-b63e-1d833caef6d1.png)



* Next, we can download a test program to test the cpuid leaf nodes by downloading cpuid from,
http://archive.ubuntu.com/ubuntu/pool/universe/c/cpuid/cpuid_20170122.orig.tar.gz into this new 32-bit Ubuntu VM 

* Once the archive is extracted we can navigate to the extracted directory and run the make command to build the program.

 ## Output  

* To check the Total number of Exits we can run the cpuid command with leadnode 0x4FFFFFFC with command below,

```

./cpuid -l 0x4ffffffc    

```

Here, the leaf node is specified with “-l” argument.

![6_Output_leaf_0x4ffffffc](https://user-images.githubusercontent.com/85700971/205465296-bd3aab18-b309-4be3-997c-1cb7cf7cc1ff.png)


We see the total number of exit counts increasing and the values are returned in the eax register. The leafnode and the increasing exit counts are highlighted in the screenshot above.

* To check the Total time taken to process these exits we can run the cpuid command with leafnode 0x4FFFFFFD with command below,

```

./cpuid -l 0x4ffffffd    

```

![7_Output_leaf_0x4ffffffd](https://user-images.githubusercontent.com/85700971/205465303-c84fbda4-26fe-44c5-9868-8975bfcefd26.png)



We see the total processing time for exits increasing and the values are returned in registers ebx and ecx. The leafnode and the increasing values returned are highlighted in the screenshot above. The high 32-bits of total time spent are returned in register ebx and the low 32-bit of the total time spent are returned in the register ecx.

## Assignment 3 

### Implementation of CPUID leaf nodes 0x4FFFFFFE and 0x4FFFFFFF <hr style="border:2px solid gray">

* To add support for 0x4ffffffe and 0x4fffffff I have modified the cpuid.c and vmx.c. The SDM defines Exit reasons on Volume 3 Appendix C VMX Basic Exit Reasons section. Here we find all the exit reasons defined by Intel SDM. The Exit reasons supported by VMX are mentioned in file vmx.h at linux/arch/kvm.vmx/vmx.h
First we list the exit reasons not supported by SDM and create a separate list for exit reasons not supported by KVM. The functions to validate the exit reason in modified cpuid code is shown below,

![01_exit_reason_check_function](https://user-images.githubusercontent.com/85700971/206966923-a21961a8-eeba-4fd2-81c1-bfabd5a1fca8.png)

* Once we verify that the subleaf provided to cpuid program is a valid exit reason, we proceed to increase the count and the timer for the particular exit reason in the defined arrays. 

* To add support for CPUID  leaf nodes  0x4ffffffe and 0x4fffffff please checkout the repo 
```
 git clone https://github.com/hegdebhavya/linux.git 
 ```

* Changes to add support for the new leafnodes can be seen [here](https://github.com/hegdebhavya/linux/commit/e44d5bcb9bcf586438dc1fea16b4dede2b7fe844) , 
 Next we build the modules by running the commands

```
sudo make modules
```
* We then install the modules

```
sudo make INSTALL_MOD_STRIP=1 modules_install

```

* Please reload the modules by following the below commands

```
rmmod kvm_intel
rmmod kvm
modprobe kvm_intel
modprobe kvm

```

* Next, we launch the 32-bit Ubuntu VM using virt-manager
```
virt-manager
```

* The output for testing the number of exits for exit reason 10 (0x0a) which is EXIT_REASON_CPUID can be checked by running command below,
```
./cpuid -s 0x4ffffffe -s 10 
```
* The output can be seen in the eax register in screenshot below,
![02_output_0x4ffffffe](https://user-images.githubusercontent.com/85700971/206967270-dabe81bd-ff89-4003-a599-8f8b8cb82236.png)

* We observe that the total count for exit 10 is increasing everytime we run the cpuid command.
 The dmesg for this exit can be seen in screenshot below

![03_dmesg_0x4ffffffe](https://user-images.githubusercontent.com/85700971/206967332-9da28612-0635-41f8-a7f8-606d588f8951.png)

* The output for testing the total time needed for processing exit reason 10 (0x0a) which is EXIT_REASON_CPUID can be checked by running command below,

```
./cpuid -s 0x4fffffff -s 10
```

* For output, the high 32-bits of total time spent are returned in register ebx and the low 32-bit of the total time spent are returned in the register ecx.

The output in the ebx, ecx registers can be seen in the screenshot below,

![04_output_0x4fffffff](https://user-images.githubusercontent.com/85700971/206967438-c4d847e4-6d18-4238-884f-7b15cca14dd6.png)

We observe that the total time for exit 10 is increasing everytime we run the cpuid command; this is seen in registers ebx and ecx.

* Next, we test the output of cpuid when we give exit reason which is not supported by SDM, we can test this by using Exit reason 77

![05_unsupported_SDM](https://user-images.githubusercontent.com/85700971/206967476-632dba39-f14c-448f-bd52-9cfbde198e1c.png)

* Next, we test the output of cpuid when we give exit reason which is not supported by KVM but is supported by SDM, we can test this by using Exit reason 5

![06_unsupported_KVM](https://user-images.githubusercontent.com/85700971/206967530-32960422-207d-489b-937f-7f1dfb3d2f48.png)

* The output contains eax, ebx, ecx and edx as zero.

### Questions


#### 3. Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there more exits performed during certain VM operations? Approximately how many exits does a full VM boot entail?

#### Ans.
Yes I see IO_INSTRUCTION (exit reason 30) exit increase at a stable rate. I also observed CPUID (exit reason 10) exit increasing at stable rate while running the cpuid instructions. 

To calculate the number of exits for full VM boot we can check this by running reboot command and then checking cpuid for leafnode 0x4ffffffc, this can be seen below,

![08_boot_exits](https://user-images.githubusercontent.com/85700971/206967645-829abae7-a4ec-4dbe-b87e-e0ead7256a8e.png)

We observe that the reboot  caused 3,334,075 (0x0032dfbb) total exits.


#### 4. Of the exit types defined in the SDM, which are the most frequent? Least?

####  Ans
I have collected the output of cpuid command for exit reasons from 0 - 75 and found the following values in the counter

| Exit Number | Exit Reason | Exit Frequency |
| :---         |     :---:      |          ---: |
| 48   | EXIT_REASON_EPT_VIOLATION    | 502051    |
| 30    | EXIT_REASON_IO_INSTRUCTION      | 195510      |
| 10  | EXIT_REASON_CPUID     | 72893    |
| 28    | EXIT_REASON_CR_ACCESS       | 27148      |
| 0  | EXIT_REASON_EXCEPTION_NMI    | 13994    |
| 40    | EXIT_REASON_PAUSE_INSTRUCTION      | 1987      |
| 31   | EXIT_REASON_MSR_READ                | 162    |
|54    | EXIT_REASON_WBINVD       | 6      |
|55   |  EXIT_REASON_XSETBV     | 2    |
| 29   | EXIT_REASON_DR_ACCESS      | 1      |



I found Exit reason **48 (EPT_VIOLATION) and Exit reason 30 (IO_INSTRUCTION) to be most frequent** . 
The Exit reason **29 (DR_ACCESS) is the least frequent**.

![07_checking_frequency](https://user-images.githubusercontent.com/85700971/206967722-0f378bc5-039a-4a2e-ba4b-0c292c3709ee.png)



