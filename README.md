# Installing OpenFOAM on the LUMI supercomputer using EasyBuild
This guide explains how to compile OpenFOAM using EasyBuild on the [LUMI supercomputer](https://lumi-supercomputer.eu). It is explained how to install both the Foundation and ESI version of OpenFOAM, i.e., OpenFOAM-9 and OpenFOAM v2106, respectively. Should you need any other versions you may use one of the existing EasyBuild files and modify as needed. The people at LUMI keep a [GitHub repo](https://github.com/Lumi-supercomputer/LUMI-EasyBuild-contrib/tree/main/easybuild/easyconfigs/o/OpenFOAM) with the files, which serve as excellent starting points.

Throughout this guide, ```<project_id>``` refers to your project ID allocated to you by LUMI (commonly includes "project_"). In my case the ```<project_id>``` is "project_465000092". Go ahead and export your own project id:
```shell
export project_id=project_465000092
```

## 1. Prepare Environment

*1a)* First, set the EBU_USER_PREFIX variable. To make it permanent, run:
```shell
echo "export EBU_USER_PREFIX=/scratch/$project_id/EasyBuild" >> $HOME/.bashrc
```
*1b)* Reload the shell:
```shell
source $HOME/.bashrc
```
## 2. Compile and load OpenFOAM

In this step, we will install OpenFOAM-v2106. The procedure is easily adapted to other OpenFOAM versions.

Load required modules:
```shell
module load LUMI/22.08
```
```shell
module load partition/C
```
```shell
module load EasyBuild-user/LUMI
```
Next, search for and list the available OpenFOAM versions:
```shell
eb -S OpenFOAM
```

In the output you will see the avialable OpenFOAM versions (e.g., OpenFOAM-v2106-cpeGNU-22.08.eb corresponds to OpenFOAM-v2106). Finally, compile OpenFOAM-v2106 version with
```shell
eb --try-toolchain-version=22.08 --robot OpenFOAM-v2106-cpeGNU-22.08.eb
```
We will now see how to load OpenFOAM-v2106.

*Note:* The following commands can instead be added to the top of your SLURM file. See 
[3. Example of slurm file](#3-example-of-slurm-file).

First, load the required modules ```LUMI/21.12``` and ```partition/C```:
```shell
module load LUMI/22.08
```
```shell
module load partition/C
```
Next, load the freshly compiled OpenFOAM module:
```shell
module load OpenFOAM/v2106-cpeGNU-22.08
```
Finally, source the OpenFOAM installation:
```shell
source $EBROOTOPENFOAM/etc/bashrc WM_COMPILER=Cray WM_MPLIB=CRAY-MPICH
```
Now you should have sourced a fully-working OpenFOAM-v2106 installation. You can test your installation by:
```shell
simpleFoam -help
```


## 3. Example of slurm file
Below is an example of a [slurm file](https://github.com/jakobhaervig/openfoam-lumi-hpc-installation/blob/main/slurmFile). I recommend placing the slurm file in the OpenFOAM case directory. In the following remember to change:
- ```<simulation_name>```: A user-specified name to easier keep track of your running simulations.
- ```<project_id>```: The ID given to the project (including "project_").
- ```<email_address>```: The e-mail address that will receive updates.
```bash
#!/bin/bash -l

#SBATCH --job-name=<simulation_name>    # Job name
#SBATCH --output=<simulation_name>.o%j  # Name of stdout output file
#SBATCH --error=<simulation_name>.e%j   # Name of stderr error file
#SBATCH --partition=standard            # Partition (queue) name
#SBATCH --nodes=1                       # Total number of nodes
#SBATCH --ntasks=128                    # Total number of mpi tasks
#SBATCH --mail-type=all                 # Send email at begin and end of job
#SBATCH --account=<project_id>          # Project for billing
#SBATCH --mail-user=<email_address>     # Email address
#SBATCH --time 1-00:00:00               # Max time

# Load required modules
module load LUMI/22.08
module load partition/C
module load OpenFOAM/v2106-cpeGNU-22.08

# Source the OpenFOAM installation
source $EBROOTOPENFOAM/etc/bashrc WM_COMPILER=Cray WM_MPLIB=CRAY-MPICH

# Run OpenFOAM utilities     
blockMesh
decomposePar

# Start parallel simulation
srun pisoFoam -parallel
```

Finally, commit the simulation to the slurm job scheduling system (here ```slurmFile``` is the filename of the slurm file):
```shell
sbatch slurmFile
```

You may check the status of your running jobs by:
```shell
squeue -u $USER
```

And cancel running jobs using the ```JOBID``` shown by the above command:
```shell
scancel JOBID
```

