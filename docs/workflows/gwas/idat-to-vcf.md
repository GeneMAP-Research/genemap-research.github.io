---
layout: default
title: IDAT to VCF
parent: gwas
grand_parent: Workflows
nav_order: 1
---

<p align="center"><img src="../../../assets/img/genemap-gwas.svg" height="50%" width="50%"></p>

---

# Convert Illumina IDAT files to VCF/PED
---

# Contents
- [Installation](#install)
- [Edit the configuration file](#config)


---


# Installation <a name="install"></a>
---
See how to install the `genemapgwas` workflow [here](index.md)


After installing the workflow, navigate to the illumina folder.

```sh
cd genemapgwas-0.1/illumina/
```


# Edit the Configuration file <a name="config"></a>
---
In the `illumina` folder is a master configuration file '`nextflow.config`' where all your input options will be specified.

Only edit the lines displayed below.

```
params {

  // data-related parameters
  idat_dir          = ""
  manifest_bpm      = ""
  manifest_csv      = ""
  cluster_file      = ""
  fasta_ref         = ""
  bam_alignment     = ""                          // if processing in build 38 (hg38), provide a bam alignment file
  output_prefix     = ""
  output_dir        = ""


  // computing resource-related parameters
  account           = ""                          // if using SLURM or PBSPRO, provide your cluster project or account name
  partition         = ""                          // if using SLURM or PBSPRO, specify the queue or partition name
  njobs             = 5                           // number of jobs to submit at once each time
  containers_dir    = ""                          // if using singularity containers, provide a path where the images will be stored.
                                                  // NB: The path should have sufficient storage capacity


  // IDAT to GTC: requires less resources
  idat_threads      = 4                           // number of cpus/threads per job
  idat_max_time     = 1.h                         // maximum time each job is allowed to run (h - hours; m - minutes)
  idat_max_memory   = 2.GB                        // maximum memory each job is allowed to use (GB - gigabyte; MB - megabyte)


  // GTC to VCF: requires more resources
  gtc_threads       = 10                           // number of cpus/threads per job
  gtc_max_time      = 3.h                         // maximum time each job is allowed to run (h - hours; m - minutes)
  gtc_max_memory    = 3.GB                        // maximum memory each job is allowed to use (GB - gigabyte; MB - megabyte)
}
```


## Config parameters
Let's go through each parameter one after the other

---

`idat_dir`:
  - The illumina gencall algorithm needs you to provide a folder that contains pairs of green and red idat files (see example below `034928383`).
  - You many have multiple folders, each containing a different number of idat pairs (example below shows one idat folder containing two pairs of idat files).
  - The folders are contained within a folder called `intensities`. This is the path we provide to the workflow (the absolute path).
  - The workflow will grab all the sub-folders containing the idat files and pass them to genecall, hence enabling batch processing.


```
.
├── intensities
│   ├── 034928383
│   │   ├── file1.grn.idat
│   │   ├── file1.red.idat
│   │   ├── file2.grn.idat
│   │   └── file2.red.idat
│   ├── 236572634
│   ├── 562525763
│   └── 748974817

```
---

`manifest_bpm`, `manifest_csv` and `cluster_file`:
  - These files are specific to the chip that was used to genotype your samples and should be provided by the genotyping company.
  - Request them if you don't have them.
  - For the H3Africa chip, you can get these files from [here](https://chipinfo.h3abionet.org/downloads){:target="_blank"}.
  - NB: The cluster file can be custom-made. See [this document](https://www.illumina.com/content/dam/illumina-marketing/documents/products/technotes/custom-cluster-file-cnv-tech-note-770-2017-017.pdf) for more information.

---

`fasta_ref` and `bam_alignment`:
  - For hg19 dataset, only the references fasta file is required.
  - The gtc2vcf tool adds a functionality to process calls generated in hg19 in hg38. This requires you to realign the flanking sequences from the manifest files. This is described [here](https://github.com/freeseek/gtc2vcf#using-an-alternative-genome-reference){:target="_blank"}.

---

`output_prefix`: Output file name
`output_dir`: Path to save output

---

If working on a SLURM or PBSPRO cluster, you might be associated to a specific group

`account`: 
  - If you are required to specify an account name (slurm) or project name (pbspro) in your sbatch or qsub job submission scripts, provide it here.

`partition`:
  - If you are required to select a specific partition (queue) when requesting resources, provide it here.

If you are not required to specify any of these in your job submission scripts, leave these parameters empty.

---

`njobs`: 
  - Number of jobs (processes; e.g. each idat file pair) to submit to the cluster each time.
  - Some accounts have limits to the number of jobs they can submit. It is best to set a value that does not exceed this limit.

---

`containers_dir`:
  - Path where docker images (the gencall image) will pulled from our [dockerhub](https://hub.docker.com/repository/docker/sickleinafrica/gencall/general) and stored.
  - This path should have sufficient storage capacity. 
    - The gencall image is about 530 MB in size.
    - The plink2 test image is abot 80 MB.

---

`idat_threads`, `idat_max_time` and `idat_max_memomy`:
  - These resources will be used when converting from IDAT to GTC with gencall.
  - From tests, this step works pretty well with minimal resources.
  - Try it with the default ones, and only edit if your unsatisfied with the outcome.
  - Requesting minimal resources will enable your jobs to fit within the cracks and get submitted well ahead of other jobs that are requesting massive resources.

`gtc_threads`, `gtc_max_time` and `gtc_max_memomy`:
  - The GTC to VCF step is only one job, processed using bcftools.
  - From experience, it requires a little more resources.
  - Again, try with these default ones.

---



_under development_
