![SCynergy 2026](./assets/scynergy2026.png){ width="640" .off-glb }

#  Hands-on: Urban wind simulation and visualization

![Wind on the Philharmonie](images/philharmonie-rt.png)

In this part, you will get a quick overview of an HPC workflow. 
Your will work with a simplified example of a Computational Fluid Dynamics (CFD) simulation of the **urban wind around the Philharmonie of Luxembourg**.
This workflow is designed for you to practice the different ways to access and use the [MeluXina supercomputer](https://docs.lxp.lu/system/overview/). 
You will proceed to the following steps:

1. [Step 1: Submit a batch job for a parallel CFD simulation and monitor its execution](#submit-of-a-parallel-cfd-simulation-monitor-execution)
2. [Step 2: Visualize and post-process the simulation result interactively](#post-processing-the-simulation-output)
3. [Step 3: Download the final post-processed results on your laptop](#download-the-final-output)

## ▶️ Submit of a parallel CFD simulation & monitor execution

The execution of large parallel computing job is usually done in **non-interactive mode** (or batch) using a submission script.
In this manner, the computation job is queued for execution on system and will be executed as soon as resource are available.
Hence, the user does not have to stay connected during the actual execution which can take place at any time (e.g., night or week-end).

This step is done using the shell access via a terminal.

### Shell Access

👉 Open a shell on MeluXina supercomputer. 

!!! tip

    You can use your prefered method among the of the two options configured in the [previous part](connections_meluxina.md):
    
    - [Direct shell access via SSH](connections_meluxina.md#command-line-access-using-ssh)
    - [Shell access via the web-portal](connections_meluxina.md#web-portal-access)

Once connected, you should see the MeluXina welcome banner:

![MeluXina welcome banner](images/meluxina_banner.png){.center}

### HPC Job Submission

A submission script for Slurm has been prepared for you to easily run this parallel HPC simulation.

👉 Submit the simulation job using the following command:
```bash
sbatch /project/home/p201259/materials/14April_GettingStartedWithMeluXina/wind_philharmonie/run.sh
```

!!! tip "Using the command line"

    To avoid mistake, simply copy the line above and paste it in the MeluXina terminal. Then press `Enter` to execute the command.
    
    If succesfull, you should see something like this:
    ```
    Submitted batch job 4472627
    ```

??? info "Details of the submission script"

    You can quickly inspect the content of submission script using the `cat` command 🔎
    
    ```bash
    cat /project/home/p201259/materials/14April_GettingStartedWithMeluXina/wind_philharmonie/run.sh
    ```
    

    ```bash linenums="1"
    #!/bin/bash -l
    #SBATCH --job-name WindPhilharmonie                        (1)
    #SBATCH --time 0-00:30:00
    #SBATCH --nodes 1
    #SBATCH --ntasks 16
    #SBATCH --cpus-per-task 1
    #SBATCH --partition cpu
    #SBATCH --qos default
    #SBATCH --account p201259
    #SBATCH --output SLURM_%x_%j.log
    
    echo "== Starting job ${SLURM_JOBID} at $(date)"
    
    # Setup OpenFOAM                                           (2)
    module load env/release/2025.1
    module load OpenFOAM/13-foss-2025a
    source "${FOAM_BASH}"
    
    # Create a new run directory populated with input files    (3)
    RUNDIR="/project/home/p201259/workspaces/${USER}/wind_philharmonie-${SLURM_JOBID}"
    mkdir -p "${RUNDIR}"
    rsync -avzR /project/home/p201259/materials/14April_GettingStartedWithMeluXina/wind_philharmonie/./ "${RUNDIR}"
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
    
    1. The `#SBATCH` directives specify the configuration of the Slurm job. 
    2. Setup the software environment, in this case we use OpenFOAM.
    3. Prepare simulation input: copy from a template and decompose input for parallel execution
    4. Run the OpenFOAM simulation in parallel

??? example "Running with more processes?"

    You feel playful and want to run the simulation with more parallel processes? 🧑‍🔬
    
    You can specify options on the command line to override the values of the script. For example, you can add `--ntasks 32` to run the simulation with 32 parallel processes: 
    ```bash
    sbatch --ntasks 32 /project/home/p201259/materials/14April_GettingStartedWithMeluXina/wind_philharmonie/run.sh
    ```
    
    Keep in mind that:
    
    - Resources are shared and limited during the training.
    - There are limitations on the number of processes you can run on a single compute node.
    - Using more processes does not systematically mean a faster execution. In this case, reconstructing the parallel output might actually be slower.


### Execution monitoring

Your HPC job is now queued on MeluXina supercomputer. You can monitor its status using the Slurm command `squeue`.

👉 Run the `squeue` command to see the status of your job:
```
squeue
```

!!! info "About the job state"

    The output of `squeue` shows information about your current jobs in the queue:
    ![`squeue` output](images/squeue_output.png){.center}
    
    The field `STATE` indicate the current status of the job. It typically goes from:
    ```mermaid
    flowchart LR
        PENDING --> CONFIGURING --> RUNNING
        RUNNING --> COMPLETED
        RUNNING --> FAILED
    ```
    
    After some time, the finished jobs (i.e., `COMPLETED` or `FAILED`) disapear from the list.

👉 Monitor the execution of your job until it is finished. Then you can continue to the next step.


## ▶️ Post-processing the simulation output

Once the execution of the simulation is over, we can proceed to the analysis of the results.
This step is done interactively using the [ParaView visualization software](https://www.paraview.org/), started from the MeluXina web-portal.

### Open ParaView

👉 Start the MeluXina web-portal: [**https://portal.lxp.lu/**](https://portal.lxp.lu/). 

!!! tip

    If needed, follow the instructions of the [previous part](connections_meluxina.md):
    
    - [Web-portal access](connections_meluxina.md#web-portal-access)

👉 Open the ParaView application.

1. Click on the [ParaView application on the web-portal](https://portal.lxp.lu/pun/sys/dashboard/batch_connect/sys/bc_meluxina_paraview/session_contexts/new).
2. Use the settings defined below and click the **Launch** button.
![Paraview settings](images/ood_paraview_settings.png){.center}
3. Wait for job to run and then click the **Launch ParaView** button as shown below.
![Paraview settings](images/ood_paraview_job.png){.center}

You now have the ParaView software running on a MeluXina compute node. 
The graphical interface is streamed in real time to your laptop. 
You can use it as any other graphical application, but keep in mind that there can be some additional latency when using a busy wifi network.

### Load the simulation results

We will now open the results of the urban wind simulation in Paraview.

👉 In the Paraview menu **File**, click **Load State...**

![Paraview load state](images/paraview_load_state-1.png){.center}

!!! info "Finding your simulation results"

    - Navigate through your files to locate the results of your simulation. 
    - They should be located in `/project/home/p201259/workspaces/u10XXXX/wind_philharmonie-447ZZZZ`, with
        - `u10XXXX` being your actual username;
        - `447ZZZZ` being the job ID of your submitted job.
    - In this directory, open the file named **`philharmonie.pvsm`**.

Then, make sure to use the settings below:
![Paraview load state](images/paraview_load_state-2.png){.center}

![Urban Wind around the Philharmonie in Paraview](images/paraview_philharmonie.png){.center}

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
