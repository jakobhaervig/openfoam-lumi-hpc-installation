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
#SBATCH --time 1-00:00:00               # Max time (days-hours:minutes:seconds)

# Load required modules
module load LUMI/23.09 partition/C EasyBuild-user
module load OpenFOAM/v2312-cpeGNU-23.09

# Source the OpenFOAM installation
source $EBROOTOPENFOAM/etc/bashrc WM_COMPILER=Cray WM_MPLIB=CRAY-MPICH

# Run OpenFOAM utilities     
blockMesh
decomposePar

# Start parallel simulation
srun pisoFoam -parallel