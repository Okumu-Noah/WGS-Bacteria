# Analysis of llumina WGS data
## Illumina Pipeline
1. Data acqusition in fastq format and unzipping

2. Quality control (QC) of raw data: This step involves checking the quality of the raw sequencing data using quality control software, such as FastQC/MultiQC, to assess the overall quality, sequence length distribution, base quality, and sequence duplication.

3. Read trimming and filtering: In this step, low-quality bases and reads are removed to improve the overall quality of the sequencing data. Popular tools for trimming and filtering include Fastp, Trimmomatic, Sickle and BBDuk. Here we will use Fastp.

4. Genome assembly: This step involves the construction of contigs from the filtered reads using assembly software, such as SPAdes, ABySS, RadTag, BWA, Bowtie or Velvet. This step generates the draft genome assembly of the bacterium. Here we will use SPAdes. de novo assembly

5. Quality assessment of genome assembly: This step involves assessing the quality of the genome assembly using tools like QUAST, which calculates assembly metrics such as contig N50, genome size, and number of contigs. Tools applied QUAST, SAM & Quali Maps.

6. Polishing refers to the process of refining and improving the quality of a genome sequence assembly. It involves correcting errors in the initial assembly, closing gaps between contigs, and improving the accuracy of base calls. Tools applied Pilon, Quiver, and Racon.

7. Pathogen identification: This step involves identifying the bacterial species or strains present in the sequenced sample using tools such as Kraken, MetaPhlAn, or Centrifuge.

8. Multilocus sequence typing (MLST) is a typing technique of multiple loci, using DNA sequences of internal fragments of multiple housekeeping genes to characterize isolates of microbial species/to classify and identify different strains of bacteria.

9. SNPs/InDels Identification involves variants calling ie identifying single nucleotide polymorphisms (SNPs) and small insertions or deletions (indels) in the genome assembly. Tools like FreeBayes, GATK, BCF, rtg can be used for this purpose. Viewing can be done by IGV, UCSC etc

10. Genome annotation: In this step, the genome assembly is annotated with gene prediction software such as Prokka or RAST, which predict genes and annotate them with functional information ie checks non-coding RNAs, CRISPRs, pathogenic and susceptibility genes/virulence. Blasting is also done here.

11. Comparative genomics: Comparative genomics involves comparing the newly sequenced genome with existing genomes to identify unique features, virulence factors, and antibiotic resistance genes. Tools like Roary or Prokka can be used for this purpose. i. Evolutionary analysis use phylogenetic tree/mapping can be constructed using the identified SNPs to analyze the evolutionary relationship between different bacterial strains. ANI-Dendogram which uses dREP gives genetic relatedness between two bacterial genomes. It calculates the average percentage of nucleotide sequence identity between the two genomes, taking into account both conserved and divergent regions. ii. Pangenome analysis use Roary. iii. Genome visualization in ring structures use BRIG. iv. AMR genes use RGI, CARD, PCA, ResFinder v. Functional annotation COG, GO, KEGG vi. DIvergence Time estimation

12. Metadata - linking genotypic data with phenotypic traits, Plasmids, Mutations etc

# Steps
1. Log into hpc via ssh & interactive computing
```
interactive -w compute05 -c 3
```
2. Creating directory
```
cd /var/scratch/
mkdir -p $USER/bacteria-wgs/crpa
cd $USER/bacteria-wgs/crpa
```
The mkdir -p command is used to create a directory and its parent directories (if they do not exist) in a single command.

The -p option allows the mkdir command to create the parent directories if they do not exist.

3. Data retieval: Use scp username@remote:/path/to/data /path/to/local/directory
```
scp -r nokumu@hpc.ilri.cgiar.org:/path/to/data  .
```

The -r option is used to copy the directory recursively. The . at the end of the command specifies the current directory as the destination. space fullstop " ." specifies your current folder.

4. Load modules, cd database
```
module load fastqc/0.11.9
module load fastp/0.22.0
module load krona/2.8.1
module load centrifuge/1.0.4
module load kraken/2.1.2
module load spades/3.15
module load quast/5.0.2
module load samtools/1.15.1
module load BUSCO/5.2.2
module load bowtie2/2.5.0
module load bedtools/2.29.0
module load bamtools/2.5.1
module load ivar/1.3.1
module load snpeff/4.1g
module load bcftools/1.13
module load nextclade/2.11.0
module load R/4.2
module load prokka/1.11
module load blast/2.12.0+
```
5. Run fastqc on 

i. Two samples
```
fastqc \
        -t 4 \
        --extract \
        -o ./results/fastqc \
        -f fastq ./raw_data/merged_fastq_pass/barcode02.all.fastq \
                 ./raw_data/merged_fastq_pass/barcode48.all.fastq
                 ```

ii. Create the loop for all samples

```
#!/usr/bin/bash -l
#SBATCH -p batch
#SBATCH -J fastqc
#SBATCH -n 4

# Set the input and output directories
INPUT_DIR=./raw_data/merged_fastq_pass/
OUTPUT_DIR=./results/fastqc/

# Make directory to store the results
mkdir -p "$OUTPUT_DIR"

# Load FastQC module
module load fastqc/0.11.9

# Run FastQC on all ONT sequencing files in the directory "raw_data/merged_fastq_pass/"
for file in $INPUT_DIR/*.fastq; do
    fastqc -t 4 --extract -o $OUTPUT_DIR -f fastq $file
done
```
