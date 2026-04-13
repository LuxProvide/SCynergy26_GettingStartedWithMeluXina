![SCynergy 2026](./assets/scynergy2026.png){ width="640" }

#  Hands-on: Urban wind simulation and visualization

![Wind on the Philharmonie](images/philharmonie-rt.png)

In this part, you will get a quick overview of an HPC workflow. 
With this example, you will run and post-process a simulation of urban wind around the Philharmonie of Luxembourg based on Computational Fluid Dynamics (CFD). 
This workflow is designed for you to practice the different ways to access and use the [MeluXina supercomputer](https://docs.lxp.lu/system/overview/). You will proceed to the following steps:

1. Submit a batch job for a parallel CFD simulation and monitor its execution.
2. Visualize and post-process the simulation result interactively.
3. Download the final post-processed results on your laptop.

## ▶️ Submit of a parallel CFD simulation & monitor execution

The execution of large parallel computing job is usually done in non-interactive mode (or batch) using a submission script.
In this manner, the computation job is queued for execution on system and will be executed as soon as resource are available.
The user does not have to stay connected during the actual execution which can take place at any time (e.g., night or week-end).

### TODO shell access





### TODO submit script

```bash
sbatch /project/home/p201259/materials/14April_GettingStartedWithMeluXina/wind_philharmonie_par16/run.sh
```

### TODO script detail

```bash linenums="1"
#!/bin/bash -l
#SBATCH --job-name WindPhilharmonie                        (1)
#SBATCH --time 0-00:30:00
#SBATCH --nodes 1
#SBATCH --ntasks 16
#SBATCH --cpus-per-task 1
#SBATCH --partition cpu
#SBATCH --qos default
#SBATCH --account lxp
#SBATCH --output SLURM_%x_%j.log

echo "== Starting job ${SLURM_JOBID} at $(date)"

# Setup OpenFOAM                                           (2)
module load env/release/2025.1
module load OpenFOAM/13-foss-2025a
source "${FOAM_BASH}"

# Create a new run directory populated with input files    (3)
RUNDIR="/project/home/p201259/workspaces/${USER}/wind_philharmonie_par16-${SLURM_JOBID}"
mkdir -p "${RUNDIR}"
rsync -avzR /project/home/p201259/materials/14April_GettingStartedWithMeluXina/wind_philharmonie_par16/./ "${RUNDIR}"
echo "Using run directory '${RUNDIR}'"
cd "${RUNDIR}"

# Set the number of processes in OpenFOAM settings
foamDictionary -entry numberOfSubdomains -set "${SLURM_NTASKS}" system/decomposeParDict

# Decompose the problem for parallel execution
time decomposePar -force

# Run the OpenFOAM solver in parallel                      (4)
time srun -n "${SLURM_NTASKS}" -c 1 foamRun -solver incompressibleFluid -parallel

# Reconstruct output after parallel execution
time reconstructPar

echo "== Finished job ${SLURM_JOBID} at $(date)"
```
{ .annotate }

1. The `#SBATCH` directives specify options that can 
2. Setup the software environment, in this case we use OpenFOAM.
3. Prepare simulation input: copy from a template and decompose input for parallel execution
4. Run the OpenFOAM simulation in parallel



## ▶️ Post-processing the simulation output

### Load the simulation results

![](images/paraview_load_state-1.png){.center}

![](images/paraview_load_state-2.png){.center}

![](images/paraview_philharmonie.png){.center}

### Create a video of the results

![](images/paraview_animation-1.png){.center}

![](images/paraview_animation-2.png){.center}

![](images/paraview_animation-3.png){.center}


## ▶️ Download the final output

![](images/ood_download-1.png){.center}

![](images/ood_download-2.png){.center}

![](images/philharmonie-wind-meluxina.gif){.center}



---

[![EPICURE](./assets/logo_epicure.png){ width="420" }](https://epicure-hpc.eu/) 
[![LuxProvide](./assets/logo_luxprovide.png){ width="320" }](https://luxprovide.lu)
