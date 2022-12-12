# Bias factorized, base-resolution deep learning models of chromatin accessibility reveal cis-regulatory sequence syntax, transcription factor footprints and regulatory variants

- This repo contains code for the paper `Bias factorized, base-resolution deep learning models of chromatin accessibility reveal cis-regulatory sequence syntax, transcription factor footprints and regulatory variants` (techincal report coming soon) by  Anusri Pampari*, Anna Shcherbina*, Anshul Kundaje. (*authors contributed equally)  
- Please contact [Anusri Pampari] (\<first-name\>@stanford.edu) for suggestions and comments. 
- Here is a link to the [slides](https://docs.google.com/presentation/d/1Ow6K8TYN40u7T3ODdo-JRCLuv5fUUacA/edit?usp=sharing&ouid=104820480456877027097&rtpof=true&sd=true) and a comprehensive [tutorial](https://github.com/kundajelab/chrombpnet/wiki). Please see the [FAQ](https://github.com/kundajelab/chrombpnet/wiki/FAQ) and file a github [issue](https://github.com/kundajelab/chrombpnet/issues) if you have questions.

Chromatin profiles (DNASE-seq and ATAC-seq) exhibit multi-resolution shapes and spans regulated by co-operative binding of transcription factors (TFs). This complexity is further difficult to mine because of confounding bias from enzymes (DNASE-I/Tn5) used in these assays. Existing methods do not account for this complexity at base-resolution and do not account for enzyme bias correctly, thus missing the high-resolution architecture of these profile. Here we introduce ChromBPNet to address both these aspects.

ChromBPNet (shown in the image as `Bias-Factorized ChromBPNet`) is a fully convolutional neural network that uses dilated convolutions with residual connections to enable large receptive fields with efficient parameterization. It also performs automatic assay bias correction in two steps, first by learning simple model on chromatin background that captures the enzyme effect (called `Frozen Bias Model` in the image). Then we use this model to regress out the effect of the enzyme from the ATAC-seq/DNASE-seq profiles. This two step process ensures that the sequence component of the ChromBPNet model (called `TF Model`) does not learn enzymatic bias. 

<p align="center">
<img src="images/chrombpnet_arch.png" alt="ChromBPNet" align="center" style="width: 400px;"/>
</p>

## Table of contents

- [Installation](#installation)
- [QuickStart](#quickstart)
- [How-to-cite](#how-to-cite)

## Installation

This section will discuss the packages needed to train a ChromBPNet model. Firstly, it is recommended that you use a GPU for model training and have the necessary NVIDIA drivers and CUDA already installed. You can verify that your machine is set up to use GPU's properly by executing the `nvidia-smi` command and ensuring that the command returns information about your system GPU(s) (rather than an error). Secondly there are two ways to ensure you have the necessary packages to train ChromBPNet  models which we detail below,

### 1. Installation setup through Docker

Download and install the latest version of Docker for your platform. Here is the link for the installers -<a href="https://docs.docker.com/get-docker/">Docker Installers</a>.  Run the docker run command below to open a environment with all the packages installed and do `cd chrombpnet` to start running the tutorial.

> **Note:**
> To access your system GPU's from within the docker container, you must have [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) installed on your host machine.

```
docker run -it --rm --memory=100g --gpus device=0  kundajelab/chrombpnet:dev
```

### 2. Installation setup throuh github within a conda environment 

Create a clean conda environment with python >=3.8 
```
conda create -n chrombpnet python=3.8
conda activate chrombpnet
```

Install non-Python  requirements via conda
```
conda install -y -c conda-forge -c bioconda samtools bedtools ucsc-bedgraphtobigwig pybigwig meme
```

Git clone the staging branch of chrombpnet and install via pip

```
git clone https://github.com/kundajelab/chrombpnet.git
pip install -e chrombpnet
```
	
## QuickStart

### Bias-factorized ChromBPNet training

The command to train ChromBPNet with pre-trained bias model will look like this:

```
train_chrombpnet_model.sh \
  -i /path/to/input.bam \
  -t "bam" \
  -d "ATAC" \
  -g /path/to/hg38.fa \
  -c /path/to/hg38.chrom.sizes \ 
  -p /path/to/peaks.bed \
  -n /path/to/nonpeaks.bed \
  -f /path/to/fold_0.json \
  -b /path/to/bias.h5 \ 
  -o path/to/output/dir/ \
```

#### Input Format

- `-i`: input file path with filtered reads. Example files for supported types - bam, fragment, tagalign 
- `-t`: type of input file. Following string inputs are supported - "bam", "fragment", "tagalign". 
- `-d`: assay type.  Following types are supported - "ATAC" or "DNASE"
- `-g`: reference genome fasta file. Example file in human - hg38.fa
- `-c`: chromosome and size tab seperated file. Example file in human - hg38.chrom.sizes
- `-p`: Input peaks in narrowPeak file format, and must have 10 columns, with values minimally for chr, start, end and summit (10th column). Every region 	  is centered at start + summit internally, across all regions. Example file in  - peaks.bed.gz
- `-n`: Input nonpeaks (background regions)in narrowPeak file format, and must have 10 columns, with values minimally for chr, start, end and summit 	  	(10th column). Every region is centered at start + summit internally, across all regions. Example file in - nonpeaks.bed.gz
- `-f`: json file showing split of chromosomes for train, test and valid. Example file - 
- `-b`: Bias model in `.h5` format. Bias models are generally transferable across same assay types. Repository of pre-trained bias models for use - here. . Instructions to train custom bias model - here.
- `-o`: Output directory path

Please find helper scripts and instructions to filter input reads, generate peaks and non-peaks here. 

#### Output Format

The ouput directory will be populated as follows -

```
models\
	...
	chrombpnet.h5
	chrombpnet_nobias.h5 (TF-Model i.e model to predict bias corrected accessibility profile) 
	...
logs\
	...
	
intermediates\
	...

evaluation\
	...
	pwm_from_input.png 
	bias_metrics.json 
	chrombpnet_metrics.json
	chrombpnet_only_peaks.png
	chrombpnet_only_peaks.jsd.png
	profile_motifs.pdf
	counts_motifs.pdf
	footprints/bias_footprints_score.txt
	footprints/corrected_footprints_score.txt
	...
```

To interpret the output files, please find documentation on expected output, their interpretation and instructions to improve bias correction here. 

## Bias Model training

The command to train bias model will look like this:


```
train_bias_model.sh \
  -i /path/to/input.bam \
  -t "bam" \
  -d "ATAC" \
  -g /path/to/hg38.fa \
  -c /path/to/hg38.chrom.sizes \ 
  -p /path/to/peaks.bed \
  -n /path/to/nonpeaks.bed \
  -f /path/to/fold_0.json \
  -b /path/to/bias.h5 \ 
  -o path/to/output/dir/ \
```

#### Input Format

- `-i`: input file path with filtered reads. Example files for supported types - bam, fragment, tagalign 
- `-t`: type of input file. Following string inputs are supported - "bam", "fragment", "tagalign". 
- `-d`: assay type.  Following types are supported - "ATAC" or "DNASE"
- `-g`: reference genome fasta file. Example file in human - hg38.fa
- `-c`: chromosome and size tab seperated file. Example file in human - hg38.chrom.sizes
- `-p`: Input peaks in narrowPeak file format, and must have 10 columns, with values minimally for chr, start, end and summit (10th column). Every region 	  is centered at start + summit internally, across all regions. Example file in  - peaks.bed.gz
- `-n`: Input nonpeaks (background regions)in narrowPeak file format, and must have 10 columns, with values minimally for chr, start, end and summit 	  	(10th column). Every region is centered at start + summit internally, across all regions. Example file in - nonpeaks.bed.gz
- `-f`: json file showing split of chromosomes for train, test and valid. Example file - 
- `-b`: Bias model in `.h5` format. Bias models are generally transferable across same assay types. Repository of pre-trained bias models for use - here. . Instructions to train custom bias model - here.
- `-o`: Output directory path

Please find helper scripts and instructions to filter input reads, generate peaks and non-peaks here. 

#### Output Format

The ouput directory will be populated as follows -

```
models\
	...
	chrombpnet.h5
	chrombpnet_nobias.h5 (TF-Model i.e model to predict bias corrected accessibility profile) 
	...
logs\
	...
	
intermediates\
	...

evaluation\
	...
	pwm_from_input.png 
	bias_metrics.json 
	chrombpnet_metrics.json
	chrombpnet_only_peaks.png
	chrombpnet_only_peaks.jsd.png
	profile_motifs.pdf
	counts_motifs.pdf
	footprints/bias_footprints_score.txt
	footprints/corrected_footprints_score.txt
	...
```

To interpret the output files, please find documentation on expected output, their interpretation and instructions to improve bias correction here. 


## How to Cite
