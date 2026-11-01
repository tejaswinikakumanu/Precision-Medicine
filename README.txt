
PRECISION MEDICINE MIDTERM
Oxford Nanopore Direct RNA Sequencing Analysis Using RNA004 Chemistry

======================================================================
1. PROJECT OVERVIEW
======================================================================

This analysis used Oxford Nanopore direct RNA sequencing (DRS) data
generated with RNA004 chemistry.

The workflow included:

1. Downloading and verifying the POD5 dataset
2. Basecalling raw nanopore signal using Dorado
3. Converting basecalled BAM to FASTQ
4. Assessing sequencing quality using NanoPlot
5. Aligning reads to a human reference transcriptome using minimap2
6. Preparing Dorado move-table information for Squigualiser
7. Converting POD5 raw signal to BLOW5 format
8. Aligning and realigning reads for the required transcript
9. Visualizing raw signal-to-reference alignment for
   ENST00000387347.2, positions 1360-1400

Main working directory:

/N/slate/tkakuman/midterm_rna004


======================================================================
2. DIRECTORY STRUCTURE
======================================================================

The following analysis directories were used:

data/
basecalling/
qc/
alignment/
squigualiser/
models/
tools_env/
submission/


======================================================================
3. SOFTWARE AND VERSIONS
======================================================================

Dorado
Version: 0.5.3+d9af343

samtools
Version: 1.17

minimap2
Version: 2.26-r1175

NanoPlot
Version: 1.48.0

Squigualiser
Version: 0.7.0

blue-crab
Version: 0.6.0

A Python virtual environment named tools_env was used for software
installed with pip.


======================================================================
4. INPUT DATASET
======================================================================

Dataset:
Oxford Nanopore RNA004 human direct RNA sequencing dataset

Zenodo record:
15853348

Input POD5 file:

data/rna004_hsapiens_r10k.pod5

Download command:

wget -O data/rna004_hsapiens_r10k.pod5 
"https://zenodo.org/records/15853348/files/rna004_hsapiens_r10k.pod5?download=1"

The downloaded file was verified using MD5.

MD5 checksum:

6aae84947c546ef4300c8cd3fceb2809


======================================================================
5. DORADO BASECALLING
======================================================================

Dorado was used to convert the raw nanopore electrical signal stored
in the POD5 file into nucleotide sequences.

Model used:

rna004_130bps_hac@v3.0.1

Model directory:

models/rna004_130bps_hac@v3.0.1

The RNA004 model was selected because the sequencing data were
generated using RNA004 chemistry.

The HAC model was selected because it provides a balance between
basecalling accuracy and computational cost.

Final basecalling command:

dorado basecaller 
models/rna004_130bps_hac@v3.0.1 
data/rna004_hsapiens_r10k.pod5 
--device cuda:0 
--emit-moves \
> basecalling/basecalls.bam

Important parameters:

--device cuda:0
Used the allocated NVIDIA GPU for basecalling.

--emit-moves
Stored Dorado move-table information in the BAM output. This was
required for the Squigualiser Option 2 workflow.

The final successful basecalling run was performed using an NVIDIA
H100 GPU.

Basecalling runtime:
Approximately 48 seconds.

CPU-based testing was substantially slower, so GPU acceleration was
used for the final analysis.

Basecalled read count was checked using:

samtools view -c basecalling/basecalls.bam

Output:

10077

Final number of basecalled reads:

10,077


======================================================================
6. CONVERSION OF BAM TO FASTQ
======================================================================

The unaligned Dorado BAM was converted to FASTQ for NanoPlot and
minimap2 analysis.

Command:

samtools fastq 
basecalling/basecalls.bam \
> basecalling/basecalls.fastq

samtools reported:

10,077 reads processed
0 singletons discarded

FASTQ output:

basecalling/basecalls.fastq


======================================================================
7. NANOPLOT QUALITY ASSESSMENT
======================================================================

NanoPlot was used to evaluate read length, sequencing quality, and
sequencing yield.

Command:

NanoPlot 
--fastq basecalling/basecalls.fastq 
-o qc/NanoPlot 
--title "RNA004 Direct RNA Sequencing"

Main output files:

qc/NanoPlot/NanoPlot-report.html
qc/NanoPlot/NanoStats.txt


NanoPlot results:

Number of reads:
10,077

Total bases:
10,703,297

Mean read length:
1,062.2 bp

Median read length:
787 bp

Read length N50:
1,479 bp

