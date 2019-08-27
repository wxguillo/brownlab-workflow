# brownlab-workflow
This is a tutorial for the phylogenomic workflow used by the Brown lab, where we use [UCEs](https://www.ultraconserved.org/) to uncover evolutionary histories, mostly in Neotropical poison frogs (Dendrobatidae). In this tutorial I provide sample data and take you through the steps of read processing, sequence assembly, read-to-locus matching, and sequence alignment, and finally provide a few examples of phylogenetic analyses that can be performed on UCE data.
## Contents
- [Directory structure and example files](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#directory-structure-and-example-files)
- [Read trimming](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#read-trimming)
   - [Making the Illumiprocessor configuration file](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#making-the-illumiprocessor-configuration-file)
      - [Making the configuration file quickly with Excel](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#making-the-configuration-file-quickly-with-excel)
   - [Running Illumiprocessor](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#running-illumiprocessor)
      - [Troubleshooting Illumiprocessor](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#troubleshooting-illumiprocessor)
- [Sequence assembly](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#sequence-assembly)
   - [Making the assembly configuration file](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#making-the-assembly-configuration-file)
   - [Running Trinity to assemble cleaned reads](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#running-trinity-to-assemble-cleaned-reads)
      - [Troubleshooting Trinity](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#troubleshooting-trinity)
- [Locus matching](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#locus-matching)
- [Sequence alignment](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#sequence-alignment)
- [Phylogenetic analysis](https://github.com/wxguillo/brownlab-workflow/blob/master/README.md#phylogenetic-analysis)
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
### Making the Illumiprocessor configuration file
Since we are only using twelve samples (note that there are two .fastq files per sample), I simply wrote the configuration file by hand without too much difficulty. In cases where you want to process more samples, though, you will want to streamline the process. More on this later.  

First, I will explain the structure of the configuration file. The file consists of four sections, each identified by four headers surrounded by [square brackets]. The first section is listed under the `[adapters]` header. This simply lists the adapters. They should be the same for all samples. You can get the adapters from the readme file in each plate folder (the name is consistent). The section should look like this:
```
[adapters]
i7: GATCGGAAGAGCACACGTCTGAACTCCAGTCAC-BCBCBCBC-ATCTCGTATGCCGTCTTCTGCTTG
i5: AATGATACGGCGACCACCGAGATCTACAC-BCBCBCBC-ACACTCTTTCCCTACACGACGCTCTTCCGATCT
```
This is generally the easiest part. The i7 adapter is the "right" side (if oriented 5' to 3'), and the i5 is the "left".

The second section is under the `[tag sequences]` header. This lists the sequence tags used for each sample. We are using dual-indexed libraries, so there will be *two* tags per sample, one corresponding to the i7 adapter and the other corresponding to i5. This section will look like this:
```
[tag sequences]
i5_P01_WG01:TCTACTCT
i5_P02_WB02:TCAGAGCC
i5_P02_WD04:TCAGAGCC
i5_P04_WG01:CTTCGCCT
i5_P04_WF11:CTTCGCCT
i5_P05_WH08:ACGTCCTG
i7_P01_WG01:CACCTTAC
i7_P02_WB02:GGCAAGTT
i7_P02_WD04:CTTCGGTT
i7_P04_WG01:CACAGACT
i7_P04_WF11:CTGATGAG
i7_P05_WH08:CTGGTCAT
```
Note the structure: to the left of the colon is the tag name (must start with either i5 or i7, and should be unique to the sample), and to the right of the colon is the tag sequence. The sequences can be obtained from the `SampleSheet.csv`. The tag names are up to you.

The third section is under the `[tag map]` header. This connects each sequence tag to a particular sample (remember, there are two per sample). It should look something like this:
```
[tag map]
RAPiD-Genomics_HJYMTBBXX_SIU_115401_P01_WG01:i5_P01_WG01,i7_P01_WG01
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WB02:i5_P02_WB02,i7_P02_WB02
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WD04:i5_P02_WD04,i7_P02_WD04
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WC09:i5_P04_WG01,i7_P04_WG01
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WF11:i5_P04_WF11,i7_P04_WF11
RAPiD-Genomics_GW180505000_SIU_115401_P05_WH08:i5_P05_WH08,i7_P05_WH08
```
The structure is the sample name, separated by a colon from the two corresponding tag names (which themselves are separated by commas).  
Let's talk about the sample name, because it looks like a bunch of gobbledy-gook. This sample name is the one returned by RAPiD Genomics, and is only connected to a particular sample via the `SampleSheet.csv` file. In this case, "RAPiD-Genomics_HJYMTBBXX_SIU_115401_P01_WG01" is the same sample as "ApeteJLB07-001-0008-AAAI". Basically all of it can be ignored except for the last two bits (for this sample, "P01_WG01"). That first bit is the plate the sample was sequenced in, and the second bit is the unique identifier for the sample. Note that different plates will use the same identifiers, so including the plate is important (notice that we have both P01_WG01 and P04_WG01). Also note that this sample name corresponds to the actual filenames of the two corresponding .fastq files (remember, there are two .fastq files per sample). However, it is partially truncated, only going as far as the unique ID of the sample. The full filename for this particular sample is `RAPiD-Genomics_HJYMTBBXX_SIU_115401_P01_WG01_i5-512_i7-84_S1152_L006_R2_001.fastq.gz`, while in our configuration file we only include the name up to the `WG01` part. It is unnecessary to include the rest.

The fourth and final section is under the `[names]` header. It connects the nigh-unreadable RAPiD sample name to a more human-readable name of your choice. The name can be whatever you want, but we're going to make it the actual Brown Lab sample name. It should look like this:
```
[names]
RAPiD-Genomics_HJYMTBBXX_SIU_115401_P01_WG01:ApeteJLB07-001-0008-AAAI
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WB02:AbassJB010n1-0182-ABIC
RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WD04:AbassJLB07-740-1-0189-ABIJ
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WC09:AtrivJMP26720-0524-AFCE
RAPiD-Genomics_HK2T5CCXY_SIU_115401_P04_WF11:AflavMTR19670-0522-AFCC
RAPiD-Genomics_GW180505000_SIU_115401_P05_WH08:AhahnJLB17-087-0586-AFIG
```
The structure is the RAPiD sample name, separated by a colon from the Brown Lab sample name. Note that the first part is identical to the first part of the previous section; we are just renaming samples to which we already assigned sequence tags. Remember, the human-readable sample names are also obtained from the `SampleSheet.csv` file.

An important note is that this is the stage that decides your sample names for the rest of the process, so be careful. A few things that you **must** make sure of are the following:
- There are no underscores in the names (yes, no underscores. Replace them with a - dash. Underscores will screw up the assembly step down the line). 
- There are no "atypical" characters in the names. Basically you should avoid including anything that isn't a letter, a number, or a dash. This includes periods.
- There are no spaces in the names.  

Historically I have found that spaces and underscores are in a few sample names that may make their way into your file. It is often good to `ctrl+F` and search for these, and replace them with - dashes.
#### Making the configuration file quickly with Excel
For large numbers of samples, making the configuration file by hand can be taxing. For six samples, it took me about twenty minutes to gather all of the various data from different sample sheets and organize it (the data being from different plates did not help). For this reason, we have made an Excel spreadsheet in which you can copy and paste relevant information directly from the sample sheets for large numbers of samples, and it will automatically organize the information in the correctly formatted manner for the configuration file. I have provided a file named `UCE_cleanup.xlsx` in the `example-files` directory of this repository that you can use as a template and example.

Essentially the first step is to copy and paste the entire sample sheet, wholesale, into the first sheet of the file. Now you can directly access the various components and combine them in various ways to form the Illumiprocessor configuration file. The `tagi5` sheet creates the i5 portion of the `[tag sequences]` section, and the `tagi7` sheet creates the i7 portion. The `tag map` sheet creates both the `[tag map]` and `[names]` sections of the file. This may require a bit of finagling, because the sample tag (the `[Name for Tag Map]` column) needs to be truncated from the full filename (the `[Sequence_Name]` column). Samples from different plates will usually need to be truncated a different amount of characters, so you may need to modify the equation in the `[Name for Tag Map]` column to suit your needs. Note that in this example, we did not truncate to the unique ID (WH01, WH02, etc.), including a bit more information; this is not really necessary. This step can be a source of downstream errors because you may have truncated more or less than you thought you did, creating an inconsistent tag map section.

Just copy the different sections of the spreadsheet into a text file, and your configuration should be ready to go.
### Running Illumiprocessor
This is the part where things generally go horribly, woefully wrong. I don't think I've ever run this command and had it work the first time. That's because the configuration file needs to be exactly, perfectly correct, which is difficult to do with large numbers of samples. I include a "troubleshooting" section after this one that lists some of the common problems. Here is the command to be used in this particular case:
```
illumiprocessor \
    --input 1_raw-fastq \
    --output 2_clean-fastq \
    --trimmomatic ~/Desktop/BioTools/Trimmomatic-0.32/trimmomatic-0.32.jar \
    --config illumiprocessor.conf \
    --r1 _R1 \
    --r2 _R2 \
    --cores 19
 ```
>*Note that a \ backslash in Bash "escapes" the next character. In this particular command, the backslashes are escaping the invisible \n newline character, so that each argument can be written on a separate line for visual clarity. The entire command is generally written on a single line, but this becomes hard to read as arguments are added.*

You will see that most of the commands we use will be structured in this way. Here is how the command is structured (future commands will not be explained in such detail):
- `--input` requires the input folder containing the raw .fastq.gz files (in this case `1_raw-fastq`)
- `--output` is the name of the output folder, to be created. Note that we are following the aforementioned numbering structure. If the folder already exists, Illumiprocessor will ask if you want to overwrite it.
- `--trimmomatic` requires the path to the `trimmomatic-0.32.jar` file. In this case it is located in a folder called `BioTools` on my Desktop.
- `--config` is the name of the configuration file that we just struggled to make
- `--r1` is an identifier for the R1 reads (notice that the two files for each sample are identical in name, except for a bit saying either `_R1` or `_R2`. This matches that bit). I have been able to run Illumiprocessor without these commands before, but more often than not they are necessary for it to run.
- `--r2` is an identifier for the R2 reads (see above)
- `--cores` specifies the number of cores you use. Generally, the more cores specified, the faster the program will run. I am running this on a computer with 20 cores, so I specify 19 cores, leaving one to be leftover for other tasks.  

Hopefully, when you enter the command into terminal (make sure you are in the `tutorial` main directory and not a subfolder), it says "Running" rather than an error message. This process took my computer only a few minutes to run, but adding samples or using fewer cores will require longer times.

You should now have a folder in your `tutorial` directory named `2_clean-fastq`. Running the following command:
```
ls 2_clean-fastq
```
should yield the following list of samples:
```
AbassJB010n1-0182-ABIC      AflavMTR19670-0522-AFCC   ApeteJLB07-001-0008-AAAI
AbassJLB07-740-1-0189-ABIJ  AhahnJLB17-087-0586-AFIG  AtrivJMP26720-0524-AFCE
```
Note that there are now 6 files rather than 12 (1 per sample rather than 2), and that the samples have been renamed to match Brown Lab names. You can actually tell what they are now!
#### Troubleshooting Illumiprocessor
- `AssertionError: Java does not appear to be installed`: Annoyingly, Illumiprocessor (and the rest of PHYLUCE) runs on Java 7 rather than Java 8, the most current version. As such, I have both versions installed on my computer. This message probably means that you are attempting to run Illumiprocessor with Java 8. To switch (on a Linux machine), use the following command: `sudo update-alternatives --config java`. The terminal will prompt you to enter the computer's password. Do so, and then select which version of Java to use. In this case, you will want to switch to Java 7 (for me, the choice looks like `/usr/lib/jvm/jdk1.7.0_80/bin/java`).
- `AssertionError: Trimmomatic does not appear to be installed`: Ensure you have specified the correct location of the `trimmomatic-0.32.jar` file.
- `IOError: There is a problem with the read names for RAPiD-Genomics_HL5T3BBXX_SIU_115401_P02_WB02. Ensure you do not have spelling/capitalization errors in your conf file.`: This is the most common error you will get running Illumiprocessor, and it has multiple possible solutions (note that the sample name specified is from my example; it will likely differ for you). The first thing to do is to check the spelling of the tag name and ensure it matches the spelling of the corresponding filename. In my case, I had to change `RAPiD-Genomics_HL5T3BBXX_SIU_115401_P02_WB02` to `RAPiD-Genomics_HL5T3BBXX_SIU_115401_SIU_115401_P02_WB02`. The double `SIU_115401_SIU_115401` construction is probably due to this being a sample from Plate 2, which has combined read files that altered the filenames. However, another possible cause of this error is not specifying your `--r1` and `--r2` arguments, either correctly or at all. Assess both possibilities.
## Sequence assembly
The next step is to assemble our raw reads into usable data. This is the first stage at which we will be using the software package [PHYLUCE](https://phyluce.readthedocs.io/en/latest/index.html), written by Brant Faircloth at LSU (who also led the development of UCE sequence capture for phylogenomics). PHYLUCE contains tons of commands for processing UCE sequence data, and encapsulates a lot of other programs. Like Illumiprocessor, it runs on Java 7, so make sure you have set your machine to that version.

Sequence assembly is basically the process of putting your raw sequence data into a format at least reminiscent of the way the DNA was organized in life. Imagine that you have one hundred copies of a book, and you put them all in a paper shredder. Now you have to reconstruct the book from the shredded chunks. Since you have multiple copies, not all of which were shredded in the same way, you can find chunks that partially match and use these matches to connect to the next sentence. Now, however, imagine that the book contains several pages that are just ["All work and no play makes Jack a dull boy"](https://www.youtube.com/watch?v=4lQ_MjU4QHw) over and over. This makes the process much more difficult because this construction causes extreme ambiguity. This is an analogy to how assembly works, and also how repeating DNA elements make assembly difficult. This is why many computational biologists have made their careers based on writing powerful de novo assembly programs.

PHYLUCE can assemble sequences using the programs [velvet](https://www.ebi.ac.uk/~zerbino/velvet/), [ABySS](http://www.bcgsc.ca/platform/bioinfo/software/abyss), and [Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki). Out of these, ABySS is probably regarded as "the best" in terms of assembly accuracy, but Faircloth recommends using Trinity out of a combination of good speed, and providing longer contigs. I have used Trinity in all of my Brown Lab projects and will use it in this tutorial.

Just like with Illumiprocessor, the first thing we need to do is... make a configuration file!
### Making the assembly configuration file
Compared to the Illumiprocessor config file, the assembly one is very easy to make (and can be downloaded as `assembly.conf` from the `example-files` directory in this repository). You can make it with a simple Bash script:
```
cd 2_clean-fastq
echo "[samples]" > ../assembly.conf
for i in *; \
   do echo $i":/home/bender/Desktop/tutorial/2_clean-fastq/"$i"/split-adapter-quality-trimmed/"; \
   done >> ../assembly.conf
```
The first command moves you into the `2_clean-fastq` directory, and the second initiates a text file with the `[samples]` header. The third command is a `for` loop that prints out the name of each sample (represented by `$i`), followed by a colon and lastly the file path to the `split-adapter-quality-trimmed` folder inside of each sample's individual folder. The `split-adapter-quality-trimmed` folders contain .fastq.gz files with the trimmed reads. The assembler needs to know the location of each of these folders for each sample.

Bash tips:
- The `..` construction means "the containing directory"; in this case, I use it to tell the computer to create the file `assembly.conf` in the containing directory `tutorial` rather than the present directory `2_clean-fastq`
- The `>` sign means "put the results of this command into a file called 'this'", in this case `assembly.conf`
- The `>>` sign is similar to `>`, put instead it appends the results of the command into the file. Using `>` would overwrite that file.
- The `*` wildcard sign matches everything in the directory. You can use this sign in various ways to create matches. For instance, using `*2019` would match all files that end with "2019".
- A `for` loop has the following structure: "for all of the objects in this set; do this; done". The set in this case is `*`, or all of the files in the `2_clean-fastq` directory (e.g., all of the samples). `i` is basically a variable that represents each item in the set. The command "loops" through each possible value of `i`, performing the commands listed after `do` for each one (e.g., each sample). The `done` construct signals the loop to close. 

After running the command, it's usually a good idea to manually check the file to ensure that you have only the `[samples]` header and the sample paths listed. If there were any other files in the `2_clean-fastq` folder, they would be listed in this file as well and need to be removed. The file should look like this:
```
[samples]
AbassJB010n1-0182-ABIC:/home/bender/Desktop/tutorial/2_clean-fastq/AbassJB010n1-0182-ABIC/split-adapter-quality-trimmed/
AbassJLB07-740-1-0189-ABIJ:/home/bender/Desktop/tutorial/2_clean-fastq/AbassJLB07-740-1-0189-ABIJ/split-adapter-quality-trimmed/
AflavMTR19670-0522-AFCC:/home/bender/Desktop/tutorial/2_clean-fastq/AflavMTR19670-0522-AFCC/split-adapter-quality-trimmed/
AhahnJLB17-087-0586-AFIG:/home/bender/Desktop/tutorial/2_clean-fastq/AhahnJLB17-087-0586-AFIG/split-adapter-quality-trimmed/
ApeteJLB07-001-0008-AAAI:/home/bender/Desktop/tutorial/2_clean-fastq/ApeteJLB07-001-0008-AAAI/split-adapter-quality-trimmed/
AtrivJMP26720-0524-AFCE:/home/bender/Desktop/tutorial/2_clean-fastq/AtrivJMP26720-0524-AFCE/split-adapter-quality-trimmed/
```
You should go back into the `tutorial` directory after this. Use:
```
cd ..
```
### Running Trinity to assemble cleaned reads
With the assembly configuration file completed, we can now run Trinity. Use the following PHYLUCE command:
```
phyluce_assembly_assemblo_trinity \
    --conf assembly.conf \
    --output 3_trinity-assemblies \
    --clean \
    --cores 19
```
- `--clean` specifies that you want to remove extraneous temporary files. This makes the output directory much smaller.

Hopefully your run works the first time. This is generally one of the longest-duration steps in the pipeline - each assembly generally takes an hour to complete with 19 cores. For a set of 96 samples, this process can take most of a working week. I like to run it over a weekend. For these six samples, the run took about four and a half hours with 19 cores.

When the assemblies have finished, you should have a folder called `3_trinity-assemblies` in your `tutorial` folder. Using the command:
```
ls 3_trinity-assemblies
```
should display:
```
AbassJB010n1-0182-ABIC_trinity      AhahnJLB17-087-0586-AFIG_trinity  contigs
AbassJLB07-740-1-0189-ABIJ_trinity  ApeteJLB07-001-0008-AAAI_trinity
AflavMTR19670-0522-AFCC_trinity     AtrivJMP26720-0524-AFCE_trinity
```
The assembly has generated a set of six folders (one per sample) as well as a folder named `contigs`. Inside each sample folder, you will find a `Trinity.fasta` file that contains the assembly, as well as a `contigs.fasta` link that links to that .fasta file. The `contigs` folder further contains links to each sample's .fasta file.
#### Troubleshooting Trinity
Generally, the most common error with Trinity will generally be caused by specifying incorrect file paths in your configuration file. Double-check them to make sure they're correct. 

Other possible issues can arise if you copied the trimmed reads over from another directory without preserving folder structure. This can break the symlinks (symbolic links) that are in the `raw-reads` subfolder of each sample's folder. They need to be replaced for Trinity to function. You can do that with the following Bash commands:
```
cd 2_clean-fastq
echo "-READ1.fastq.gz" > reads.txt
echo "-READ2.fastq.gz" >> reads.txt
ls > taxa.txt
for j in $(cat reads.txt); \
   do for i in $(cat taxa.txt); \
         do ln -s $i/split-adapter-quality-trimmed/$i$j $i/raw-reads/$i$j; \
         done; \
   done
cd ..
```
This creates two files: the first, `reads.txt`, contains a list of two file endings that will be looped over to construct proper symlink names; the second, `taxa.txt`, contains a simple list of all of the samples in the `2_clean-fastq` folder. The next command is a set of two nested `for` loops that basically reads as "For both file endings listed in `reads.txt`, and then for every sample listed in `taxa.txt`, generate a symlink in the `raw-reads` directory for that sample that leads to the corresponding .fastq.gz file in the `split-adapter-quality-trimmed` folder for this sample".

More Bash tips:
- The `ln` command generates links. Using the `-s` flag generates symbolic links, which we desire here. The first argument is the file to be linked to, and the second argument is the name and path of the link to be generated.
- The `cat` command at its most basic level prints a file. It stands for "concatenate" and can be used to combine files if you specify more than one. In `for` loops, the construct `$(cat taxa.txt)` (using `taxa.txt` as an example file) can be used to loop over each line in that file.
### Viewing assembly summary stats
You can use a PHYLUCE command embedded in a simple `for` loop to generate a .csv file containing assembly summary stats. You may wish to put some of them in a publication, or to use them to check that your assembly went well.
```
for i in 3_trinity-assemblies/contigs/*.fasta;
do
    phyluce_assembly_get_fasta_lengths --input $i --csv;
done > assembly_stats.csv
```
The loop will loop through every file ending in .fasta located in the `3_trinity-assemblies/contigs` folder, and then process it using the `phyluce_assembly_get_fasta_lengths` script. (Note that these are not actual .fasta files, but links to them.)

In order listed, the summary stats printed to `assembly_stats.csv` will be:
>sample,contigs,total bp,mean length,95 CI length,min,max,median,contigs >1kb
## Locus matching
The next step is going to be a sequence of PHYLUCE commands that essentially takes your assemblies, finds the UCEs inside of them, and then conveniently packages them so that you can later align them.
### Matching contigs to probes
The first of these commands is `phyluce_assembly_match_contigs_to_probes`, which matches your contigs to the set of UCE probes used during sequencing. If you haven't already, download the `uce-5k-probes.fasta` file from the `example-files` directory of this repository and put it in `tutorial`. This .fasta file contains the sequences of these UCE probes. The command is as follows:
```
phyluce_assembly_match_contigs_to_probes \
    --contigs 3_trinity-assemblies/contigs \
    --probes uce-5k-probes.fasta \
    --output 4_uce-search-results
```
For a lot of samples, this command can last long enough to give you a coffee break. For our six samples, it should take only a few seconds. The output will be located in the new folder `4_uce-search-results`. Inside it, you will find six [.lastz](http://www.bx.psu.edu/miller_lab/dist/README.lastz-1.02.00/README.lastz-1.02.00a.html) files, which is another sequence storage format. There will also be a `probe.matches.sqlite` file, which is a database relating each contig to each probe.

If you have issues getting this command to work, it's likely that it's because you copied assemblies over from another directory, which breaks the links in the `contigs` folder of `3_trinity-assemblies`. You can resurrect these links using the `ln -s` command and a `for` loop, or by using it individually for particular samples. You need to link to the `Trinity.fasta` file in that particular sample's assembly directory. You will also need to remake links if you alter a sample's name, say if you forgot to remove periods from names or something like that.
### Extracting UCE locus data
The next portion takes the matched contigs/probes and extracts usable sequence data in .fasta format for each sample, organized by UCE locus. There is a bit of setup we have to do first.
#### Creating taxon sets
The first thing we have to do before we move on is create one or more "taxon sets". By this I mean a set of samples that you would like to work on. For us, this is just going to be the complete set of six samples we've been processing this whole time. But for other projects, you may wish to use a specific subset of samples at times, and the whole set at others. For example, in my own UCE projects I often have a taxon set that is the full set of samples, as well as another, smaller, one with one representative sample per species. 

To create a taxon set, we need to create yet another configuration file. Don't worry, this one is simple. The file just needs a list of the samples to be used, underneath a header in [square brackets] that gives the taxon set a name. Below is a block of code that creates a file called `taxa.txt`, which contains a taxon set called `all` that contains all six samples.
```
echo "[all]" > taxa.txt
ls 4_uce-search-results >> taxa.txt
```
This is a very crude script because we do have to go in and edit the file. It wouldn't be too hard to write a script that makes the file without any other intervention, but it's not hard to make the edits manually. If you open up the file in a program like gedit, it should look like this:
```
[all]
AbassJB010n1-0182-ABIC.contigs.lastz
AbassJLB07-740-1-0189-ABIJ.contigs.lastz
AflavMTR19670-0522-AFCC.contigs.lastz
AhahnJLB17-087-0586-AFIG.contigs.lastz
ApeteJLB07-001-0008-AAAI.contigs.lastz
AtrivJMP26720-0524-AFCE.contigs.lastz
probe.matches.sqlite
```
Notice the `[all]` header that gives the taxon set its name. The two things we need to do are:
- Remove all listed names that are not samples (in this case, `probe.matches.sqlite`)
- Remove all file endings so that all that's listed are sample names. A simple search-and-replace can take care of this (ctrl+H in gedit).

The final `taxa.txt` file should look like this:
```
[all]
AbassJB010n1-0182-ABIC
AbassJLB07-740-1-0189-ABIJ
AflavMTR19670-0522-AFCC
AhahnJLB17-087-0586-AFIG
ApeteJLB07-001-0008-AAAI
AtrivJMP26720-0524-AFCE
```
If you have more than one taxon set, you can put all of them in the same file, or put them in separate files.

Next we need to create a file structure, basically a folder for each taxon set. We'll be doing most of the remainder of our work in that folder. Use the following command:
```
mkdir -p 5_taxon-sets/all
```
The `mkdir` command simply makes a directory. The `-p` flag allows you to make nested directories. We now have a directory named `5_taxon-sets` that contains another directory called `all` that corresponds to our `[all]` taxon set. If we had more than one taxon set in our `taxa.txt` file, we would create a directory for each of them inside `5_taxon-sets`.
#### Getting .fasta files for each sample and UCE locus
Next up is a set of commands that takes our .lastz files and .sqlite database, and turns them into .fasta files for each UCE locus and each sample in our taxon set. The first command is `phyluce_assembly_get_match_counts`, used below:
```
phyluce_assembly_get_match_counts \
    --locus-db 4_uce-search-results/probe.matches.sqlite \
    --taxon-list-config taxa.txt \
    --taxon-group 'all' \
    --incomplete-matrix \
    --output 5_taxon-sets/all/all-taxa-incomplete.conf
```
This command generates a configuration file, `all-taxa-incomplete.conf`, that is located in `5_taxon-sets/all`.  
Now we're going to go into `5_taxon-sets/all` and make a `log` directory:
```
cd 5_taxon-sets/all
mkdir log
```
Then we use the following command
```
phyluce_assembly_get_fastas_from_match_counts \
    --contigs ../../3_trinity-assemblies/contigs \
    --locus-db ../../4_uce-search-results/probe.matches.sqlite \
    --match-count-output all-taxa-incomplete.conf \
    --output all-taxa-incomplete.fasta \
    --incomplete-matrix all-taxa-incomplete.incomplete \
    --log-path log
```
This is a good command to run over your lunch break if you have a lot of samples. For us, it should only take about a minute or two. This command generates a big .fasta file, `all-taxa-incomplete.fasta`, that contains each sample's set of UCE loci. I recommend against attempting to open the file in a GUI program to view it, as it is probably so big that it'll lock up your computer. You can use `less -S all-taxa-incomplete.fasta` to view it seamlessly in Terminal.

If you look at the to-screen output of the previous command, it will tell you how many UCE loci were recovered for each sample. We targeted around 5,000 loci in total, but for most samples we retain ~1,500. This is pretty normal for our poison frog samples.
#### Getting summary statistics for our UCE loci
We can use a few commands to look at summary stats pertaining to the UCE loci for each sample. First we need to "explode" the huge .fasta file we generated in the previous step to create a separate .fasta file for each sample.
```
phyluce_assembly_explode_get_fastas_file \
    --input all-taxa-incomplete.fasta \
    --output-dir exploded-fastas \
    --by-taxon
```
This generates a folder `exploded-fastas` that contains six .fasta files, one for each sample, containing the (unaligned) UCE loci for that sample. Next use this command to generate summary stats:
```
for i in exploded-fastas/*.fasta;
do
    phyluce_assembly_get_fasta_lengths --input $i --csv;
done
```
This is the same command that you might have used to get assembly summary statistics earlier. Like that case, the output will be organized as a .csv file with the following columns:
>sample,contigs,total bp,mean length,95 CI length,min,max,median,contigs >1kb
## Sequence alignment
The final step before we get to the phylogenetic analysis is sequence alignment. Like sequence assembly, alignment is a classic problem in bioinformatics that has built the careers of several computational biologists. Basically, we need to "align" DNA sequences (think of strings of ATGCGCGTACG... etc.) for each of our samples, so that homologous base pairs are located in the same column. This is a difficult problem because divergent taxa can have very different sequences even for the same gene, making the question of "are these base pairs actually homologous?" fairly nebulous. The desired end results of sequence alignment is a matrix where every column corresponds exactly to homologous base pairs for each taxon (which are represented by rows). The phylogenetics program you use will compare the sequences in the alignment to determine their evolutionary relationships. More divergent sequences should lead to those species being further removed from each other in the phylogeny.

PHYLUCE is integrated with the alignment programs [MAFFT](https://mafft.cbrc.jp/alignment/software/) and [MUSCLE](http://www.drive5.com/muscle/). Faircloth recommends using MAFFT, but it's mostly a matter of personal preference. As a creature of habit, I always tend to use MUSCLE, and will be using that program in this tutorial. Also note that PHYLUCE will perform edge-trimming of alignments (basically an alignment cleaning step) unless otherwise specified. Faircloth recommends this for "closely-related taxa" (<30-50 Ma), which our poison frogs are, so we will allow the edge-trimming.

To perform per-locus alignments, use the following command:
```
phyluce_align_seqcap_align \
    --fasta all-taxa-incomplete.fasta \
    --output-format nexus \
    --output muscle-nexus \
    --taxa 6 \
    --aligner muscle \
    --cores 19 \
    --incomplete-matrix \
    --log-path log
```
- The `--output-format` flag specifies the format of the alignment output. We are using [nexus](http://wiki.christophchamp.com/index.php?title=NEXUS_file_format) format here, a popular format that contains a bit more information than .fasta.
- The `--taxa` flag specifies the number of taxa in the alignment. I have missed changing this before, and didn't encounter any problems, but it may have a role in parallelization or something performance-related.
- The `--aligner` flag is used to specify either `muscle` or `mafft`. 
- Remember to specify the proper number of cores for your machine.

This command creates a folder called `muscle-nexus` that has individual per-locus .fasta files, each containing a sequence alignment for that locus. You can easily check how many loci you have by checking how many .fasta files are in this folder; in our case, we have 2,019 (what a coincidence).
### Locus filtering
Locus filtering is the final step before phylogenetic analysis can happen. Filtering out uninformative or largely incomplete loci can improve performance and efficiency. In our pipeline there are generally two types of locus filtering we'll be performing:
- **Filtering by completeness:** This step is technically optional but you should still do it. It removes loci that are only possessed by a few taxa, which can bias your results if left alone.
- **Filtering by parsimony-informative sites:** This is an optional step that can be useful if you want to filter to a pre-specified number of loci, or further pare down your locus set to be even more informative. A [parsimony-informative site](https://en.wikipedia.org/wiki/Informative_site) is a column in the alignment for which there are at least two different character states, each possessed by at least two taxa. This means that the site can be used to differentiate clades for that character.
#### Filtering by completeness
Filtering by completeness means that you are removing loci that are possessed only by a number of taxa below a certain threshold. Another way to think about it is that if you have a 75% complete matrix, it means that all retained loci are possessed by at least 75% of taxa. So, as you increase completeness, it generally *decreases* the number of loci that will be retained, which sounds paradoxical at first. We use the following command to retain a 75% complete matrix:
```
phyluce_align_get_only_loci_with_min_taxa \
	--alignments muscle-nexus \
	--taxa 6 \
	--percent 0.75 \
	--output muscle-nexus-75p \
	--cores 19 \
	--log-path log
```
After using this command, we generate a new folder called `muscle-nexus-75p` (the 75p stands for 75 percent completeness). We filtered down from 2,019 to 1,665 loci. You can try other levels of completeness to see how it affects your retained-locus count.
For instance, when I retain only loci possessed by 100% of taxa (e.g., all 6), I only retain 416 loci. Generally, as you increase the number of taxa, the odds of any locus being possessed by all of the taxa become lower. When I was working with >200 taxa in a previous project, a 100% complete matrix retained zero loci (meaning it was useless).

After filtering for completeness, we need to "clean up" the locus files. This can be done with the following command:
```
phyluce_align_remove_locus_name_from_nexus_lines \
    --alignments muscle-nexus-75p \
    --output muscle-nexus-clean-75p \
    --cores 19
```
This adds a new folder called `muscle-nexus-clean-75p` that contains the same alignments as `muscle-nexus-75p`, but the locus names have been removed from the taxon names in the file. We can go ahead and remove the "uncleaned" version of the folder:
```
rm -r muscle-nexus-75p
```
`rm` is the "trash" command of Bash. The `-r` flag specifies that you want to trash "recursively", meaning that you can trash a directory as well as all of the files and directories inside of it.
#### Filtering by parsimony-informative sites
Filtering by parsimony-informative sites (PIS) generally means you are filtering out loci that have below a certain number of PIS. Loci with low information content can bias your results so it's good to filter them out. To calculate how many PIS are in each locus, and perform various types of filtering, we will use an R script written by Brown lab alumnus [Connor French](https://github.com/connor-french) that makes use of the package [Phyloch](https://rdrr.io/github/fmichonneau/phyloch/man/phyloch-package.html). If you haven't already, download `pars_inform.R` from the `example-files` directory of this repository.

To use R in the terminal, you have to have it installed on your system. Then just type in `R`. Until you leave R, the terminal will now assume all of your commands are written in R, instead of Bash.

Generally my procedure here is to simply open `pars_inform.R` in a text editor and copy-and-paste commands in chunks into the terminal. You will need to alter parts of the file to suit your needs. The first part of the file looks like this:
```
#script to count parsimony informative sites and output to csv
#load phyloch
library(phyloch)

#setwd where the reads are
setwd("/home/bender/Desktop/tutorial/5_taxon-sets/all/muscle-nexus-clean-75p")
```
`setwd` is the command that sets your "working directory," which is where R assumes any files specified (and written) will be located. Here I've set the working directory to `5_taxon-sets/all`. You may need to alter it depending on your directory structure.

Next run the following chunk of code, which does most of the dirty work in calculating the PIS in each locus and other associated information. It will take a minute to finish.
```
#get list of file names and number of files
listoffiles <- list.files(pattern="*.nex*")
nooffiles <- length(listoffiles)

#these are the column names
record <- c("locusname","pis","length")

#loop to calculate PIS and write the PIS info to a text file
for (j in 1:nooffiles) {
  write.table((gsub("?","N",(readLines(listoffiles[j])),fixed=TRUE)),"list_of_pis_by_locus.txt",sep="",quote=FALSE,row.names=FALSE,col.names=FALSE)
  tempfile <- read.nex("list_of_pis_by_locus.txt")
  templength <- dim(tempfile)[2]
  temppis <- pis(tempfile)
  temp <- cbind(listoffiles[j],temppis,templength)
  record <- rbind(record,temp)
}
#add column names to text file
write.table(record, "list_of_pis_by_locus.txt",quote=FALSE, row.names=FALSE,col.names=FALSE)
```
This command will output a file named `list_of_pis_by_locus.txt` in your `muscle-nexus-clean-75p` directory that contains a list of every locus, its number of PIS, and the locus' length.

Now you need to decide what kind of filtering to perform. I have performed two types of filtering for various projects: The first is the most common, when you went to retain only loci that have PIS within a certain range of values. First, run this chunk:
```
###calculate summary data and visualize PIS variables###
par_data <- as.data.frame(read.table("list_of_pis_by_locus.txt", header = TRUE))
plot(par_data$pis) #visualize number of PIS
```
This should print a plot to the screen of each locus (x) and its number of associated PIS (y). In my case, the number of PIS is strongly clustered below 5 for most loci, with many of them having 0 PIS. Here is my graphical output:  
![PIS graph](https://i.imgur.com/yX7hNya.png)  
You can use this graph to inform what you want your thresholds for retaining PIS to be. When doing this, I generally remove low-informative sites as well as highly-informative outliers. Given this distribution, I'd like to remove loci with fewer than 3 PIS, and more than 15 (3 < PIS < 15). Use the following code block:
```
###calculate summary data and visualize PIS variables###
par_data <- as.data.frame(read.table("list_of_pis_by_locus.txt", header = TRUE))
plot(par_data$pis) #visualize number of PIS
inform <- subset(par_data, pis >= 3) #subset data set for loci with PIS > 3
inform <- subset(inform, pis < 15) #remove outliers (change based on obvious outliers to data set)
plot(inform$pis) #visualize informative loci
length(inform$pis) #calculate the number of informative loci
sum(inform$pis) #total number of PIS for informative loci
fivenum(inform$pis) #summary stats for informative loci (min, lower quartile, median, upper quartile, max)
inform_names <- inform$locusname #get locus names of informative loci for locus filtering
#write locus names to a file
write.table(inform_names, file = "inform_names_PIS_3.txt")
```
This writes a file named `inform_names_PIS_3.txt` to `muscle-nexus-clean-75p`. The file contains the names of all UCE loci that meet the criteria you set in the above code block (3 < PIS < 15).

In one of my projects, I desired instead to find the 200 *most parsimony-informative* loci rather than filter for an unknown number that fit specific criteria. This involves ranking the loci by number of PIS and then taking the top 200 (or whatever number you want). To do this, run this code block:
```
#find 200 most parsimony-informative loci and save their names
pistab <- read.delim("list_of_pis_by_locus.txt", header=TRUE, sep = " ")
pistab_sorted <- pistab[order(-pistab$pis),]
pistab_sorted_truncated <- pistab_sorted[1:200,]
topnames <- pistab_sorted_truncated$locusname
write.table(topnames, file = "top200names.txt")
```
This writes a file named `top200names.txt` to `muscle-nexus-clean-75p` that is similar in structure and principle to `inform_names_PIS_3.txt`. 

Now we have a list of the loci we desire (whether loci that fit a certain range of PIS or the most informative X loci). We now need to use this list to grab the .nexus alignment files for those loci. First we need to modify the list file itself to be in a usable format. Currently, the list file should look something like this:
```
"x"
"1" "uce-4885.nexus"
"2" "uce-2564.nexus"
"3" "uce-4143.nexus"
"4" "uce-7695.nexus"
"5" "uce-869.nexus"
...
```
We need to change it so that all that remains are .nexus filenames. We can use regular expressions to do this. If you don't know, regular expressions are code constructs that are commonly used to search and replace specific patterns in text. They are very intimidating to look at at first, but learning them can make you a badass. You can use Bash commands like `sed` and `awk` to use regular expressions in the Terminal, but we can also do it using the find-and-replace (ctrl+H) utility in gedit. Other text editor programs like Notepad++ also support the use of regular expressions. 

First, go ahead and manually remove the first line `"x"`. Then, open find-and-replace and make sure the "Regular expression" box is ticked. In the "Find" box, type in `"\d+" "(.*.nexus)"`. This is the "search" portion of the regular expression. Then, in the "Replace" box, simply type `\1`. This is the "replace" portion. If you click "Find", the program should highlight everything in the file. Click "Replace All", and you should be left with only the file names, no quotes. Doesn't that feel good? The file should now look like this:
```
uce-4885.nexus
uce-2564.nexus
uce-4143.nexus
uce-7695.nexus
uce-869.nexus
...
```

> Regular expression explained: We build the regular expression sequentially to match each line. We need to find the commonalities between each line and represent them using different constructs. First we use a `"` quote to match the starting quote of each line. Then, we use `\d+` to match "one or more digits", which are in the first column. We add `" "` to match the quote, space, quote, that follows the first number. Next we use `.*` as a "wildcard" (analogous to the `*` of Bash), which matches "anything except a line break zero or more times". This construct matches the "uce-4885" part of the first line. Then we close with `.nexus"`, which matches the last portion of the line. Note that the construct `.*.nexus` is enclosed in (parentheses). We do this to "capture" whatever is inside of the parentheses, which we can then use to Replace our match. The Replace construct `\1` signifies the first "captured" match, which is that bit in the parentheses. So basically, we are searching for the whole line, capturing the part we want (the filename), and then replacing the whole line with the capture.  

Now that that's done, we have to take our cleaned list of loci and grab the corresponding .nexus alignment files. First, quit R with `q()`. Then, enter the following Bash commands:
```
mkdir muscle-nexus-clean-75p_3
cd muscle-nexus-clean-75p
for n in $(cat inform_names_PIS_3.txt); \
   do cp "$n" ../muscle-nexus-clean-75p_3; \
done
cd ..
```
We first create a new directory `muscle-nexus-clean-75p_3` to place our filtered loci in (the `_3` part signifies that we filtered loci with fewer than 3 PIS). Then, we go into the `muscle-nexus-clean-75p` directory. The final section is a `for` loop that, for every line in `inform_names_PIS_3.txt` (i.e., for every locus with 3 < PIS < 15), copy that locus' corresponding .nexus file from `muscle-nexus-clean-75p` to `muscle-nexus-clean-75p_3`. Finally, we go back to the `all` directory to prepare for the next step.

After running the command, `muscle-nexus-clean-75p_3` contains 385 loci. That's quite a lot of filtering. You may wish to filter less stringently in order to retain more loci. I'll be working on the loci in this folder for the remainder of the tutorial.
### Concatenating alignments
The final step before phylogenetic analysis is to concatenate your set of filtered loci into a single alignment. Concatenation in this context means that your alignments will be combined into a single file, and lined up one after another. Note that concatenation is a problematic assumption in phylogenetics and is mostly used for convenience; this is because you're assuming that all of your loci are evolving as a "supergene", when we know in fact this is not the case. We can use "partitions" to divide the concatenated matrix into portions corresponding to each locus, and then assign a different substitution model to each; while more technically "correct," this does have the side effect of making your phylogenetic analysis take way longer. In this tutorial we will not be using partitions, although I will show you how to generate them automatically if you want to use them.

To concatenate the loci in `muscle-nexus-clean-75p_3`, use this command:
```
phyluce_align_format_nexus_files_for_raxml \
    --alignments muscle-nexus-clean-75p_3 \
    --output muscle-nexus-iqtree-75p_3 \
    --charsets
```
The output will be a new folder named `muscle-nexus-iqtree-75p_3` that contains two files: `muscle-nexus-clean-75p_3.phylip`, a [PHYLIP](http://scikit-bio.org/docs/0.5.0/generated/skbio.io.format.phylip.html)-formatted alignment file containing all 385 retained loci, and `muscle-nexus-clean-75p_3.charsets`, a list of partitions that was generated because we specified the `--charsets` option. This list specifies the location of each locus in the alignment and can be used to specify partitions for phylogenetic analysis, although we will not be using it.
## Phylogenetic analysis
Now we're finally at the good part. This isn't intended to be a comprehensive guide for constructing phylogenies but I will show you how to perform some basic analyses using various methods. The previous steps outlined in this guide are basically a pipeline for preparing your data for this step, which is the actual data analysis part of your project. 

Now that we're doing using PHYLUCE, you should switch back to Java 8 using `sudo update-alternatives --config java`.
### Maximum likelihood analysis with RAxML
I think of maximum likelihood (ML) methods as like the peanut-butter-and-jelly sandwich of phylogenetics. A PB&J isn't the most delicious, rigorously constructed sandwich (like a Bayesian Lettuce and Tomato), but it's certainly better than your grandpa's Parsimonious Mayonnaise-only sandwich. A PB&J is serviceable, quick, and always there when you need it. There are several ML methods commonly in use for phylogenetics, but by far the most widely-used is [RAxML](https://cme.h-its.org/exelixis/web/software/raxml/index.html).  
First, make a new RAxML directory, copy your PHYLIP file into it, and go into it yourself:
```
mkdir muscle-nexus-raxml-75p_3 
cp muscle-nexus-iqtree-75p_3/muscle-nexus-clean-75p_3.phylip muscle-nexus-raxml-75p_3/
cd muscle-nexus-raxml-75p_3 
```
Here's the RAxML command I used:
```
~/Documents/RAxML/raxmlHPC-PTHREADS-AVX \
	-f a \
	-x $RANDOM \
	-p $RANDOM \
	-# autoMRE \
	-m GTRGAMMA \
	-s muscle-nexus-clean-75p_3.phylip \
	-n muscle-nexus-raxml_75p_3 \
	-T 19
```
- `-f` selects the algorithm to use. The `a` argument selects a rapid bootstrap analysis and best-tree search in one run.
- `-x` random seed for rapid bootstrapping. `$RANDOM` is a UNIX variable that specifies a random number.
- `-p` random seed for parsimony inferences.
- `-#` number of alternative runs on distinct starting trees. `autoMRE` specifies majority-rule-tree-based boostopping criteria.
- `-m` select the model of evolution. `GTRGAMMA` specifies the general-time-reversible nucleotide substitution model with a gamma model of rate heterogeneity. One of RAxML's main weaknesses is that GTRGAMMA and variations of it are really the only substitution models it offers.
- `-s` input alignment file.
- `-n` name of output file.
- `-T` number of threads to use (cores).

To see more of RAxML's options, you can type `~/Documents/RAxML/raxmlHPC-PTHREADS-AVX -h | less -S`. Note that when I call RAxML (`~/Documents/RAxML/raxmlHPC-PTHREADS-AVX`) I am specifyng the path to its location, which is in my Documents folder. Yours may differ.

Running only 6 taxa, 385 loci, and over 19 cores, the analysis finished in a matter of seconds. You can view the tree by installing [FigTree](https://github.com/rambaut/figtree/releases). To open it on a Linux machine, simply type `figtree` into the Terminal (note that you have to be running Java 8, not Java 7). Open the file `RAxML_bipartitions.muscle-nexus-raxml_75p_3`. Figtree contains numerous options for manipulating images of phylogenetic trees. You can reroot, rotate branches, adjust branch widths, create a circle tree, and more. Here is a modified image of my RAxML tree:  

![raxml tree](https://i.imgur.com/iVswDBF.png)  
This is more or less in line with what we know of the *Ameerega* phylogeny. The node labels are boostrap values, a measure of support for that node. You can add them by selecting "label" as the displayed value for node labels in FigTree.
### Maximum likelihood analysis with IQ-TREE
RAxML is very popular, but I actually prefer to use [IQ-TREE](http://www.iqtree.org/), an alternative ML phylogenetics program. I like IQ-TREE because it has great documentation (and a flashy website), flexible and easy-to-understand options, and most importantly it integrates model selection with a wide array of substitution models, certainly more than are offered by RAxML. I generally use IQ-TREE for all of my ML analyses. 

We already made a directory for IQ-TREE when we concatenated our alignments. Go ahead and go into it with `cd muscle-nexus-iqtree-75p_3`. Then run the following command:
```
iqtree \
	-s muscle-nexus-clean-75p_3.phylip \
	-bb 10000 \
	-m GTR \
	-nt 19
```
- `-s` input alignment file.
- `-bb` number of ultrafast bootstrap replicates. Another advantage of IQ-TREE is its super-fast bootstrapping algorithm, which allows you to go for a larger number of bootstraps in a reasonable amount of time. Make sure to cite [Minh et al. 2013](https://academic.oup.com/mbe/article/30/5/1188/997508) if you use it.
- `-m` substitution model. I use GTR to be roughly equivalent to RAxML. You can also use `-m MFP` to initiate ModelFinder Plus, which will find the best-fit substitution model before running the tree search. 
- `-nt` number of threads (cores).
There are many many more options and things you can do with IQ-TREE - this is about as basic of an analysis as you can do with it. The output tree is stored in the file `muscle-nexus-clean-75p_3.phylip.treefile`, which I show below:  

![iqtree tree](https://i.imgur.com/pWzz7Zg.png)  
As you can see, it is identical in topology to my RAxML tree, but with slightly higher overall bootstrap values and slightly different branch lengths. When comparing the two programs, I find that this is a consistent pattern.
