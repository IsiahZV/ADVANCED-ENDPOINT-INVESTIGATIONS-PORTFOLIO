# FAT32 Analysis

**Date:** 2025-12-11

**Objective:** 
-

**Environment:** TryHackMe – “FAT32 Analysis” room  
**Tools Used:** HxD,

## FAT32 Structure : Reserved and FAT Areas

**Question 1: We have a hypothetical file B and its cluster chain starts at cluster F and ends at cluster 10 . What would be the value of the FAT entry at cluster F? Provide the value as you would read it in the HxD editor (e.g. 00001111). Note: File B is not a file on the image.**

Disk
└── Partition
    └── FAT32
        ├── FAT
        │   ├── FAT[F]  → next cluster
        │   └── FAT[10] → EOF
        └── Data
            ├── Cluster F
            ├── Cluster ?
            └── Cluster 10

Before starting, I need to clarify for the sake of myself due to the complexities of FAT32 analysis and for this, I will zoom out to understand the bigger picture. I do acknowledge that this question assumes that I already know where file B starts and that I'm already working within FAT #1.

- Theoretically, **firstly**, I'd begin by identifying where the partitions start when working with MBR/GPT
- Upon finding the partition location, I'd have to shift my mindset from disk to strictly to the partition I'm working in and acknowledge that there's a filesystem structure (FAT32, NTFS, ext4, ...)
- Gather information within the boot sector which is the first sector of the FAT32 filesystem
  - Information here pertains to sector size, reserved sector count, number of FATs, FAT size, root directory location, and cluster size.
  - This is how I map out the entire partition I'm working in as the boot sector values help me calculate where **FAT #1 & 2 starts**, where the **root directory** starts, and where the **data region (clusters)** starts

**QUESTION APPLICATION**
File B is found in the root directory because it is the only place where files are listed, containing the file name, size, timestamps, and starting cluster number. The root directory should always be consulted first while FAT will tell you where the file continues and the data region will contain the actual information.

Disk
↓
MBR / GPT
↓
Partition start
↓
Boot sector (filesystem map)
↓
Root directory (find File B)
↓
Starting cluster (from directory entry)
↓
FAT #1 (follow cluster chain)
↓
Data clusters (rebuild file)

Hex uses base-16 - reference:
0 1 2 3 4 5 6 7 8 9 A B C D E F (15)

Mental example: If chaining from 5 to 6 would be 06 00 00 00 and 10 comes after F, it would be **10 00 00 00**.
- My issue was that i didn't make much of a mental note of how they are displayed for continuation (i.e., 0D 00.., 0E 00.., 0F 00..,)

So in all, the **value of of the fat entry at cluster F** (to chain to 10) would look like:
- Cluster [F] : 10 00 00 00

## Question 2: Using the FAT32_structure.001 image, answer the following question: At which offset does the  FAT2 table start ( give in the offset value without spaces)? Remember, FAT1 starts right after the Reserved Sectors and FAT2 starts right after FAT1.


