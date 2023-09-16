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
See how to install `genemapgwas` workflow [here](../index.md)


After installing the workflow, navigate to the illumina folder.

```sh

cd genemapgwas-0.3/illumina/

```

---


# Edit the Configuration file <a name="config"></a>
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

---




_under development_
