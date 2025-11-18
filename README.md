## Data Wrangling

Reproduces the _escherichia coli_ variant calling [workflow](https://datacarpentry.github.io/wrangling-genomics) using snakemake and an Apptainer container. The workflow proceeds by (i) fetching both the sequenced genomes of _e. coli_ from successive generations and a reference genome; (ii) assessing read quality using FastQC and/or trimming and filtering; (iii) aligning reads to the reference genome; (iv) post-alignment cleanup; (v) variant calling.

## Installation

On linux, clone the repository
```bash
git clone https://github.com/ajipalar/ecoli-bioinformatics .
```

Install apptainer
```bash
# Detect architecture
ARCH=$(uname -m)
if [ "$ARCH" = "x86_64" ]; then
    DEB_ARCH="amd64"
else
    DEB_ARCH="arm64"
fi

wget https://github.com/apptainer/apptainer/releases/download/v1.3.4/apptainer_1.3.4_${DEB_ARCH}.deb
sudo dpkg -i apptainer_1.3.4_${DEB_ARCH}.deb
```

Pull docker image and convert to .sif
```bash
cd containers
apptainer pull docker://ghcr.io/ajipalar/genomics:latest
mv genomics_latest.sif ecoli-bioinformatics.sif
cd ..
```

Test the installation
```bash
snakemake --use-singularity --cores 4 fastqc_test_1
```

## Running the workflow
To run all steps for a single sample (e.g., SRR2584866) run
```bash
snakemake --use-singularity --cores 4  results/vcf/SRR2584866_final_variants.vcf
```


