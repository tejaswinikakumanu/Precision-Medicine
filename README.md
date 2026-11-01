# Precision-Medicine
Oxford Nanopore RNA004 direct RNA sequencing analysis for Precision Medicine Midterm
# Midterm

## Oxford Nanopore Direct RNA Sequencing Analysis — RNA004

This repository contains the analysis workflow, results, and required output files for the Precision Medicine Midterm assignment using Oxford Nanopore direct RNA sequencing (DRS) data generated with RNA004 chemistry.

The workflow included:

1. Basecalling raw POD5 signals using Dorado
2. Quality assessment using NanoPlot
3. Transcriptome-wide alignment using minimap2
4. Raw nanopore signal-to-reference visualization using Squigualiser Option 2, based on the Dorado move table

---

## 1. Working Directory

The analysis was performed in:

```text
/N/slate/tkakuman/midterm_rna004
```

Main analysis directories:

```text
data/
basecalling/
qc/
alignment/
squigualiser/
models/
tools_env/
submission/
```

---

## 2. Software and Versions

The following software was used:

- Dorado: `0.5.3+d9af343`
- samtools: `1.17`
- minimap2: `2.26-r1175`
- NanoPlot: `1.48.0`
- Squigualiser: `0.7.0`
- blue-crab: `0.6.0`

A Python virtual environment named `tools_env` was used for NanoPlot, Squigualiser, and blue-crab.

---

## 3. Input Dataset

The input dataset was an Oxford Nanopore RNA004 human direct RNA sequencing dataset containing approximately 10,000 reads.

Zenodo record:

```text
15853348
```

Input POD5 file:

```text
data/rna004_hsapiens_r10k.pod5
```

Download command:

```bash
wget -O data/rna004_hsapiens_r10k.pod5 \
"https://zenodo.org/records/15853348/files/rna004_hsapiens_r10k.pod5?download=1"
```

The downloaded file was verified using MD5.

```text
MD5: 6aae84947c546ef4300c8cd3fceb2809
```

---

# 4. Basecalling with Dorado

The raw RNA004 POD5 signals were basecalled using Dorado.

## Model

The selected model was:

```text
rna004_130bps_hac@v3.0.1
```

The HAC model was chosen because it provides a balance between basecalling accuracy and computational cost. The RNA004-specific model was appropriate because the dataset was generated using RNA004 chemistry.

The model was stored in:

```text
models/rna004_130bps_hac@v3.0.1
```

## Dorado command

The final basecalling run was performed using an NVIDIA H100 GPU.

```bash
dorado basecaller \
models/rna004_130bps_hac@v3.0.1 \
data/rna004_hsapiens_r10k.pod5 \
--device cuda:0 \
--emit-moves \
> basecalling/basecalls.bam
```

The option:

```text
--device cuda:0
```

instructed Dorado to use the allocated GPU.

The option:

```text
--emit-moves
```

was included because Squigualiser Option 2 requires the Dorado move-table information to connect the raw nanopore signal with the basecalled sequence.

## Basecalling result

The number of reads was checked with:

```bash
samtools view -c basecalling/basecalls.bam
```

Output:

```text
10077
```

Therefore, the final basecalling output contained:

```text
10,077 reads
```

The successful GPU basecalling job completed in approximately:

```text
48 seconds
```

CPU testing was substantially slower, so GPU acceleration was used for the final workflow.

---

# 5. BAM to FASTQ Conversion

NanoPlot did not successfully process the unaligned Dorado BAM directly, so the basecalled reads were converted to FASTQ.

Command:

```bash
samtools fastq basecalling/basecalls.bam > basecalling/basecalls.fastq
```

samtools reported:

```text
10,077 reads processed
0 singletons discarded
```

The resulting FASTQ file was:

```text
basecalling/basecalls.fastq
```

---

# 6. Quality Assessment with NanoPlot

NanoPlot was used to evaluate read length, sequencing quality, and yield.

Command:

```bash
NanoPlot \
--fastq basecalling/basecalls.fastq \
-o qc/NanoPlot \
--title "RNA004 Direct RNA Sequencing"
```

Main outputs:

```text
qc/NanoPlot/NanoPlot-report.html
qc/NanoPlot/NanoStats.txt
```

## NanoPlot Results