Read length standard deviation:
953.9 bp

Mean read quality:
Q13.0

Median read quality:
Q15.4

Reads above Q10:
9,423 reads (93.5%)

Reads above Q15:
6,054 reads (60.1%)

Reads above Q20:
31 reads (0.3%)


Interpretation:

Most reads were of usable nanopore sequencing quality because 93.5%
of reads had quality scores above Q10.

The median read quality was Q15.4.

The read-length distribution was broad and right-skewed, which is
expected for RNA sequencing because transcripts naturally vary in
length.

The read length N50 was 1,479 bp.

The total sequencing yield was approximately 10.7 Mb.

The longest reported read was 39,996 bp but had a low quality score
of Q1.8, so it was treated as a low-quality outlier.


======================================================================
8. HUMAN REFERENCE TRANSCRIPTOME
======================================================================

Reads were aligned to the human Ensembl GRCh38 cDNA transcriptome.

Reference file:

data/Homo_sapiens.GRCh38.cdna.all.fa.gz

Reference:

Ensembl Homo sapiens GRCh38 cDNA transcriptome
Release 116


======================================================================
9. TRANSCRIPTOME ALIGNMENT WITH MINIMAP2
======================================================================

minimap2 was used as the long-read aligner.

Command:

minimap2 -ax map-ont 
data/Homo_sapiens.GRCh38.cdna.all.fa.gz 
basecalling/basecalls.fastq \
> alignment/transcriptome.sam

Parameter:

-ax map-ont

The map-ont preset was used because the reads were generated using
Oxford Nanopore sequencing and were being aligned against transcript
sequences.

minimap2 reported approximately:

Real time:
37.998 seconds

Peak memory usage:
6.932 GB


======================================================================
10. TRANSCRIPTOME ALIGNMENT RATE
======================================================================

The SAM file can contain multiple alignment records for an individual
read because reads may match related transcript isoforms.

To count mapped primary reads while excluding unmapped, secondary,
and supplementary records, the following command was used:

samtools view -c -F 2308 alignment/transcriptome.sam

Output:

9258

Total basecalled reads:

10077

Alignment percentage:

9258 / 10077 x 100 = 91.87%

Final transcriptome alignment rate:

91.87%

Interpretation:

Most basecalled reads aligned successfully to the human reference
transcriptome.

The high alignment rate indicates that the majority of the reads
represent human transcript sequences contained in the reference.

Reads that did not align may reflect lower-quality sequencing reads,
incomplete RNA molecules, sequencing errors, transcripts not
represented in the reference, or limitations of the alignment.


======================================================================
11. REQUIRED TARGET TRANSCRIPT
======================================================================

The Squigualiser visualization required the following transcript:

ENST00000387347.2

Required positions:

1360-1400

The target transcript reference file used for the signal visualization
was:

data/ENST00000387347.fa

Transcript length:

1,559 bp

This target FASTA was used specifically for the required
signal-to-reference visualization.


======================================================================
12. TARGET TRANSCRIPT ALIGNMENT
======================================================================

Reads were aligned to the required target transcript.

Command:

minimap2 -ax splice -uf -k14 
data/ENST00000387347.fa 
basecalling/basecalls.fastq \
> alignment/ENST00000387347.sam

The SAM file was converted to BAM:

samtools view -bS 
alignment/ENST00000387347.sam \
> alignment/ENST00000387347.bam

The BAM file was sorted:

samtools sort 
alignment/ENST00000387347.bam 
-o alignment/ENST00000387347.sorted.bam

Reads mapped to the target transcript were counted with:

samtools view -c -F 4 alignment/ENST00000387347.sam

Output:

180

This target-specific count was used for the Squigualiser workflow and
was not used as the overall transcriptome alignment percentage.


======================================================================
13. SQUIGUALISER OPTION 2 - MOVE TABLE PREPARATION
======================================================================

Squigualiser Option 2 uses the basecaller move table generated during
Dorado basecalling.

Because Dorado was run using --emit-moves, the move information was
available in basecalling/basecalls.bam.

The move table was converted to PAF format using:

squigualiser reform 
--sig_move_offset 0 
--kmer_length 1 
--rna 
-c 
--bam basecalling/basecalls.bam 
-o alignment/reform_moves.paf

Parameters:

--sig_move_offset 0
Signal move offset used for reform.

--kmer_length 1
Used one base per reform step.

--rna
Specified RNA data.

-c
Enabled the relevant reform output mode required for downstream
processing.

