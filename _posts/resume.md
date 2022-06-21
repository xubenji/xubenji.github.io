## Education Background   
### Stevens Institute of Technology               	 GPA 3.9       
       09/2021-Expected 01/2023 M.S. in Computer Science
       Core Courses: Data structure; Computer organization and Programming; 
                     Advanced Programming in Linux Environment; 
                     Foundation of Cybersecurity; 
                     Real-time and Embedded Systems; etc.
                     
### Skills 
      Software: gdb, qemu, git, docker, UEFI.
      Programming Languages: C/C++, makefile, Shell, Python, Intel & ARM assembly.
      Operating Systems & Kernel: Linux, kdump, Operating System DolphinOS, and dolphin2. 

## Work Experience   
### Linux System Administration| KylinSoft |  Changsha, China               05/2019-11/2019 
    ● Performing Kylin (a Linux distribution) server updates and repairs. 
      I mostly work at boot problems in UEFI, BIOS, and U-boot, 
      including debugging OS bootstrap(boot.efi) in the UEFI framework with edk2.
    ● Maintained the server to make sure it was stable and scalability. 
      When the kernel crashed, I had to make sure the kdump kernel would correctly run in the servers to generate the log.
    ● Maintaining the libraries and services in Linux distributions. 
    ● Install, deploy, and debug private cloud server with docker.
    ● Provide a customized version of the Linux distribution(KylinSoft in Desktop) with LFS.
    ● Code version control, code repositories management with GitLab, and GitHub.
    ● Configure the development environment for Linux drive engineers.
      
## Project 
### DolphinOS and dolphin2
    ● MatchOS (support x86) has basic paging, thread scheduling, keyboard drive, hard-disk drive, file system, 
      and a simple graphical interface that could run in VMware and qemu.
    ● DolphinOS has the 4kb paging, and the kernel can add a new thread by itself. 
      The MatchOS has a fixed thread amount.
    ● dolphin2 is the last version of my OS project. I wrote it with a brand new makefile. 
      It supports x64 and has the basic paging, 
      and I switch from 4kb paging to 2MB paging because it is in the long mode. 
      Dolphin2 also supports the aarch64. 
    ● kernel is very similar to the Linux kernel in a lot of features.
Project Examples:  https://github.com/xubenji/dolphin2 