| Metric | Result |
|---|---:|
| Number of reads | 10,077 |
| Total bases | 10,703,297 |
| Mean read length | 1,062.2 bp |
| Median read length | 787 bp |
| Read length N50 | 1,479 bp |
| Read length standard deviation | 953.9 bp |
| Mean read quality | Q13.0 |
| Median read quality | Q15.4 |

Quality thresholds:

| Quality Threshold | Reads |
|---|---:|
| > Q10 | 9,423 (93.5%) |
| > Q15 | 6,054 (60.1%) |
| > Q20 | 31 (0.3%) |

## Interpretation

The dataset contained predominantly usable nanopore reads, with **93.5% of reads above Q10**.

The median read quality was **Q15.4**, indicating that a substantial fraction of the dataset was of relatively good quality for nanopore direct RNA sequencing.

The read-length distribution was broad and right-skewed, which is expected because RNA transcripts vary considerably in length.

The total sequencing yield was approximately:

```text
10.7 Mb
```

The longest observed read was approximately 39,996 bp but had a low quality score of Q1.8, so it was interpreted as a low-quality outlier rather than strong evidence of a genuine transcript of that length.

---

# 7. Transcriptome-Wide Alignment

The basecalled reads were aligned to the human Ensembl GRCh38 cDNA reference transcriptome.

Reference file:

```text
data/Homo_sapiens.GRCh38.cdna.all.fa.gz
```

Reference:

```text
Ensembl Homo sapiens GRCh38 cDNA transcriptome, release 116
```

minimap2 was used as the long-read aligner.

The `map-ont` preset was selected because the query sequences are Oxford Nanopore long reads and the reference contains transcript sequences rather than genomic sequences containing introns.

Command:

```bash
minimap2 -ax map-ont \
data/Homo_sapiens.GRCh38.cdna.all.fa.gz \
basecalling/basecalls.fastq \
> alignment/transcriptome.sam
```

minimap2 reported approximately:

```text
Real time: 37.998 seconds
Peak RSS: 6.932 GB
```

The SAM file contains multiple possible transcript alignments for some reads because many human transcripts have related isoforms. Therefore, mapped primary reads were counted while excluding unmapped, secondary, and supplementary alignments.

Command:

```bash
samtools view -c -F 2308 alignment/transcriptome.sam
```

Output:

```text
9258
```

Total basecalled reads:

```text
10077
```

Alignment percentage:

```text
9258 / 10077 × 100 = 91.87%
```

Therefore:

```text
9,258 of 10,077 reads aligned
Alignment rate = 91.87%
```

## Interpretation

The **91.87% transcriptome alignment rate** indicates that most of the basecalled reads correspond to human transcript sequences represented in the reference transcriptome.

Reads that did not align may reflect:

- lower-quality reads
- incomplete RNA molecules
- sequences absent from the reference
- sequencing errors
- alignment limitations

---

# 8. Target Transcript for Squigualiser

The required Squigualiser visualization was:

```text
Transcript: ENST00000387347.2
Positions: 1360-1400
```

The exact target transcript sequence was obtained separately because the required versioned transcript was not present in the downloaded current GRCh38 cDNA transcriptome.

Target FASTA:

```text
data/ENST00000387347.fa
```

The target transcript sequence length was:

```text
1,559 bp
```

---

# 9. Target Transcript Alignment

The reads were aligned to the required transcript for downstream Squigualiser visualization.

Command:

```bash
minimap2 -ax splice -uf -k14 \
data/ENST00000387347.fa \
basecalling/basecalls.fastq \
> alignment/ENST00000387347.sam
```

The SAM file was converted to BAM:

```bash
samtools view -bS \
alignment/ENST00000387347.sam \
> alignment/ENST00000387347.bam
```

The BAM file was sorted:

```bash
samtools sort \
alignment/ENST00000387347.bam \
-o alignment/ENST00000387347.sorted.bam
```

Mapped alignments to the target transcript:

```bash
samtools view -c -F 4 alignment/ENST00000387347.sam
```

Result:

```text
180
```

This target-specific value was used only for the required signal visualization and was not used as the transcriptome-wide alignment percentage.

---

# 10. Squigualiser Option 2 — Basecaller Move Table

Squigualiser Option 2 was used because Dorado basecalling was performed with:

```text
--emit-moves
```

This preserved the move-table information necessary to connect raw nanopore signal samples with the basecalled sequence.

## Reform move table

Command:

```bash
squigualiser reform \
--sig_move_offset 0 \
--kmer_length 1 \
--rna \
-c \
--bam basecalling/basecalls.bam \
-o alignment/reform_moves.paf
```

