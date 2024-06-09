---
layout: default
title: NGS Analysis
parent: Workflows
nav_order: 3
---

<p align="center"><img src="../../assets/img/genemap-ngs.svg" height="50%" width="50%"></p>

---

# Quick Start
- [Installation](#install)
  - [Requirements](#requirements)
  - [Install](#procedure)
- [Things to note before running the workflow](#run)
- [Test the installation](#test)
- [Final setup](#setup)

---

# Installation <a name="install"></a>

## Requirements <a name="requirements"></a>
- [Nextflow](https://www.nextflow.io/docs/latest/getstarted.html#installation)
  
  Check that nextflow is installed on your cluster and load it.

{: .note-title }
> Preferred version:
>
> 21.10 or greater


- [Singularity](https://apptainer.org/docs/user/latest/quick_start.html#quick-installation)
  
  Singularity has now been changed to Apptainer, but most clusters to run it as singularity.
  We'll need singularity to pull the `gencall` docker container from our dockerhub account as
  a singularity image file.

  Check if singularity is installed on your cluster and load it.

{: .note-title }
> Preferred version:
>
> 3.5 or greater

---

## Install <a name="procedure"></a>
Navigate to the path where you would like to run the workflow from and run the code below.

```sh
wget https://github.com/GeneMAP-Research/genemapngs/archive/refs/tags/v0.1.tar.gz
tar zxvf v0.1.tar.gz
rm v0.1.tar.gz
cd genemapngs-0.1
```
---

{: .important .fs-3 }
> Without the main workflow configuration file (`nextflow.config`) nextflow cannot run at all.
> 
> Copy the `system.config` file to `nextflow.config`.
> ```
> cp system.config nextflow.config
> ```
>
> Then edit the `nextflow.config` file with the correct parameter values.

---


# Things to note before running the workflow <a name="run"></a>
---

The workflow runs on the concept of nextflow profiles.

There are three profile categories:
- **executors**: there are three executors based on where you are working
  - local: this can be used anywhere; your computer (laptop) or any cluster (slurm, pbspro, etc). 
  - slurm: cluster running a slurm job workload manager/scheduler.
  - pbspro: cluster running a pbspro job workload manager/scheduler.
<br>
- **containers**: there are three containers
  - apptainer: formerly singularity. Some clusters might still not run it.
  - singularity: now apptainer. Most clusters still run it.
  - docker: Most clusters do not run docker for security reasons. It can be used on local computers.
<br>
- **references**: there are three references
  - hg19: human reference build 37 or GRCh37
  - hg38: human reference build 38 or GRCh38
  - t2t: human Telomere-to-Telomere reference (T2T-CHM13)
<br>

The workflow commandline is built as follows.

```sh
nextflow run <workflow script> -profile <executor>,<container>,<reference> -w <work directory>
```

[Back to top](#top)


## Test the installation <a name="test"></a>
---
This example will pull and use a plink2 image (which is light-weight) using singularity.


{: .note .fs-3 }
> First, make sure nextflow and singularity are loaded and working.
> Check how these are loaded on your system. This should be done on a worker node via interactive
> job.
>
> ```sh
> module load <your nextflow version>
> ```
> 
> Most systems have singularity automatically loaded on worker nodes. Try by simply running `singularity`.
>
> Otherwise, load singularity
> ```sh
> module load <your singularity version>
> ```


## Test installation without selecting a profile
---
The workflow will be executed locally and nextflow will expect all tools to be already loaded. So, we 
must load plink2 for the test.

```sh

module load <your version of plink2>

./genemapngs.sh test

nextflow -c test.config run test.nf -w workdir 
```

If all goes well, you should see something like this, but can be different based on your plink2 version
```
N E X T F L O W  ~  version 23.04.3
Launching `test.nf` [condescending_hopper] DSL2 - revision: 943b38a3a4

ALIGNMENT WORKFLOW: TEST

pe=true
aligner=BWA
ftype=FASTQ
input_dir=/path/to/fastq/
output_dir=/path/to/save/results/
output_prefix=my-ngs
single_caller=gatk
exome=true
joint_caller=gatk
gvcf_dir=NULL
spark=false
threads=1
njobs=1

executor >  local (1)
[52/6e9e57] process > plink (processing ...) [100%] 1 of 1 ✔
PLINK2 is used for the test as it is light-weight and easily pulled from docker hub
PLINK v2.00a3.3LM 64-bit Intel (3 Jun 2022)    www.cog-genomics.org/plink/2.0/
(C) 2005-2022 Shaun Purcell, Christopher Chang   GNU General Public License v3

--pedmap <prefix>  : Specify .ped + .map filename prefix.
--ped <filename>   : Specify full name of .ped file.
--map <filename>   : Specify full name of .map file.

Workflow completed at: 2024-06-09T06:47:43.261741+02:00
     Execution status: OK
```
{: .fs-3 .lh-tight }


## Test installation selecting the singularity profile
---
Singularity is the preferred way of running the workflow since all tools needed are pulled directly from docker hub.
```sh
./genemapngs.sh test  #you don't need to run this again if you already did

nextflow -c test.config run test.nf -w workdir -profile singularity

# The result should be the same as above
```

[Back to top](#top)


# Final setup <a name="setup"></a>
Central to nexflow workflow are configuration (config) files. They direct nextflow on where to look for stuff or place stuff.
There are a few more config files, in addition to the main config file, that we need to setup for the workflow to run smoothly. 
We will do this step by step
- ...


{: .note .fs-3 }
> You might see a few warnings:
> - regarding `echo` and `debug`. These are caused by different versions of nextflow and do not pose any issues.
> - regarding singularity cache directory. As long as you set a value for `containers_dir` in your `nextflow.config` file, it should be no problem.
>
> If the containers directory is not set, the workflow will create one in your work directory.


_under development_