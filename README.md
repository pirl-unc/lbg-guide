# lbg-guide
Basic usage tips for the LBG cluster at UNC-CH

## What is LBG?
Scientific computing at UNC-CH happens mostly on the [Longleaf Cluster](https://its.unc.edu/research-computing/longleaf-cluster/). Additionally, the [Lineberger Cancer Center](https://unclineberger.org/) also has its own dedicated cluster called LBG, which is administered by the [Lineberger Bioinformatics Core](https://lbc.unc.edu/). This unofficial repository will hopefully serve as a basic guide for LCCC researchers to get started using LBG. There is additional information on the [LBG wiki](https://lbgwiki.bioinf.unc.edu/index.php?title=Main_Page). 

## Access

To connect to any LBG login node: 

```sh
ssh <your_onyen>@login.bioinf.unc.edu
```

If you want to maintain a tmux session and reconnect to it later, it is better to instead connect to a particular login node:

```sh
ssh <your_onyen>@vm-login-2-0.bioinf.unc.edu
```

## Usage

### Start an interactive session 

```sh 
srun -p allnodes --pty -c 1 --mem 2g bash
```

### Scheduling wrapped commands

Jobs are typically submitted using Slurm's [sbatch](https://slurm.schedmd.com/sbatch.html) command. For example this command wraps running a singularity container and submits it for batch scheduling: 

```sh
sbatch 
  --priority=TOP 
  --partition allnodes 
  --mail-type ALL 
  --mail-user <your_onyen>@ad.unc.edu 
  --chdir /datastore/nextgenout5/share/labs/Vincent_Lab/members/dbortone/rstudio-common/projects/testing/mtb_bridge/_run_scripts 
  --error /datastore/nextgenout5/share/labs/Vincent_Lab/members/dbortone/rstudio-common/projects/testing/mtb_bridge/_run_scripts/slurm_error_gene_VST_pearson_correlation_plot.txt 
    --output /datastore/nextgenout5/share/labs/Vincent_Lab/members/<your_onyen>/rstudio-common/projects/testing/mtb_bridge/_run_scripts/slurm_output_gene_VST_pearson_correlation_plot.txt 
    --job-name gene_VST_pearson_correlation_plot 
    --cpus-per-task 4 
    --mem=4g 
    --time=UNLIMITED 
    --wrap="singularity exec --nohttps --bind /datastore,/home,/datastore/alldata/shiny-server/rstudio-common/:/rstudio-common docker://benjaminvincentlab/rserver-binfotron:4.0.3.10 Rscript /datastore/nextgenout5/share/labs/Vincent_Lab/members/dbortone/rstudio-common/projects/testing/mtb_bridge/_run_scripts/gene_VST_pearson_correlation_plot.R"
```

### Scheduling a shell script

Alternatively, you can create a shell script with `#SBATCH` directives and pass that whole script to `sbatch` for scheduling. For example, this script (named `SAMP1.sh`): 

```sh

#!/bin/bash
#SBATCH --job-name SAMP1
#SBATCH --partition allnodes
#SBATCH --time UNLIMITED
#SBATCH --cpus-per-task 1
#SBATCH --mem-per-cpu 10g
#SBATCH --output /datastore/alldata/shiny-server/rstudio-common/dbortone/dataset_prep/OSU_mHA_VL194/cluster_output/mHA/SAMP1_out.txt
#SBATCH --error /datastore/alldata/shiny-server/rstudio-common/dbortone/dataset_prep/OSU_mHA_VL194/cluster_output/mHA/SAMP1_err.txt
#SBATCH --chdir /datastore/nextgenout5/share/labs/Vincent_Lab/tools/mHA

singularity exec --bind /datastore /datastore/nextgenout5/share/labs/Vincent_Lab/singularity/mHA.simg \
  python mHA_prediction_v2.py \
  --ref_dir /datastore/nextgenout5/share/labs/Vincent_Lab/tools/mHA \
  --vcf genetic_data_nonsyn_info09.vcf \
  --recipient SAMP1 \
  --donor SAMP123 \
  --peptide_lengths 8,9,10,11 \
  --HLAtype A_01_01,A_24_02,B_57_01,C_06_02,C_06_02 \
  --temp_dir temp
```
...can be scheduled using the following command:

```sh
sbatch /datastore/alldata/shiny-server/rstudio-common/<your_onyen>/dataset_prep/OSU_mHA_VL194/cluster_commands/mHA/SAMP1.sh
```
### Installing programs

Unlike other scientific computing clusters like Longleaf, LBG doesn't use the [environment modules](https://www.admin-magazine.com/HPC/Articles/Environment-Modules) to load programs. Instead, you should install conda: 

```sh 
cd ~/
wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.11.0-Linux-x86_64.sh
bash Miniconda3-py39_4.11.0-Linux-x86_64.sh
```

...and then try to work within a different [conda environment](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) for each project. 

## Storage

LBG and Longleaf do not share a filesystem, so if you would like to access files from Longleaf you must use [scp](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/) to copy them between clusters. 


### Quota

You are limited to 1 TB in your /home directory. You can check your current limit with `quota -s`. 

### Usage

You can see how much space is used by a lab directory like this: 

```sh
du -sHh  /datastore/lbcfs/labs/rubinsteyn_lab
```

### Backups

Most storage areas on LBG have timed backups.  From a login node run `cd .snapshot` to see hourly and weekly snapshots.

## Cluster info

### Active nodes in different partitions
To see which nodes are active on the cluster run:

```sh
sinfo -N -o "%.15C %.10e %.10m %.10t %.30N" -p allnodes
```

Different helpful partitions (-p flag):

* `allnodes`:  most jobs will go here
* `datamover`: for transferring files to/from the outside
* `dockerbuild`: special access needed, for building docker images in your home directory.
* `interactive`: if the cluster is busy some resources are kept reserved for interactive sessions

### How many jobs are currently running?

```sh
squeue -o %u | grep -v USER | sort | uniq -c | sort -nr
```


## Assistance 

If you need help with something on the cluster send an email to [informaticshelp@unc.edu](mailto:informaticshelp@unc.edu), which will then reach [Lineberger Bioinformatic Core personnel](https://lbc.unc.edu/index.php/personnel) such as the director Sai Balu or infrastructure supervisor Feri Zsuppan. 
