# Installing OpenFOAM on the LUMI supercomputer using EasyBuild
This guide explains how to compile OpenFOAM using EasyBuild on the [LUMI supercomputer](https://lumi-supercomputer.eu). It is explained how to install both the Foundation and ESI version of OpenFOAM, i.e., OpenFOAM-9 and OpenFOAM v2106, respectively. Current (August 2022) LUMI toolchains are 21.12 and 22.06 (expected to be superseeded by 22.08). With EasyBuild, OpenFOAM v9 is available with the 21.12 toolchain while OpenFOAM v2106 is available with the 22.06 toolchain.

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
## 2. Compile and load OpenFOAM (versions 9 and v2106)
### OpenFOAM-9 
Load required modules:
```shell
module load LUMI/21.12
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

In the output you will see the avialable OpenFOAM versions (e.g., OpenFOAM-9-cpeGNU-21.12.eb corresponds to OpenFOAM-9). Compile OpenFOAM-9 version with
```shell
eb --try-toolchain-version=21.12 --robot OpenFOAM-9-cpeGNU-21.12.eb
```
We will now see how to load OpenFOAM-9.

*Note:* The following commands can instead be added to the top of your SLURM file. See 
[3. Example of slurm file](#3-example-of-slurm-file).

First, load the required modules ```LUMI/21.12``` and ```partition/C```:
```shell
module load LUMI/21.12
```
```shell
module load partition/C
```
Next, load the freshly compiled OpenFOAM module:
```shell
module load OpenFOAM/OpenFOAM-9-cpeGNU-21.12
```
Finally, source the OpenFOAM installation:
```shell
source $EBROOTOPENFOAM/etc/bashrc WM_COMPILER=Cray WM_MPLIB=CRAY-MPICH
```
Now you should have sourced a fully-working OpenFOAM-9 installation. You can test your installation by:
```shell
simpleFoam -help
```

### OpenFOAM v2106 
The compilation of OpenFOAM v2106 under toolchain 22.06 is slightly more complicated due to compilation issues between GCC 11 and at least certain versions of GLIBC. 

Two files that are not currently on the system should be put
together in a subdirectory, preferably with no further subdirectories:

```shell
wget https://raw.githubusercontent.com/Lumi-supercomputer/LUMI-EasyBuild-contrib/OpenFOAM/22.06/easybuild/easyconfigs/c/CGAL/CGAL-4.12.2-cpeGNU-22.06-OpenFOAM.eb
```
```shell
wget https://raw.githubusercontent.com/Lumi-supercomputer/LUMI-EasyBuild-contrib/OpenFOAM/22.06/easybuild/easyconfigs/o/OpenFOAM/OpenFOAM-v2106-cpeGNU-22.06.eb
```
Enter that subdirectory and load modules:
```shell
module load LUMI/22.06
```
```shell
module load partition/C
```
```shell
module load EasyBuild-user/LUMI
```
Now, install OpenFOAM from your chosen subdirectory using 
```shell
eb -r . CGAL-4.12.2-cpeGNU-22.06-OpenFOAM.eb OpenFOAM-v2106-cpeGNU-22.06.eb
```
We will now see how to load OpenFOAM v2106.

*Note:* The following commands can instead be added to the top of your SLURM file. See 
[3. Example of slurm file](#3-example-of-slurm-file).

First, load the required modules ```LUMI/22.06``` and ```partition/C```:
```shell
module load LUMI/22.06
```
```shell
module load partition/C
```
Next, load the freshly compiled OpenFOAM module:
```shell
module load OpenFOAM/v2106-cpeGNU-22.06
```
Finally, source the OpenFOAM installation:
```shell
source $EBROOTOPENFOAM/etc/bashrc WM_COMPILER=Cray WM_MPLIB=CRAY-MPICH
```
Now you should have sourced a fully-working OpenFOAM v2106 installation. You can test your installation by:
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
module load LUMI/21.08
module load partition/C
module load OpenFOAM/OpenFOAM-9-cpeGNU-21.08

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

