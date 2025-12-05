# MBR and GPT Analysis

**Date:** 2025-12-01

**Objective:** Explore the boot process, understand MBR and GPT structure, and learn the attacks on MBR and GPT

**Environment:** TryHackMe – “MBR and GPT Analysis” room  
**Tools Used:** HxD, FTK Imager
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

<img width="1440" height="568" alt="Screenshot 2025-12-01 at 8 44 15 PM" src="https://github.com/user-attachments/assets/71b3d729-3973-4636-af57-ffe36334c72a" />

32,192,512 x 512 = 16,482,566,144

Now, I have to convert this number in bytes to gigabytes. This can be done using an online converter or calculator.

The formula is:
One GB = 1,073,741,824 bytes

Gigabytes = # of bytes / 1GB in bytes

The number I get is: 15.35, with rounding up, itd be 15.4 and even then, it'd be less than .5, however, the answer will take 16 instead of 15. 

- 16GB



# Scenario: MBR Corruption

An organization's critical database server suddenly became unbootable, causing widespread panic. The initial investigation found an employee opened a malicious email attachment, prompting a reboot. The employee rebooted the system, after which the system became completely unbootable. Most of the clues point to the malware's deliberate corruption of the server's Master Boot Record (MBR).

A quick look at the server's MBR points to two corruptions. One of them is the logical address of the first partition, which was previously 00 08 00 00, and the other is a critical component of the MBR that must be the same across all the MBRs. Fixing these corrupted bytes will make the system bootable again, which can be verified by re-opening the fixed disk image using FTK Imager.


## Instructions

- Utilze two tools: FTK Imager and HxD

## Context

The infected disk image was preserved immediately after the attack by booting the system with a live USB. Placed at C:\Analysis\MBR_Corrupted_Disk.001


## Question 1: How many partitions are on the disk?

When in relation to the MBR partition scheme, I know that the partitions table have the byte range of 446 - 509 and start with the very first two bytes being right after where the bootloader code ends. Another variable that helps is the row "1B0", focusing on the very last two bytes in that row. 

<img width="1440" height="612" alt="Screenshot 2025-12-02 at 7 15 44 PM" src="https://github.com/user-attachments/assets/3309e3df-abc4-4e3a-87a3-7a430b0c2cb2" />

- 1 partition



## Question 2: What is the first byte at the starting LBA of the partition? (represented by two hexadecimal digits)

Highlighted in the image below is the start of the LBA address, the first byte of it is "FF", however this needs to be fixed.

<img width="1440" height="612" alt="Screenshot 2025-12-02 at 7 22 23 PM" src="https://github.com/user-attachments/assets/cac244db-8da6-4b0f-b38c-f6f3df87d1a9" />

Upon editing the bytes (00 08 00 00), the decimal value (Int32) is 2048. 

2048 multiplied by the size of the MBR sector (512) = 1,048,576 

Now is the time to search this value in HxD

**Note:** Ensure search is set to "Dec"

<img width="701" height="193" alt="Screenshot 2025-12-02 at 7 48 31 PM" src="https://github.com/user-attachments/assets/a5b5ebee-0f03-4129-b87e-8f5e8bcea8b7" />

- EB


<img width="567" height="193" alt="Screenshot 2025-12-02 at 8 15 49 PM" src="https://github.com/user-attachments/assets/03b8e8f5-e075-4748-8a02-64b1ea3f969a" />


**Sidenote:** The questions don't have you identify the second point of corruption which is the **MBR signature** (55 AA) as this has to be applied for the last two bytes. Additionally, as rules requested, I saved the file to overwrite the corrupted MBR scheme and will now utilize FTK Imager for further analysis. 

As a result, FTK Imager shows the working disk.

<img width="646" height="231" alt="Screenshot 2025-12-02 at 8 18 18 PM" src="https://github.com/user-attachments/assets/0b8e8b86-e87d-49ff-9482-2e20a8e342ee" />



## Question 3: What is the type of the partition?

<img width="646" height="231" alt="Screenshot 2025-12-02 at 8 21 00 PM" src="https://github.com/user-attachments/assets/4a450b62-5598-4ede-a4d3-501ae85bbb31" />

- NTFS

This indicates the filesystem of a partition



## Question 4: What is the size of the partition? (rounded to the nearest GB)

<img width="1440" height="549" alt="Screenshot 2025-12-02 at 8 53 57 PM" src="https://github.com/user-attachments/assets/41a909a6-36f9-49d6-bc50-1b5f966d4b03" />

To do this, I concern myself only with the last 4 bytes of the one partition available.
- 00 F0 BF 03 = 62910464
  - 62910464 x 512 = 32,210,157,568
  - Either a tool can be used for conversion or manual math, either way, the result is 32GB once rounded



## Question 5: What is the flag hidden in the Administrator's Documents folder?

Back to using FTK Imager, I'll go through some of the files and look for the Administrator's Documents folder.

<img width="933" height="549" alt="Screenshot 2025-12-02 at 9 20 23 PM" src="https://github.com/user-attachments/assets/883df558-3b77-47ab-91b0-16126865ef29" />

<img width="933" height="549" alt="Screenshot 2025-12-02 at 9 21 49 PM" src="https://github.com/user-attachments/assets/8d8d5884-55b1-495c-84b5-7600810e8c99" />

- THM{Cure_The_MBR}



# GPT Analysis

## Question 1: What is the partition type GUID of the 2nd partition given in the attached GPT file?


<img width="1440" height="818" alt="Screenshot 2025-12-05 at 2 35 16 PM" src="https://github.com/user-attachments/assets/b3a1d02d-f038-4e63-aa3f-438eea0add7d" />


Highlighted in its entirety is the second partition in the GPT file within the Partition Entry Array. The first row highlighted in blue is the partition type GUID. The first 3 groupings of these bytes go through reversing due to being in little-endian format.

- 16 E3 C9 E3 5C 0B B8 4D 81 7D F9 2D F0 02 15 AE
  - 16 E3 C9 E3 -> E3 C9 E3 16
  - 5C 0B -> 0B 5C
  - B8 4D -> 4D B8
  - 81 7D
  - F9 2D F0 02 15 AE

This will result in: E3C9E316-0B5C-4DB8-817D-F92DF00215AE

As an extra, research will show this to be a Microsoft Reserved Partition where the Windows system uses this partition for compatibility purposes.


