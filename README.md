# brownlab-workflow
This is a tutorial for the phylogenomic workflow used by the Brown lab, where we use [UCEs](https://www.ultraconserved.org/) to uncover evolutionary histories, mostly in Neotropical poison frogs (Dendrobatidae). In this tutorial I provide sample data and take you through the steps of read processing, sequence assembly, read-to-locus matching, and sequence alignment, and finally provide a few examples of phylogenetic analyses that can be performed on UCE data.
## Directory structure and example files
In this tutorial I will be using a Linux machine (named Bender) for all steps. We need to start by creating a directory to put the example data in.  
In my examples I will be using Bash commands on the Linux command line (Terminal), but many of the mundane commands (switching directories, moving files, creating directories), can also be completed simply using the desktop interface. In the following suite of Bash commands, we will move to the desktop, create a folder `tutorial` from which will be working, go into `tutorial`, and create a subfolder `1_raw-fastq`, in which we will place our raw read data.
```
cd ~/Desktop  
mkdir tutorial
cd tutorial
mkdir 1_raw-fastq
```
Subsequent steps will be conducted in numbered folders following the 1_, 2_, 3_... numbering scheme, which will make it easier to keep track of what's going on.  

Now we need to move the provided example files into `tutorial`. Download the following twelve [.fastq](https://support.illumina.com/bulletins/2016/04/fastq-files-explained.html) files from X and place them into `1_raw-fastq`. (Probably easiest to do this via desktop). When you run the following command:
```
 ls 1_raw-fastq/
```
You should see the following:
```
RAPiD-Genomics_GW180505000_SIU_115401_P05_WH08_i5-507_i7-188_R1_001.fastq.gz
RAPiD-Genomics_GW180505000_SIU_115401_P05_WH08_i5-507_i7-188_R2_001.fastq.gz
RAPiD-Genomics_HJYMTBBXX_SIU_115401_P01_WG01_i5-512_i7-84_S1152_L006_R1_001.fastq.gz
RAPiD-Genomics_HJYMTBBXX_SIU_115401_P01_WG01_i5-512_i7-84_S1152_L006_R2_001.fastq.gz
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WC09_i5-505_i7-129_S33_L005_R1_001.fastq.gz
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WC09_i5-505_i7-129_S33_L005_R2_001.fastq.gz
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WF11_i5-505_i7-167_S71_L005_R1_001.fastq.gz
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WF11_i5-505_i7-167_S71_L005_R2_001.fastq.gz
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WB02_R1_combo.fastq.gz
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WB02_R2_combo.fastq.gz
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WD04_R1_combo.fastq.gz
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WD04_R2_combo.fastq.gz
```
These twelve files correspond to six samples:
- ApeteJLB07-001-0008-AAAI (*Ameerega petersi*)
- AbassJB010n1-0182-ABIC (*Ameerega bassleri*)
- AbassJLB07-740-1-0189-ABIJ (*Ameerega bassleri*)
- AtrivJMP26720-0524-AFCE (*Ameerega trivittata*)
- AflavMTR19670-0522-AFCC (*Ameerega flavopicta*)
- AhahnJLB17-087-0586-AFIG (*Ameerega hahneli*)

A bit about the organization and naming of our samples: They are organized into "plates", where each was a batch of samples sent to [RAPiD Genomics](http://rapid-genomics.com/home/) (Gainesville, FL). Each plate folder contains two .fastq files for each sample in that plate, an `info.txt` file that contains the adapters, and a `SampleSheet.csv` that contains valuable information on each sample, including connecting the *RAPiD Genomics* sample name with the *Brown lab* sample name.  
The way sample names work is that there is generally a truncated version of the species name (Apete in the first sample above, short for *Ameerega petersi*), followed by a collection number (JLB07-001, this stands for Jason L. Brown, 2007 trip, sample 001), and then followed by a two part unique ID code (0008-AAAI). The truncated species name is sometimes unreliable, especially for cryptic species such as *A. hahneli*. Generally the unique ID code is the most searchable way to find each sample's info in a spreadsheet. The numerical and alphabetical components of the code are basically the same, so either is searchable by itself (for instance, you can search "0008" or "AAAI" with the same degree of success).  

>*Note that Plate 2 is a bit different than the others in that there are four .fastq files for each sample rather than two. Basically, an additional sequencing run was necessary. We combined both runs for each lane for each sample and put them into the folder `combo` that is located inside the `plate2` folder. Use those files.*  

Also place the various files in the `example-files` folder in this repository directly into the `tutorial` folder on your computer (not a subfolder).
## Read trimming
The first real step in the process is to trim the raw reads with [Illumiprocessor](https://illumiprocessor.readthedocs.io/en/latest/), a wrapper for the [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) package (make sure to cite both). Illumiprocessor trims adapter contamination and low quality bases from the raw read data. It runs fairly quickly, but be warned that this is generally one of the most onerous steps in the whole workflow, because getting it to run in the first place can be fairly challenging. One of the main difficulties comes in making the configuration file, which tells Illumiprocessor what samples to process and how to rename them. 
#### Making the Illumiprocessor configuration file
Since we are only using twelve samples (note that there are two .fastq files per sample), I simply wrote the configuration file by hand without too much difficulty. In cases where you want to process more samples, though, you will want to streamline the process. More on this later.  

First, I will explain the structure of the configuration file. The file consists of four sections, each identified by four headers surrounded by [square brackets]. The first section is listed under the `[adapters]` tag. This simply lists the adapters. They should be the same for all samples. You can get the adapters from the 
## Sequence assembly
## Locus matching
## Sequence alignment
## Phylogenetic analysis
