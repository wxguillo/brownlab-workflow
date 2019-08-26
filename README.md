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

Hopefully your run works the first time. This is generally one of the longest-duration steps in the pipeline - each assembly generally takes an hour to complete with 19 cores. For a set of 96 samples, this process can take most of a working week. I like to run it over a weekend.

When the assemblies have finished, you should have a folder called `3_trinity-assemblies` in your `tutorial` folder. Using the command:
```
ls 3_trinity-assemblies
```
should display:
```
DISPLAY THIS
```
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
## Locus matching
## Sequence alignment
## Phylogenetic analysis
