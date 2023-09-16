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
  - [Config parameters](#parameters)
- [Running the workflow](#run)
  - [Test](#test)


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


## Config parameters <a name="parameters"></a>
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
  - NB: The cluster file can be custom-made. See [this document](https://www.illumina.com/content/dam/illumina-marketing/documents/products/technotes/custom-cluster-file-cnv-tech-note-770-2017-017.pdf){:target="_blank"} for more information.

---

`fasta_ref` and `bam_alignment`:
  - For hg19 dataset, only the reference fasta file is required.
  - The gtc2vcf tool adds a functionality to process calls generated in hg19 in hg38. This requires you to realign the flanking sequences from the manifest files. This is described [here](https://github.com/freeseek/gtc2vcf#using-an-alternative-genome-reference){:target="_blank"}.
  - NB: Provide these with their absolute paths.

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
  - Path where docker images (the gencall image) will be pulled from our [dockerhub](https://hub.docker.com/repository/docker/sickleinafrica/gencall/general){:target="_blank"} and stored.
  - This path should have sufficient storage capacity. 
    - The gencall image is about 530 MB in size.
    - The plink2 test image is abot 80 MB.

---

`idat_threads`, `idat_max_time` and `idat_max_memomy`:
  - These resources will be used when converting from IDAT to GTC with gencall.
  - From experience, this step works pretty well with minimal resources.
  - Try it with the default ones, and only edit if you are unsatisfied with the outcome.
  - Requesting minimal resources will enable your jobs to fit within the cracks and get submitted well ahead of other jobs that are requesting massive resources.

`gtc_threads`, `gtc_max_time` and `gtc_max_memomy`:
  - The GTC to VCF step is only one job, processed using bcftools.
  - From experience, it requires a little more resources.
  - Again, try with these default ones.

---


# Running the workflow <a name="run"></a>
---

The workflow runs on a simple logic based on nextflow profiles.

There are three profile categories in the workflow:
- **executors**: there are three executors based on where you are working
  - local: this can be used anywhere; your computer (laptop) or any cluster (slurm, pbspro, etc). 
  - slurm: cluster running a slurm job workload manager/scheduler.
  - pbspro: cluster running a pbspro job workload manager/scheduler.


- **containers**: there are three containers
  - apptainer: Formerly singularity. Some clusters might still not run it
  - singularity: Now apptainer, Some clusters might still run it (actually most systems still do).
  - docker: Most clusters do not run docker for security reasons. It can be used on local computers.

The workflow commandline is built as follows.

```sh
nextflow run <workflow script> -profile <executor>,<container>,<reference> -w <work directory>
```

{: .important }
> Nextflow can generate many large intermediate files. So it is important to specify an appropriate work directory with the `-w` option.
>
> Nextflow needs these intermediates files to resume your job in case the workflow terminates without completing. So only delete the work directory when you are sure your workflow is complete and you are satisfied with all the outputs. Else, rerunning it will start from scratch.


## Test <a name="test"></a>
---
Let's see an example. This example will pull and use a plink2 image (which is light-weight) using singularity.


{: .note }
> First, make sure nextflow and singularity are loaded.
> Check how these are loaded on your system
>
> `module load nextflow`
>    
> `module load singularity`

---

### Running with all three profile categories
```sh
nextflow run test.nf -profile test,local,singularity,hg19 -w "./work/"
```

NB: For the test, we add a test profile before the three categories we mentioned above.



If all goes well, you should see a message like this
```
N E X T F L O W  ~  version 21.10.6
Launching `test.nf` [jolly_jennings] - revision: c99dfcf444

        IDAT to VCF: TEST

idat_dir        = YOUR IDAT PARENT DIRECTORY
manifest_bpm    = CHIP-SPECIFIC BPM MANISFEST
manifest_csv    = CHIP-SPECIFIC CSV MANISFEST
cluster_file    = YOUR CLUSTER FILE
fasta_ref      = HUMAN REFERENCE FASTA
output_prefix   = YOUR OUTPUT FILE NAME PREFIX
output_dir      = PATH WHERE YOUR OUTPUT IS STORED
containers_dir  = PATH WHERE CONTAINERS ARE STORED
account         = humgen        # CHANGE TO YOURS
partition       = sadacc-short      # CHANGE TO YOURS

executor >  local (1)
[5f/ddaf29] process > plink (processing ... YOUR IDAT PARENT DIRECTORY) [100%] 1 of 1 ✔
PLINK v2.00a4.7LM 64-bit Intel (12 Sep 2023)   www.cog-genomics.org/plink/2.0/
(C) 2005-2023 Shaun Purcell, Christopher Chang   GNU General Public License v3

--pedmap <prefix>  : Specify .ped + .map filename prefix.
--ped <filename>   : Specify full name of .ped file.
--map <filename>   : Specify full name of .map file.

Workflow completed at: 2023-09-16T13:35:26.858+02:00
     Execution status: OK
```

{: .note }
> You might see a few warnings:
> - regarding `echo` and `debug`. These are caused by different versions of nextflow and do not pose any issues.
> - regarding singularity cache directory. As long as you set a value for `containers_dir` in your `nextflow.config` file, it should be no problem.
>
> If the containers directory is not set, the workflow will create one in your work directory.

---

### Running without selecting an executor
```sh
nextflow run test.nf -profile test,singularity,hg19 -w "./work/"
```

The local executor will be used by default.

The output should be similar as above. However this time, the plink2 image will not be redownloaded since it is already present in the containers directory we set.



### Running on a cluster without selecting a container
```sh
# first load plink2 on your cluster

# on the UCT HPC cluster, it is
module load software/plink-2.00a

# then run
nextflow run test.nf -profile test,slurm,hg19 -w "./work/"
```

```
N E X T F L O W  ~  version 21.10.6
Launching `test.nf` [astonishing_solvay] - revision: c99dfcf444

        IDAT to VCF: TEST

idat_dir        = YOUR IDAT PARENT DIRECTORY
manifest_bpm    = CHIP-SPECIFIC BPM MANISFEST
manifest_csv    = CHIP-SPECIFIC CSV MANISFEST
cluster_file    = YOUR CLUSTER FILE
fasta_ref      = HUMAN REFERENCE FASTA
output_prefix   = YOUR OUTPUT FILE NAME PREFIX
output_dir      = PATH WHERE YOUR OUTPUT IS STORED
containers_dir  = PATH WHERE CONTAINERS ARE STORED
account         = humgen        # CHANGE TO YOURS
partition       = sadacc-short      # CHANGE TO YOURS

executor >  slurm (1)
[3d/e89581] process > plink (processing ... YOUR IDAT PARENT DIRECTORY) [100%] 1 of 1 ✔
PLINK v2.00a3.6LM AVX2 Intel (14 Aug 2022)     www.cog-genomics.org/plink/2.0/
(C) 2005-2022 Shaun Purcell, Christopher Chang   GNU General Public License v3

--pedmap <prefix>  : Specify .ped + .map filename prefix.
--ped <filename>   : Specify full name of .ped file.
--map <filename>   : Specify full name of .map file.

Workflow completed at: 2023-09-16T13:36:57.528+02:00
     Execution status: OK
```

You can put these in you job submission script. See below.



{: .important }
> If submittng your job to a workload manager like slurm, you must specify your account and partition in the `nextflow.config` file.
> 
> But if you already requested an interactive job, then either select a local executor or do not select an executor at all.


Example on the UCT HPC cluster: `test.sbatch`
```
#!/usr/bin/env bash
#SBATCH --account humgen
#SBATCH --partition sadacc-short
#SBATCH --nodes 1
#SBATCH --time 06:00:00
#SBATCH --ntasks 1

module load software/plink-2.00a

nextflow run test.nf -profile test,slurm,hg19 -w "./work/"
```

{: .highlight }
> In my sbatch script, I only requested 1 thread/cpus (--ntasks) because I am only running one nextflow commandline.
>
> This will ensure that the job is submitted quickly, allowing nextflow to request the rest of the resources for each process.
>
> However, we have set enough time (in HH:MM:SS - one more than the maximum time requested in the config file) to avoid premature termination of the jobs.



{: .highlight .fs-1 }
> Test Font size 1

_under development_
