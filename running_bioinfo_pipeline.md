Artic pipeline
================
Kirstyn Brunker
2023-06-29

- <a href="#1-bioinformatic-pipeline" id="toc-1-bioinformatic-pipeline">1
  Bioinformatic pipeline</a>
  - <a href="#11-log-in-to-climb" id="toc-11-log-in-to-climb">1.1 Structure
    of Artic-nf directory</a>
  - <a href="#12-notes-on-these-instructions"
    id="toc-12-notes-on-these-instructions">1.2 Mandatory parameters 
    required to run the workflow</a>
  - <a href="#13-activate-the-conda-environment"
    id="toc-13-activate-the-conda-environment">1.3 Activate the conda
    environment</a>
  - <a href="#14-data-organisation" id="toc-14-data-organisation">1.4 Data
    organisation</a>
  - <a href="#15-data" id="toc-15-data">1.5 Data</a>
  - <a href="#16-gather-the-data" id="toc-16-gather-the-data">1.6 Gather the
    data</a>
  - <a href="#17-medaka-pipeline" id="toc-17-medaka-pipeline">1.7 Medaka
    pipeline</a>
  - <a href="#18-looking-at-the-data" id="toc-18-looking-at-the-data">1.8
    Looking at the data</a>
  - <a href="#19-bam-files" id="toc-19-bam-files">1.9 BAM files</a>

# 1 Bioinformatic pipeline

During this practical session will be processing some output data from a
nanopore sequencing run. We will use an existing pipeline that contains
commands to take the raw reads through the necessary steps to obtain a
final consensus sequence. A consensus sequence is used for downstream
analyses like building phylogenetic trees.

Our starting point today is basecalled and demultiplexed reads, we
allowed MinKNOW to perform these tasks for us.

The main 2 commands that we will use today are: 1. **artic guppyplex** -
combines all a samples FASTQ reads into a single file and size filters
them (it can also perform a quality score check, which is not needed
here as the reads are already split into pass and fail folders based on
quality by guppy) 2. **artic minion** - aligns the reads to the
Wuhan-Hu-1 reference sequence, trims the amplicon primer sequences from
the aligned reads, downsamples amplicons to reduce the data, and creates
a consensus sequence utilising medaka for variant calling to correct for
common MinION errors (such as those associated with homopolymer
regions).

## 1.1 Structure of Artic-nf directory

Artic-nf workflow contains 6 major components
- <a href="https://github.com/RAGE-toolkit/Artic-nf/blob/main/environment.yml">Evnironment setup</a>
- <a href="https://github.com/RAGE-toolkit/Artic-nf/blob/main/main.nf">Main workflow</a>
- <a href="https://github.com/RAGE-toolkit/Artic-nf/tree/main/meta_data">Meta_data directory</a>
- <a href="https://github.com/RAGE-toolkit/Artic-nf/tree/main/modules">Modules</a>
- <a href="https://github.com/RAGE-toolkit/Artic-nf/blob/main/nextflow.config">Config file</a>
- <a href="https://github.com/RAGE-toolkit/Artic-nf/tree/main/raw_files/fast5">Raw files</a>
- <a href="https://github.com/RAGE-toolkit/Artic-nf/tree/main/scripts">Scripts</a>
- <a href="">Results</a>

<b> Evnironment setup: </b> Conains necessary modules/tools required to
support the workflow. It is recommended use the  <b>environment.yml</b> 
file to setup the workflow. Alternatively, manual_package_install.txt can be used to install the package
manually when the environment.yml is unable to run sucessfully. 

<b> Main workflow: </b> This enables the execution of the entire workflow.
Generally all the sub-tasks are stiteched together and executed by the 
"main.nf" module.

<b> Meta_data directory: </b> This directory contains the sample_sheet,
providing information about the barcode and its corresponding sample 
names.

<b> Modules: </b> All the sub workflow's are stored here, enabling in
easy maintanance, replacement and troubleshooting.

<b> Raw files: </b> It is recommended to store the raw fast5/pod5 files
inside raw_file directory to keep the well organized project directory. 
Althought program can accept the raw_files from any directory. 

