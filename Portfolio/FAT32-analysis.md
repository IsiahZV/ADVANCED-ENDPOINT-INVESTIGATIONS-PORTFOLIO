he# FAT32 Analysis

**Date:** 2025-12-11

**Objective:** 
-

**Environment:** TryHackMe – “FAT32 Analysis” room  
**Tools Used:** HxD,

# FAT32 Structure : Reserved and FAT Areas

## Question 1: We have a hypothetical file B and its cluster chain starts at cluster F and ends at cluster 10 . What would be the value of the FAT entry at cluster F? Provide the value as you would read it in the HxD editor (e.g. 00001111). Note: File B is not a file on the image.**

<img width="291" height="204" alt="Screenshot 2025-12-13 at 12 35 00 PM" src="https://github.com/user-attachments/assets/dda9b6d8-67bf-4198-a2a6-3bdc76c0d615" />

Before starting, I need to clarify for the sake of myself due to the complexities of FAT32 analysis and for this, I will zoom out to understand the bigger picture. I do acknowledge that this question assumes that I already know where file B starts and that I'm already working within FAT #1.

- Theoretically, **firstly**, I'd begin by identifying where the partitions start when working with MBR/GPT
- Upon finding the partition location, I'd have to shift my mindset from disk to strictly to the partition I'm working in and acknowledge that there's a filesystem structure (FAT32, NTFS, ext4, ...)
- Gather information within the boot sector which is the first sector of the FAT32 filesystem
  - Information here pertains to sector size, reserved sector count, number of FATs, FAT size, root directory location, and cluster size.
  - This is how I map out the entire partition I'm working in as the boot sector values help me calculate where **FAT #1 & 2 starts**, where the **root directory** starts, and where the **data region (clusters)** starts

**QUESTION APPLICATION**

File B is found in the root directory because it is the only place where files are listed, containing the file name, size, timestamps, and starting cluster number. The root directory should always be consulted first while FAT will tell you where the file continues and the data region will contain the actual information.

<img width="301" height="304" alt="Screenshot 2025-12-13 at 12 39 52 PM" src="https://github.com/user-attachments/assets/591e25a9-5c66-4955-949c-11741bd3d5f2" />


Hex uses base-16 - reference:
0 1 2 3 4 5 6 7 8 9 A B C D E F (15)

Mental example: If chaining from 5 to 6 would be 06 00 00 00 and 10 comes after F, it would be **10 00 00 00**.
- My issue was that i didn't make much of a mental note of how they are displayed for continuation (i.e., 0D 00.., 0E 00.., 0F 00..,)

So in all, the **value of of the FAT entry at cluster F** (to chain to 10) would look like:
- Cluster [F] : 10 00 00 00



## Question 2: Using the FAT32_structure.001 image, answer the following question: At which offset does the  FAT2 table start ( give in the offset value without spaces)? Remember, FAT1 starts right after the Reserved Sectors and FAT2 starts right after FAT1.

**SIDENOTE:**
- To find FAT1, you calculate the value of reserved sectors and multiply it by sector size (512) and convert to hex.
- To find FAT2, you add the value of sectors per FAT to the reserved sectors field, multiply it by sector size (512) and convert to hex
- For reference, I'm using 16-bit size in HxD


Highlighted in the image below is the **Sectors Per FAT** hex value. 
<img width="1440" height="609" alt="Screenshot 2025-12-15 at 6 34 21 PM" src="https://github.com/user-attachments/assets/ba24bd48-0d86-4055-987b-ec83fb0eb94a" />
The hex value translated is 961

Now I add this value to the **Reserved Sectors** value. This can be found in offset 0000 0000 where the last two bytes reside (7E 18 -> 18 7E when little endian)
- The reserved sectors value is 6270
  - Sectors Per Fat (961) + Reserved Sectors (6270) * 512 = 3 702 272
    - Now to convert to hex (because of locating by offset) -> 387E00

Finally, I enter this value into the search in HxD.

<img width="1440" height="489" alt="Screenshot 2025-12-15 at 6 53 22 PM" src="https://github.com/user-attachments/assets/1bb68bbd-1058-4022-b2a6-e4c0056cd303" />
<img width="1440" height="175" alt="Screenshot 2025-12-15 at 6 53 56 PM" src="https://github.com/user-attachments/assets/d4f60688-1289-4d3f-b49a-52802ed7052b" />

This is where the FAT2 table starts: 00387E00


# Data Area

## What is the filename of the file that starts at cluster 9?




