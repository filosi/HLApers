
HLApers
=======

Getting started
---------------

### Install required software

##### 1. HLApers

    git clone https://github.com/genevol-usp/HLApers.git

##### 2. R v3.4+

##### 3. In R, install the following packages

-   from CRAN:

        install.packages(c("devtools", "tidyverse", "stringdist"))

-   from Bioconductor:

<!-- -->

    if (!requireNamespace("BiocManager", quietly = TRUE))
        install.packages("BiocManager")

    BiocManager::install("Biostrings")

-   from GitHub:

        devtools::install_github("genevol-usp/hlaseqlib")

##### 4. For STAR-Salmon-based pipeline, install:

-   STAR v2.5.3a+

-   Salmon v0.8.2+

-   samtools 1.3+

-   seqtk

##### 5. For kallisto-based pipeline, install:

-   kallisto

### Download data:

##### 1. IMGT database

    git clone https://github.com/ANHIG/IMGTHLA.git

##### 2. Gencode:

-   transcripts fasta (e.g., Gencode v30 fasta)

-   corresponding annotations GTF (e.g., Gencode v30 GTF)

HLApers usage
-------------

### Getting help

HLApers is composed of the following modes:

``` bash
./hlapers --help
```

    Usage: hlapers [modes]

    prepare-ref          Prepare transcript fasta files.
    index                Create index for read alignment.
    bam2fq               Convert BAM to fastq.
    genotype             Infer HLA genotypes.
    quant                Quantify HLA expression.

### 1. Building a transcriptome supplemented with HLA sequences

The first step is to use `hlapers prepare-ref` to build an index composed of Gencode transcripts, where we replace the HLA transcripts with IMGT HLA allele sequences.

``` bash
./hlapers prepare-ref --help
```

    Usage: hlapers prepare-ref [options]

    -t | --transcripts   Fasta with Gencode transcript sequences.
    -a | --annotations   GTF from Gencode for the same Genome version.
    -i | --imgt          Path to IMGT directory.
    -o | --out           hladb directory.

Example:

    ./hlapers prepare-ref -t gencode.v25.transcripts.fa.gz -a gencode.v25.annotation.gtf.gz -i IMGTHLA -o hladb

### 2. Creating an index for read alignment

``` bash
./hlapers index --help
```

    Usage: hlapers index [options]

    -t | --transcripts   Fasta with Gencode transcript sequences.
    -p | --threads       Number of threads.
    -o | --out           Output directory.
    --kallisto           Create index for kallisto pipeline instead of STARsalmon.

Example:

    ./hlapers index -t hladb/gencode_MHC_HLAsupp.fa -p 4 -o index

### 3. HLA genotyping

Given a BAM file from a previous alignment to the genome, we first need to extract the reads mapped to the MHC region and those which are unmapped. For this, we can use the `bam2fq` utility.

``` bash
./hlapers bam2fq --help
```

    Usage: hlapers bam2fq [options]

    -m | --mhc-coords    Genomic coordinates of the MHC region in chrN:start-end format if MHC fastq is desired.
    -b | --bam           BAM file.
    -o | --outprefix     Output prefix name.

Example:

    ./hlapers bam2fq -b ERR188021.bam -m ./hladb/mhc_coords.txt -o ERR188021

Then we run the genotyping module.

``` bash
./hlapers genotype --help
```

    Usage: hlapers genotype [options]

    -i | --index         Index generated by 'hlapers index'.
    -t | --transcripts   Fasta with Gencode transcripts sequences used for 'hlapers index'.
    -1 | --fq1           Fastq for READ1.
    -2 | --fq2           Fastq for READ 2.
    -p | --threads       Number of threads.
    -o | --outprefix     Output prefix name.
    --kallisto           Use kallisto for genotyping.

Example:

    ./hlapers genotype -i index/STARMHC -t ./hladb/gencode_MHC_HLAsupp.fa -1 ERR188021_mhc_1.fq -2 ERR188021_mhc_2.fq -p 8 -o results/ERR188021

### 4. Quantify HLA expression

In order to quantify expression, we use the `quant` module. If the original fastq files are available, we can proceed directly to the quantification step. If only a BAM file of a previous alignment to the genome is available, we first need to convert the BAM to fastq using the `bam2fq` utility.

Example:

    ./hlapers bam2fq -b ERR188021.bam -o ERR188021

Proceed to the quantification step.

``` bash
./hlapers quant --help
```

    Usage: hlapers quant [options]

    -t | --transcripts   Reference transcripts directory.
    -f | --hlafasta      Fasta with the individual's HLA sequences.
    -1 | --fq1           Fastq for READ 1.
    -2 | --fq2           Fastq for READ 2.
    -p | --threads       Number of threads.
    -o | --out           Output prefix name.

Example:

    ./hlapers quant -t ./hladb -f ./results/ERR188482_index.fa -1 ERR188021_1.fq.gz -2 ERR188021_2.fq.gz -o ./results/ERR188021 -p 8
