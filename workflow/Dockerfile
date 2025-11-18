# Dockerfile for bioinformatics tools used in E. coli evolution pipeline
# Snakemake runs outside Docker, this container provides the tools

FROM ubuntu:22.04

# Explicitly set shell
SHELL ["/bin/bash", "-c"]

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies and standard utilities
RUN apt-get update && apt-get install -y \
    bash \
    curl \
    wget \
    tar \
    gzip \
    bzip2 \
    ca-certificates \
    vim \
    && rm -rf /var/lib/apt/lists/*

# Install micromamba
ENV MAMBA_ROOT_PREFIX=/opt/conda

# Detect architecture and download appropriate micromamba binary
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
        MAMBA_ARCH="linux-64"; \
    elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then \
        MAMBA_ARCH="linux-aarch64"; \
    else \
        echo "Unsupported architecture: $ARCH" && exit 1; \
    fi && \
    cd && \
    curl -Ls "https://micro.mamba.pm/api/micromamba/${MAMBA_ARCH}/latest" | tar -xvj bin/micromamba && \
    mv bin/micromamba /usr/local/bin/ && \
    chmod +x /usr/local/bin/micromamba && \
    rm -rf bin



# Set up PATH and environment variables
ENV PATH=$MAMBA_ROOT_PREFIX/bin:$PATH
ENV CONDA_DEFAULT_ENV=base
ENV CONDA_PREFIX=$MAMBA_ROOT_PREFIX

# Initialize micromamba and create base environment with channels specified
# Step 1: Create the base environment and install tools with channels
RUN /usr/local/bin/micromamba create -y -p $MAMBA_ROOT_PREFIX \
    -c conda-forge -c bioconda \
    fastqc trimmomatic openjdk bwa samtools bcftools

# Configure micromamba channels for future use
# Create .condarc file manually to avoid config command issues
# Micromamba checks multiple locations, so we create it in both places
RUN mkdir -p $MAMBA_ROOT_PREFIX /root && \
    printf "channels:\n  - conda-forge\n  - bioconda\nchannel_priority: strict\n" > $MAMBA_ROOT_PREFIX/.condarc && \
    printf "channels:\n  - conda-forge\n  - bioconda\nchannel_priority: strict\n" > /root/.condarc

# Create initialization script for all shell types
RUN echo 'eval "$(/usr/local/bin/micromamba shell hook -s bash)"' > /etc/profile.d/micromamba.sh && \
    chmod +x /etc/profile.d/micromamba.sh && \
    echo 'eval "$(/usr/local/bin/micromamba shell hook -s bash)"' >> /root/.bashrc && \
    echo 'micromamba activate base' >> /root/.bashrc

# Clean up micromamba cache to reduce image size
RUN micromamba clean -afy

# Verify installations (using full path to ensure environment is active)
RUN /usr/local/bin/micromamba run -p $MAMBA_ROOT_PREFIX fastqc --version && \
    /usr/local/bin/micromamba run -p $MAMBA_ROOT_PREFIX trimmomatic -version

# Set default command
# Note: Tools are available via PATH, and micromamba is initialized in .bashrc for interactive shells
CMD ["/bin/bash"]

