
CMPE-283 Assignments - Fall 2022
==================================================


*	University: [San Jose State University](http://www.sjsu.edu/)
*   Course: [Virtual Technologies](https://catalog.sjsu.edu/preview_course_nopop.php?catoid=12&coid=58409)
*	Professor: [Michael Larkin](mailto:michael.larkin@sjsu.edu)
## Team Members

*   [Priya Gupta](https://github.com/priyagupta)
*   [Phani Sai Ram Munipalli](https://www.linkedin.com/in/iamphanisairam/)

### Distribution
**Priya Gupta**: 
*   Setup and Create Google Cloud Account, Project and VM Instance.
*   For CPUID leaf node %eax=0x4FFFFFFD:
    *   Return the high 32 bits of the total time spent processing all exits in %ebx
    *   Return the low 32 bits of the total time spent processing all exits in %ecx    
*   Linux Kernel Installation and Build 
*   Write a Test File.
*   Word Documentation

**Phani Sai Ram Munipalli**: 
*   Create SSH Key and Link to the VM Instance.
*   For CPUID leaf node %eax=0x4FFFFFFC:
    *   Return the total number of exits (all types) in %eax
*   Linux Kernel Installation Verification.
*   Run the test file and Verify the output.
*   Readme.md Documentation

## Implementation
 *   We have used two atomic variables , atomic_t for exit counter and atomic64_t  for counter total CPU cycles to process all exits and we are exporting these variable through EXPORT_SYMBOL so that we can access those variable inside vmx.c file while handing exits.
 
 *  We have created a function to records the total CPU processing time to `cpu_cycles` in vmx.c

## Required Software
*   Ubuntu Machine or Google Cloud Platform
*   Basic Shell scripting
*   Understanding of Linux Kernel

## Files
*   [Assignment 1 - Discovering VMX Features](https://github.com/Priygupt13/CMPE283_VirtualizationAssignments)
*   [Assignment 2 & 3 - Instrumentation in Hypercall](https://github.com/phanisaimunipalli/cmpe283-vt/blob/main/283_Assignment2-3_Fall2022.pdf)
*   [Word Documentation of Assignment 2](/vtassignment2_fall2022.pdf)





## Setup and Run on GCP


1. Create a New Project on GCP when you signup using your student account, you will get $300 free credits.

2. Setup GCP VM with Nested Virtualization enabled. (Either from UI or GCP Cloud Shell)

```bash
  gcloud compute instances create vt-instance --project=YOUR_PROJECT_ID --zone=us-west2-a --machine-type=n2-standard-8 --network-interface=network-tier=PREMIUM --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=240577316264-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/ubuntu-os-cloud/global/images/ubuntu-2204-jammy-v20221018,mode=rw,size=150,type=projects/cmpe-283-370620/zones/us-west2-a/diskTypes/pd-ssd --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any --enable-nested-virtualization --min-cpu-platform="Intel Cascade Lake"
```

3. Try to use machine type as ``n2-standard-8`` to get 8vcpu cores and 32GB RAM - this will help expedite the Kernel Compile and Build.
4. Create SSH Keys and Link it to your current project and VM Instance.
5. Now launch your cloud shell, and verify the nested virtualization enabled by running below commands.

```bash
  * cat /proc/cpuinfo (confirm vmx flags)
            or
  * grep -cw vmx /proc/cpuinfo (confirm output > 0)
```

6. Install the required compilation tools by running the below commands

```bash
        sudo apt-get update
        sudo apt-get install gcc
        sudo apt-get install make
        sudo apt-get install flex
        sudo apt-get install bison
        sudo apt-get install libssl-dev
        sudo apt-get install libelf-dev
```

7. Clone and setup linux repository in your VM instance.

```bash
  1. Fork Linux repo from Torvalds into your GitHub.
  2. Then do: 
     git clone https://github.com/Priygupt13/linux
  3. After cloning your linux repo from GitHub into VM Instance, go inside and ensure all files are okay.
```

## Kernel Build

``1. Copy the config file in the linux repo.``

``2. cp /boot/config-5.15.0-1025-gcp ~/linux/.config.``
  
``3. Install build dependencies``

``4. sudo apt-get install build-essential bionic ccache flex bison libssl-dev libelf-dev``

Format: 
``make -j TOTAL_CPU_CORES modules ``
 
```bash

* Prepare the linux 
    make prepare

* Build the modules
    make -j 4 modules

* Build the kernel
    make -j 4
    
* Packaging the modules
    sudo make -j 4 INSTALL_MOD_STRIP=1 modules_install
```

##  How to Install Kernel?
```bash

* Install the kernel into VM
     sudo make -j 4 install
     
* Reboot to see the changes
   sudo reboot
```

##  How to Verify Kernel Installation?
```bash

 check your old kernel version with : uname -a
  
 make -j 4
 sudo make -j 4 INSTALL_MOD_STRIP=1 modules_install
 sudo make -j 4 install
 sudo reboot 
 
 check your new kernel version with : uname -a
```
##  Where is the Code Change/Addition?

*   [cpuid.c - /arch/x86/kvm/](https://github.com/Priygupt13/linux/blob/master/arch/x86/kvm/cpuid.c)
*   [vmx.c - /arch/x86/kvm/vmx/](https://github.com/Priygupt13/linux/blob/master/arch/x86/kvm/vmx/vmx.c)


we have modified vmx.c and cpuid.c file and after modification we need to rebuild the kernal using make command.
  ```bash
        make -j 4 modules
        sudo make -j 4 INSTALL_MOD_STRIP=1 modules_install && make install
        sudo lsmod | grep kvm
        sudo rmmod kvm_intel
        sudo rmmod kvm
        sudo modprobe kvm
        sudo modprobe kvm_intel
        sudo lsmod | grep kvm
        sudo reboot 
   ```    
## Create a inner VM inside our current VM 
   
   * We need to install some extra tools for KVM, then check kvm-ok, and reboot:
     
     ```bash
          sudo apt-get install cpu-checker
          sudo apt update
          sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
          sudo kvm-ok
          sudo reboot
     ```
   Install Virtual Manager to create inner vm in current VM
   
      * sudo apt-get install virt-manager

##  How to Authorize User to Provide Access to NestedVM?
*   uname -a
*   sudo adduser er_priyagupta123 libvirt
    *   sudo getent group | grep libvirt
*   sudo adduser er_priyagupta123 kvm
    *   sudo getent group | grep kvm
*   sudo virsh list --all
    *   Expect an empty list
*   sudo systemctl status libvirtd
    *   sudo systemctl enable --now libvirtd

##  How to run a test file?

1. Use remote desktop to connect with the inner VM:
    https://remotedesktop.google.com/headless
2. Downlaod ubuntu image 
   *  wget https://releases.ubuntu.com/jammy/ubuntu-22.04.1-desktop-amd64.iso
   
3. Open virt Manager and install this ubuntu image on your inner VM using the GUI
   * sudo virt-manager
   
4. Now we created a test file in nested VM to test our functionalities. In our test file, we mov $0x4FFFFFFC to eax and we are executing CPUID in to get back results of the new leaf function.

5. Compile the test.c file and verify the output
   
![Compile](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image1.png)


![Login](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image2.png)




![NestedVM](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image3.png)



![Config](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image4.png)

![Install](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image5.png)



![GitLab](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image6.png)


![Complete](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image7.png)



![Build](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image8.png)



![TestOutput](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image9.png)

![TestOutput](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image10.png)

![TestOutput](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image11.png)

![TestOutput](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image12.png)

![TestOutput](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image13.png)

![TestOutput](https://raw.githubusercontent.com/phanisaimunipalli/cmpe283-vt/main/screenshots/image14.png)
