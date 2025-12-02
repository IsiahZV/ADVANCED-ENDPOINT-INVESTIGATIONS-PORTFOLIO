# MBR and GPT Analysis

**Date:** 2025-12-01

**Objective:** Explore the boot process, understand MBR and GPT structure, and learn the attacks on MBR and GPT

**Environment:** TryHackMe – “MBR and GPT Analysis” room  
**Tools Used:** HxD
##


## Question 1: Which component of the MBR contains the details of all the partitions present on the disk?

The second component of MBR is the **partition table** which can be compared to rooms in a house (i.e., one partition may contain files that are relative to the OS files necessary for the booting process (bootable partition) or one may contain files relative to personal use). This table contains details of all (4) partitions present on the disk.



## Question 2: What is the standard sector size of a disk in bytes?

Disks are divided into sectors with a standard size of **512 bytes**

**Sidenote:** The MBR/GPT partitioning scheme are located in the first sector of a disk.



## Question 3: Which component of the MBR is responsible for finding the bootable partition?

The bootloader code (practically all of the sectors **besides** the the partion table going forward - 446 bytes) contains the initial bootloader which is the first thing that executes in the MBR scheme. Its purpose is to find the bootable partition from the table present on the scheme.



## Question 4: What is the magic number inside the MBR?

"**55 AA**", signifying the end of the MBR scheme.



## Question 5: What is the maximum number of partitions MBR can support?

4



## Question 6: What is the size of the second partition in the MBR file found in C:\Analysis\MBR\? (rounded to the nearest GB)

Highlighted in the image below is the second partition out of the four.

<img width="1440" height="568" alt="Screenshot 2025-12-01 at 8 39 09 PM" src="https://github.com/user-attachments/assets/752758b6-e0ec-4a73-8248-e54a5863d620" />


Now, to calculate the size, I need to see what the last 4 bytes of this partition converts to in decimal format (in reference to Int32)