<b> Scripts: </b> Any programs/scripts used in the workflow, other than 
*.nf are saved in this directory.

<b> Results: </b> All the workflow outputs are stored here in a sub-folders.
This directory is generated during the workflow run. It can be "results" or
"analysis" or in other names (as per the user defination).

Tree structure below may give you a better idea of how the files are organized
in Artic-nf.

<img src="images/Artic-nf_dir_structure.png"/>

## 1.2 Mandatory parameters

The workflow requires some of the mandate parameters to run the workflow, this 
includes

- output_dir: To store all the analysis results
- meta_file: Contains barcode and their sample details along with primer schema
  and version
- basecaller: Enables user to select basecaller software [Guppy or Dorado]
- fast5_or_pod5_dir: Directory for raw files
- primer_schema: Directory of primer_schema
- kit_name: Barcode kit name for demux
- dorado_dir: Location of the Dorado software (not required if Guppy is used)
- guppy_dir: Location of the Guppy software (not required if Dorado is used)

## 1.3 Tasks run by the Artic-ng workflow
Artic-nf workflow runs multiple tasks symoultaniously, Here are the list of
tasks run by the Artic-nf.
- Basecalling: Basecalling is performed by the user specified basecalling
  tool
- Barcodering: Performs demultiplexing for the basecalled fastq files
- Plex/demux: Aggregate pre-demultiplexed reads
- Medaka: This steps performs multiple steps related to medaka and data filtering
  Some of the major steps are mentioned below.
  - Alignment
  - Variant calling
  - Variant filter
- Concat: Concatinating all the sequences to a single fasta file
- Mafft: Performing mafft alignment on the concatinated sequence

The medaka step executes multiple commands, if you are curious to know about the
commands you can have a look at 

```shell
less results/medaka/<sample_name>.minion.log.txt
```

## 1.4 Notes on these instructions

This is not simply a copy and paste exercise! The commands you are
instructed to run require some editing, e.g. to tell the pipeline where
your data is. Parts of the commands with \<*some text*\> are highlighing
sections that need your input- i.e. you need to edit the code.

## 1.5 Activate the conda environment

First we need to ensure we have access to all the tools needed to run
the pipeline commands. We have a custom conda environment specifically
for this: Artic-nf

``` shell
conda activate Artic-nf
```

## 1.6 Data organisation

It important that raw data is left untouched - we don’t want to risk
modifying these files. We can use it for input but not direct
manipulation of the data. It is best to create a well defined space for
any processed data. Now let us have a look at number of raw data available
and get some more details of it.

------------------------------------------------------------------------

### 1.6.1 Task 1

### 1.6.2

list the number of raw files available in raw_file directory:

``` shell
ls -lh raw_files/fast5/
```

The '-l' in the command to list all the files in a given directory and 'h' 
for providing human redable information.

### 1.6.2 Task 2

Similarly, du command can be used provide the disk space occupied by certain
directory or files

``` shell
du -h /home/rage/workshop_dir/Artic-nf
```

The "-h" parameter provides the human readable information, such as showing
each file size in MB/GB instead of bytes. Alternatively you can try without "h"
option to see how the du command prints.

```shell
du /home/rage/workshop_dir/Artic-nf
```

------------------------------------------------------------------------

You will direct all of your output files, generated in the rest of the
tutorial, to this folder.  
That means any time you see a command that contains `<output>` you must
use this file path to precede the name of your output files to tell the
command line where to store the files. For example, I might generate a
new file that I will call sample1, my `<output>` file path would
therefore be:

``` shell
/home/jovyan/analysis/workshop_data/sample1
```

------------------------------------------------------------------------


## 1.7 Running the workflow
Artic-nf can be run in two ways. One with passing all the mandate parameters 
to the terminal.