Squigualiser reported:

```text
Processed SAM records: 9860
Skipped split reads: 217
Detected stride: 5
```

---

# 11. Filter Reads Retained by Squigualiser Reform

The `reform` step skipped 217 split reads.

Therefore, the target SAM file was filtered so that only read IDs present in the reform PAF file were retained.

Command:

```bash
awk 'BEGIN{while((getline < "alignment/reform_moves.paf") > 0) ids[$1]=1} /^@/ || ($1 in ids)' \
alignment/ENST00000387347.sam \
> alignment/ENST00000387347.reform_only.sam
```

---

# 12. Squigualiser Realignment

The signal coordinates were realigned to the reference alignment.

Command:

```bash
squigualiser realign \
--rna \
--bam alignment/ENST00000387347.reform_only.sam \
--paf alignment/reform_moves.paf \
-o alignment/ENST00000387347.realign.bam
```

Result:

```text
Processed SAM records: 179
```

The BAM was then sorted:

```bash
samtools sort \
alignment/ENST00000387347.realign.bam \
-o alignment/ENST00000387347.realign.sorted.bam
```

The sorted BAM was indexed:

```bash
samtools index alignment/ENST00000387347.realign.sorted.bam
```

---

# 13. POD5 to BLOW5 Conversion

Squigualiser requires the raw signal in SLOW5/BLOW5 format.

The POD5 file was converted using blue-crab.

Command:

```bash
blue-crab p2s \
data/rna004_hsapiens_r10k.pod5 \
-o data/rna004_hsapiens_r10k.blow5
```

Output:

```text
data/rna004_hsapiens_r10k.blow5
```

---

# 14. Signal-to-Reference Visualization

Squigualiser was used to visualize raw nanopore signal aligned to:

```text
ENST00000387347.2:1360-1400
```

Command:

```bash
squigualiser plot \
--rna \
--file data/ENST00000387347.fa \
--slow5 data/rna004_hsapiens_r10k.blow5 \
--alignment alignment/ENST00000387347.realign.sorted.bam \
--output_dir squigualiser/report \
--region ENST00000387347.2:1360-1400 \
--tag_name "ENST00000387347.2_1360-1400"
```

Squigualiser generated:

```text
108 plots
```

One representative HTML plot was copied to the final submission directory as:

```text
submission/squigualiser/ENST00000387347.2_1360-1400.html
```

## Interpretation of the Squigualiser Plot

The Squigualiser plot displays the raw nanopore ionic-current signal aligned with corresponding positions in the reference/basecalled sequence.

As an RNA molecule passes through the nanopore, different nucleotide k-mers cause characteristic changes in ionic current.

Dorado interprets these current patterns to generate the basecalled sequence.

The move table generated during basecalling connects sections of the raw electrical signal to changes in sequence position. Squigualiser uses this information together with the reference alignment to visualize how the raw nanopore signal corresponds to the basecalled RNA sequence.

---

# 15. Precision Medicine Application

One important precision-medicine application of long-read RNA sequencing is the identification of **patient-specific transcript isoforms and abnormal alternative splicing**.

Conventional short-read RNA sequencing generates many small fragments that must be computationally reconstructed into transcripts. This can make it difficult to determine which exons and splice junctions occur together in the same RNA molecule.

Long-read RNA sequencing can span much larger portions of individual transcripts and can therefore directly reveal:

- complete or near-complete transcript isoforms
- alternative exon combinations
- alternative splice junctions
- disease-associated transcript structures

This information can be useful in precision medicine because patient-specific transcript isoforms may contribute to disease mechanisms and may provide information relevant to diagnosis, prognosis, biomarker discovery, or targeted therapy selection.

---

# 16. Final Submission Files

The final submission repository contains:

```text
Precision-Medicine-Midterm/
├── README.md
├── README.txt
├── report/
│   └── Midterm_Report.pdf
├── qc/
│   ├── NanoPlot-report.html
│   └── NanoStats.txt
├── squigualiser/
│   └── ENST00000387347.2_1360-1400.html
└── commands/
    └── analysis_commands.sh
```

The following large raw or intermediate files are not included in the GitHub repository because of their size:

```text
POD5
BLOW5
FASTQ
BAM
SAM
full transcriptome FASTA
```

The commands, models, parameters, software versions, reference information, and results are documented above so that the analysis can be reproduced.
