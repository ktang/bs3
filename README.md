#BS-Seeker3 
BS-Seeker3 performs accurate and fast mapping of bisulfite-treated short reads. It incorporates a series of new features to achieve significantly faster speed and better accuracy compared to other available bisulfite reads aligners. It is 1.5 time faster than BSMAP and 10 times faster than Bismark and maps twice the reads than both aligners. BS-Seeker3 also offers additional analysis of bisulphite read data to further investigate and visualize the methylation pattern after alignment.
#Table of Contents
- [New Features](#New Features)
- [System Requirements](#System Requirements)
- [BS-Seeker3 Usage](#Running BS-Seeker3)
   - [Download BS-Seeker3](#Download BS-Seeker3)
   - [Index Buidling](#Index Buidling)
   - [Alignment](#Alignment)
   - [Methylation Rate Calculation](#Methylation Rate Calculation)
   - [Methylation Rate Statistics Display](#Methylation Rate Statistics Display)
- [Example Use Case](#Example Use Case)


#<a name="New Features"></a>New Features
* Implements Improved Indexing, Fast Alignment with SNAP, and Highly Optimized SNAP Output Post-Processing
    * Produces an ultra-fast bisulfite read maping pipeline
* Executes Local Alignment through the Unnoken Algorithm
    * Achieves high mappability and accuracy
* Plots Quality Control Graph, Meta-gene Plot, and Bisulfite Unconversion Rate Histogram
    * Allows better visualization of the methylation data

#<a name="System Requirements"></a>System Requirements
* Linux or Mac OS Environment
* Python2 (version 2.5.2 or above; it should be pre-installed in both Linux and Mac). Type 'Python' to see the installed version. Python2 could be downloaded from http://www.python.org/download/ )
* Python Modules 'Pysam' and 'Metplotlib'. To install the packages, use the following commands on an UNIX terminal:
<br /> ``` pip install pysam ``` <br /> ```   pip install metplolib ``` <br />

#<a name="Running BS-Seeker3"></a>Running BS-Seeker3
BS-Seeker3 is a 3 steps process: 1) Index-building, 2) Alignment of the bisulfite reads, and 3) Methylation Rate Calculation. Index-buidling only has to be done once, and the user should adjust some parameters based on the reference genome size (See below for details). The alignment step uses SNAP to map the bisulfite reads to the reference genome, and then further removes the non-unique and incorrectly converted mappings. The methylation rate calculation step takes in the output from the alignment step and calcualtes the methylation rate at the single-base resolution

###<a name="Download BS-Seeker3"></a>Download BS-Seeker3
Type the following commands in an Unix Terminal:
* To download the Mac verion:
<br />```git clone https://github.com/khuang28jhu/bs3/bs3-mac ```
* To download the Linux version:
<br />```git clone https://github.com/khuang28jhu/bs3/bs3-linux ```

###<a name="Index Building"></a>Index Buidling
Use the script **bs3-build.py** to build an index from a reference genome. <br / ><br / >

**Usage:**<br / >
```
$ python bs3-build.py -h 
Usage: bs3-build.py -h [options]

-f                   Path to the reference genome; the reference genome should be in fasta format

-s                   Seed size (default: 20), a SNAP option; SNAP is based on a hashtable data 
                     strucutre. It builds its index by breaking the reference genome into seqeunces
                     (seed) of a specific length. This option determines the length of each 
                     seqeunce (seed size), and SNAP can deal with seed sizes to 23. A seed size of
                     20 is recommended for bisulfite reads of 100 bp long; a longer size should be
                     used for raw reads of longer length. 
                     
-locationSize        (default: 4), a SNAP option specific to the Linux implementation; This options 
                     determines the byte size used to store the location of each seed along the 
                     reference genome. It ranges from 4 to 8 bytes. For larger genomes, a larger 
                     location size should be used; for example, to build an index based on the human 
                     genome, a location size of 5 bytes is recommended. 
```
###<a name="Alignment"></a>Alignment
Use the script **bs3-align.py** to map the raw bisulfite reads. <br / ><br / >
**Input:**<br / >
* BS reads file in fastq
```
@SRR019072.2842 HWI-EAS365_1060:4:1:51:313 length=87
TAATTAGATTTGTGTTATAGATTATTTGTAAAGAAAGTAATTATTAAAGGAAATGTTAGTTTTTATTTGATATATGATAAGAGAACG
+SRR019072.2842 HWI-EAS365_1060:4:1:51:313 length=87
BBBCC@)8ABA/<2>CB:=.:?BBABB1-:@74@B@?=@@ABB@B7@@5/98<;)<>56:?>:;A?A?A@>=AABB@A<3(@@=086
```
* BS reads file in fasta
```
>read1
TCCATTATACCGTAACCCAATACAAAAATTATTTAT
>read2
TCTGTAGACGGGTCGAATGGGGAGTTCATAGGGGGG
```
**Usage:**
```
$ python bs3-align.py -h 
Usage: bs3-align.py -h [options] 

For single end reads:

-i INFILE,           Input read file (FORMAT:  fasta, fastq). Ex: read.fa or read.fa.gz

For pair end reads:

-1 FILE,             Input read file, mate 1 (FORMAT: fasta, fastq)

-2 FILE,             Input read file, mate 2 (FORMAT: fasta, fastq)

Important General options:

-g GENOME,           Name of the reference genome (should be the same as "-f" in bs3-build.py ) [ex.
                     chr21_hg18.fa]

-m NO_MISMATCHES,    Set the number(>=1)/percentage([0, 1)) of mismatches in a read. Ex: 4 (allow 8 
                     mismatches) or 0.08 (allow 8% mismatches) [Default: 12]
                     
-l INT,              Split the input file into smaller files based on this number. Each smaller file 
                     is processed in paralell. The result is then merged. [Default: 12800000]
                     
-o OUTFILE           The name of output file 

Relevant Aligner Options:

--snap-h             MaxHits, (default: 250 on the Mac version, 300 on Linux) a SNAP option; There 
                     are often patterns that occur within multiple locations of a genome. Processing
                     the hashtable index hits with seeds that match these patterns is time-consuming. 
                     This option sets a threshold on the number of locations that a seed can match 
                     to. Seeds matching to locations more than this number are considered never 
                     existed during the alignment step.
                     
Methylation Rate Statistics Display Option:

--qcf=QC_F           Supply the length of the raw bisulfite reads to plot a quality control plot. A 
                     quality control plot tabulates the average rate of mismatches of each position 
                     on a raw read.
```
**Output:**<br / >
* Alignment Summary in .stat file
```
Number of reads in total: 10000000
Number of unique-hits reads (before post-filtering): 9359751.0
Number of reads mapped after post-filtering 3343919.0
Methylated C in mapped reads
 mCG  0.685%
 mCHG  0.031%
 mCHH  0.030%
```
* List of Aligned Reads in SAM Format ([SAM Fields Description](https://samtools.github.io/hts-specs/SAMv1.pdf))

```
SRR2058107.412129	0	10_w_c	42386003	1	90M	*	0	0	TGGATTGGAAGGTAATTATTATTGAATGGAATTGAATGGAATTATTGAATGGATTTGAATGGAATAATTATTGAATGGAATTGAATGGAA	IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII	PG:Z:SNAP	NM:i:3	RG:Z:FASTQ	PL:Z:Illumina	PU:Z:pu	LB:Z:lb	SM:Z:sm
```

###<a name="Methylation Rate Calculation"></a>Methylation Rate Calculation
Use the script **bs3-align.py** to map the raw bisulfite reads. <br / ><br / >
**Input:**
* SAM file from the previous step

**Usage:**
```
$ python bs3-call_methylation.py -h 
Usage: bs3-call_methylation.py -h [options]

Options:

-i INFILE,          Input alinged reads file in SAM format; output from bs3-align.py

-d DBPATH,          Path to the reference genome library (generated during index-buidling) (optional)

-o OUTFILE,         The output prefix to create the CGmap, ATCGmap and wiggle files

--sorted,           Specify when the input bam file is already sorted, the sorting step will be 
                    skipped [Default: False]
```
**Output:**

- wig file

    Sample:

        variableStep chrom=chr1
        3000419	0.000000
        3000423	-0.2
        3000440	0.000000
        3000588	0.5
        3000593	-0.000000


        Format descriptions:
        WIG file format. Negative value for 2nd column indicate a Cytosine on minus strand.


- CGmap file

    Sample:

        chr1	G	3000851	CHH	CC	0.1	1	10
        chr1	C	3001624	CHG	CA	0.0	0	9
        chr1	C	3001631	CG	CG	1.0	5	5

    Format descriptions:

        (1) chromosome
        (2) nucleotide on Watson (+) strand
        (3) position
        (4) context (CG/CHG/CHH)
        (5) dinucleotide-context (CA/CC/CG/CT)
        (6) methylation-level = #_of_C / (#_of_C + #_of_T).
        (7) #_of_C (methylated C, the count of reads showing C here)
        (8) = #_of_C + #_of_T (all Cytosines, the count of reads showing C or T here)


- ATCGmap file

    Sample:

        chr1	T	3009410	--	--	0	10	0	0	0	0	0	0	0	0	na
        chr1	C	3009411	CHH	CC	0	10	0	0	0	0	0	0	0	0	0.0
        chr1	C	3009412	CHG	CC	0	10	0	0	0	0	0	0	0	0	0.0
        chr1	C	3009413	CG	CG	0	10	50	0	0	0	0	0	0	0	0.83


    Format descriptions:

        (1) chromosome
        (2) nucleotide on Watson (+) strand
        (3) position
        (4) context (CG/CHG/CHH)
        (5) dinucleotide-context (CA/CC/CG/CT)

        (6) - (10) plus strand
        (6) # of reads from Watson strand mapped here, support A on Watson strand
        (7) # of reads from Watson strand mapped here, support T on Watson strand
        (8) # of reads from Watson strand mapped here, support C on Watson strand
        (9) # of reads from Watson strand mapped here, support G on Watson strand
        (10) # of reads from Watson strand mapped here, support N

        (11) - (15) minus strand
        (11) # of reads from Crick strand mapped here, support A on Watson strand and T on Crick strand
        (12) # of reads from Crick strand mapped here, support T on Watson strand and A on Crick strand
        (13) # of reads from Crick strand mapped here, support C on Watson strand and G on Crick strand
        (14) # of reads from Crick strand mapped here, support G on Watson strand and C on Crick strand
        (15) # of reads from Crick strand mapped here, support N

        (16) methylation_level = #C/(#C+#T) = C8/(C7+C8) for Watson strand, =C14/(C11+C14) for Crick strand;
        "nan" means none reads support C/T at this position.



###<a name="Methylation Rate Statistics Display"></a>Methylation Rate Statistics Display
Use the script **bs3-methyl_display.py** to plot the meta-gene file or the quality control plot.<br / ><br / > 

**Input:**
* 'CGmap' file from the 'Methylation Rate Calculation' step
* For a Metagene Plot based on a paritcular genomic structure (gene or transposon), the gene annotation file (in gff3); [Description of the fields in a gff3 file](http://gmod.org/wiki/GFF3#GFF3_Format)
* For a QC Plot, the '.qc' file from the 'Alignment' step; 

**Usage:**
```
$ python bs3-methyl_display.py -h 
Usage: bs3-methyl_display.py -h [options] 

-m MET             Supply the single-base resolution methylation level file generated during the 
                   methylation rate calculation (in CG format)
                   
-a ANNOTATION      Suppply the gene annotation file to build the meta-plot (in gff3 format)

-r GENOME_REGION   Select the genomeic region to be plotted for the meta-plot, transposon or gene. 
                   Select each with the option ```-r gene``` or ```-r transposon```; (default: gene)
                   
-q QC_F            Plot Quality Control Graph, supply the .qc file generated during the alignment 
                   step

--meta=META        Plot metagene plot
```
**Output**
* Example Meta-gene Plot
![meta] (https://github.com/khuang28jhu/bs3/blob/master/metaplot.png)
* Example Quality Control Plot
![qclot] (https://github.com/khuang28jhu/bs3/blob/master/QC_Plot.png)


Use the script **bs3-unconversion.py** to calculate the unconversion rate of the bisulfite reads if your data contains control reads from the lambda phage library. The lambda phage DNA is believed to be free of DNA methylation, so all cytosine of the genome should be converted to uracil in an ideal situation. Any unconverted cytosines of the mapped reads thus reveal the unconversionr rate
<br / ><br / >**Usage:**
```
$ python bs3-unconversion.py -h
Usage: bs3-unconversion.py -h [options]

-f INPUT          The path to the raw bisulfite read file.

-g GENOME         The path to the genome file.
```
**Output**
* Example Unconversion Rate Plot
![unconversion] (https://github.com/khuang28jhu/bs3/blob/master/Unconversion_Rate.png)

# <a name="Example Use Case"></a>Example Use Case
done by tmr