``` shell
nextflow main.nf \
 --meta_file "meta_data/sample_sheet.csv" \
 --fast5_dir "projects/fast5/" \
 --guppy_dir "projects/ont-guppy-cpu/bin/" \
 --primer_schema "projects/Artic-nf/meta_data/primer-schemes/" \
 --guppy_barcode_kits "EXP-NBD104" \
 --output_dir "results"
```

Another with editing all the mandate parameters in the 
**nextflow.config** file. The second method is useful if you want to safely
store the parameters for future use. 

``` shell
nextflow main.nf
```
________________________________________________________________________


## 1.8 Data

We will be working from some rabies virus sequence data from the
Philippines. This data was generated on a real-time sequencing run,
which means that the raw signal data (caused by disruption to chemical
signal when DNA passes through the pore) has been basecalled to genetic
code. The basecalled data is in fastq files, file formats that contain
read sequence data and associated quality scores. These files have also
been live demultiplexed i.e. during the run MinKNOW separated the reads
into barcode folders.

The fastq files are in the location:
`home/rage/workshop_dir/raw_files/fastq/

------------------------------------------------------------------------

### 1.8.1 Task 2

Navigate to the fastq_pass directory using the command line

### 1.8.2 Task 3

List the contents of the folder. How many and what barcodes were used in
the run?

------------------------------------------------------------------------

### 1.8.3 Fastq files

The fastq format is (usually) a 4 line string (text) data format
denoting a sequence and it’s corresponding quality score values. There
different ways of encoding quality in a .fastq file however, files from
ONT sequencing devices use sanger phred scores. A sequence record is
made up of 4 lines:

line 1: Sequence ID and Sequence description  
line 2: Sequence line e.g. ATCGs  
line 3: plus symbol (can additionally have description here)  
line 4: Sequence line qualities

For example a sample record looks like:

A fastq file may contain multiple records. The default number of records
in a fastq file generated during a nanopore run is 4000 reads (16000
lines).

------------------------------------------------------------------------

### 1.8.4 Task 4

Look inside one of the barcode directories, which contains fastq files.
Open a fastq file using the command:

``` shell
vim <fastq_file>
```

`<fastq_file>` = the path to the fastq file you want to open

Can you identify the different lines that make up one read?

To exit vim: type `:q` and enter

------------------------------------------------------------------------

## 1.6 Gather the data

The data in it’s current format is difficult to work with- multiple
files, each with up to 4000 reads, for each barcode.  
The first command we will use is called **artic guppyplex**. This
command is used to do 2 things: 1) collect all of the fastq reads into a
single file; 2) apply a filter on read length.

### 1.6.1 Let’s explore this a little

First let’s apply the **artic guppyplex** command without any
restrictions on read length. Enter the following command, remembering to
edit parts with `< >`:

``` shell
artic guppyplex --prefix <output> --directory <path_to_passFastq>
```

`<output>` = provide a name for the output files `<path_to_passFastq>` =
provide the location (the filepath) of the input data (the pass fastq
folder)

------------------------------------------------------------------------

### 1.6.2 Task 5

Count the number of reads in a concatenated fastq file. We can do this
using the following command:

``` shell
grep -c "^@" <barcode_fastq_file>
```

Essentially this command asks the command line to count the number of
`@` characters that occur at the start of a line in the fastq file. This
corresponds to the header line in a fastq file, remember each read has a
header line.

------------------------------------------------------------------------

### 1.6.3 A note on read length

Since we expect our amplicons to be \~400bp, we could apply a length
filter to only allow reads of this length (e.g. –min-length 350
–max-length 700). However, from experience we know that our primers
misbehave a little and sometimes combine to produce longer amplicons. To
ensure we don’t eliminate any of this data we will only apply a minimum
read length filter (–min-length 300).

------------------------------------------------------------------------

### 1.6.4 Task 6

Repeat the previous command but add the minimum length flag. I’ll let
you try that yourself! Remember to give your output file a different
name from the previous step e.g. add `_filtered` to filename.

### 1.6.5 Task 7

Count then number of reads in the filtered fastq file. Is it different
from the previous fastq where we didn’t filter by read length? Why might
this be?

------------------------------------------------------------------------

## 1.7 Medaka pipeline

The next command we will use is called **artic minion**. This is a
wrapper script, which means that it encapsulates a number of different
processing steps that are hidden. This script will take a demultiplexed
fastq file and process it in the various ways we have discussed (filter,
align, consensus generation and refinement) in one go!

Output files will be generated along the way so that we can check
outputs from each of these stages but ultimately we end up with what we
want - a consensus sequence.

The command only processes one barcode at a time. There is of course a
way to automate this repetitive task using a script (a set of commands)
but we won’t get in to that here!

The command syntax is as follows:

``` shell
artic minion --no-frameshifts --medaka --medaka-model r941_min_fast_g303 --normalise 200 --threads 4 --scheme-directory /home/jovyan/shared-team/artic-rabv/primer-schemes/ --read-file <xx.fastq> rabv_ea/V1 <samplename>
```

`<xx.fastq>` = the input date, i.e. the fastq file associated with a
barcode. Remember to provide the full filepath `<samplename>` = the name
of the output files. This will prepend every file that is produced and
should ideally be the sample name e.g. barcode 70 corresponds to
sub6988. Remember to add the output file path before this!

------------------------------------------------------------------------

### 1.7.1 Task 8

Choose a barcode file and execute the command. With your rapidly growing
understanding of “commanding the command line” I will let you try to do
this yourselves. But, we will be here to help you!

------------------------------------------------------------------------

## 1.8 Looking at the data

Hopefully you have managed to execute the commands on a few barcoded
samples. Let’s now look at some of the output files.

## 1.9 BAM files

Several bam files are produced by the pipeline:

1.  xx.sorted.bam  
2.  trimmed.bam  
3.  primertrimmed.bam  
    The names give you a hint as to how they differ.

### 1.9.1 Full alignment data

The first version contains ALL of the reads aligned to the reference
genome, without any barcode trimming. So this is the best file to look
at in order to see the full amount of data produced, similar to RAMPART.

We will use the program **Tablet** to view bam files. Please open tablet
on your computer.

------------------------------------------------------------------------

### 1.9.2 Task 9

Open a sorted.bam file in Tablet and explore the data. Note the number
of reads and the starting position of the first 10 reads.

What do you notice about the coverage profile?

------------------------------------------------------------------------

### 1.9.3 Trimmed and normalised data

One of the jobs the **artic minion** script does it to trim the primer
sequences from the reads. We do these so that we don’t artificially bias
the reads at the primer positions - we put those sequences in there,
they are not from the virus in the sample!

------------------------------------------------------------------------

### 1.9.4 Task 10

Open a primertrimmed.bam file in Tablet and explore the data. Note the
number of reads. Now compare the starting position of the first 10 reads
in this version versus the non-trimmed data. Do you see a change?

What do you notice about the coverage profile?

------------------------------------------------------------------------

The **artic minion** command also downsamples the reads. As you will
have noted in the unprocessed alignment data there is variation in
coverage across the genome. There is also a LOT of data! By downsizing
the data to a read depth of 200 we can speed up the downstream
processing (we know that 200 depth gives us plenty of confidence to
determine SNP calls) and ensure we don’t overrepresent certain areas of
the genome.

### 1.9.5 Summary statistics

Let’s get a few summary stats to help us assess the data. We will use a
programme called weeSAM.

``` shell
/home/jovyan/shared-team/weeSAM/weeSAM --bam <input_bam> --out <output>
```

`<input_bam>` = a sorted.bam file `<input_bam>` = name (and location) of
output

For your reference here is some more detail about those bam files:

1.  xx.sorted.bam - - BAM file containing all the reads aligned to the
    reference sequence (there is no amplicon primer trimming in this
    file)  
2.  trimmed.bam - BAM file containing normalised (downsampled) reads
    with amplicon primers left on - this is the file used for variant
    calling  
3.  primertrimmed.bam - BAM file containing normalised (downsampled)
    reads with amplicon primers trimmed off barcode06.pass.vcf.gz -
    detected variants that PASSed the filters in VCF format (gzipped)