Results:

Processed SAM records:
9860

Skipped split reads:
217

Detected stride:
5


======================================================================
14. FILTERING READS RETAINED BY REFORM
======================================================================

The reform step skipped 217 split reads.

Therefore, the target SAM file was filtered so that only read IDs
present in the reform PAF file were retained.

Command:

awk 'BEGIN{while((getline < "alignment/reform_moves.paf") > 0) ids[$1]=1} /^@/ || ($1 in ids)' 
alignment/ENST00000387347.sam \
> alignment/ENST00000387347.reform_only.sam


======================================================================
15. SQUIGUALISER REALIGNMENT
======================================================================

Squigualiser realign was used to connect the signal coordinates from
the move table with the reference alignment.

Command:

squigualiser realign 
--rna 
--bam alignment/ENST00000387347.reform_only.sam 
--paf alignment/reform_moves.paf 
-o alignment/ENST00000387347.realign.bam

Result:

Processed SAM records:
179

The realigned BAM was sorted:

samtools sort 
alignment/ENST00000387347.realign.bam 
-o alignment/ENST00000387347.realign.sorted.bam

The sorted BAM was indexed:

samtools index 
alignment/ENST00000387347.realign.sorted.bam


======================================================================
16. POD5 TO BLOW5 CONVERSION
======================================================================

Squigualiser uses raw nanopore signal in SLOW5/BLOW5 format.

The original POD5 file was converted to BLOW5 using blue-crab.

Command:

blue-crab p2s 
data/rna004_hsapiens_r10k.pod5 
-o data/rna004_hsapiens_r10k.blow5

Output file:

data/rna004_hsapiens_r10k.blow5


======================================================================
17. SQUIGUALISER SIGNAL-TO-REFERENCE VISUALIZATION
======================================================================

The required transcript region was visualized using Squigualiser.

Transcript:

ENST00000387347.2

Region:

1360-1400

Command:

squigualiser plot 
--rna 
--file data/ENST00000387347.fa 
--slow5 data/rna004_hsapiens_r10k.blow5 
--alignment alignment/ENST00000387347.realign.sorted.bam 
--output_dir squigualiser/report 
--region ENST00000387347.2:1360-1400 
--tag_name "ENST00000387347.2_1360-1400"

Squigualiser generated:

108 plots

A representative HTML plot was copied to:

submission/squigualiser/ENST00000387347.2_1360-1400.html


Interpretation:

The Squigualiser plot displays the raw nanopore ionic-current signal
aligned with the corresponding RNA sequence positions.

As RNA passes through the nanopore, different nucleotide k-mers
produce characteristic changes in ionic current.

Dorado interprets these electrical-current patterns to generate the
basecalled nucleotide sequence.

The Dorado move table records how signal samples correspond to
sequence positions.

Squigualiser combines the move-table information with the reference
alignment so that the raw nanopore signal can be visualized relative
to the basecalled and reference sequence.

The submitted plot represents the required region of
ENST00000387347.2 from positions 1360-1400.


======================================================================
18. PRECISION MEDICINE APPLICATION
======================================================================

One important application of long-read RNA sequencing in precision
medicine is the identification of patient-specific transcript
isoforms and abnormal alternative splicing.

Conventional short-read RNA sequencing generates short fragments,
which can make it difficult to determine which splice junctions and
alternative exons occur together in the same RNA molecule.

Long-read RNA sequencing can span much larger portions of individual
transcripts and can reveal complete or near-complete transcript
isoforms, alternative exon combinations, and disease-associated splice
patterns.

This information can be useful in precision medicine because
patient-specific transcript isoforms may help explain disease
mechanisms and may support biomarker discovery, diagnosis, prognosis,
or targeted therapy selection.


======================================================================
19. REQUIRED SUBMISSION FILES
======================================================================

Written report:

Midterm_Report.pdf

NanoPlot quality assessment HTML report:

qc/NanoPlot-report.html

Squigualiser signal-to-reference visualization:

squigualiser/ENST00000387347.2_1360-1400.html

Reproducibility file:

README.txt


======================================================================
20. LARGE FILES NOT INCLUDED IN GITHUB
======================================================================

The following large raw or intermediate files were not uploaded to
GitHub because of their size:

POD5
BLOW5
FASTQ
BAM
SAM
Full transcriptome FASTA

All commands, software versions, models, parameters, reference files,
and analysis steps required to reproduce the workflow are documented
in this README.txt.
