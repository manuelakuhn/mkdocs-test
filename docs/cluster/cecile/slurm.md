## What is Slurm?

Let's start from the beginning, as you already know a cluster is made up of multiple resources which can be exploited to run several tasks at the same time. In order to achieve this goal, you need to use a so called job scheduler, in our case is [Slurm](https://slurm.schedmd.com/quickstart.html), an open source cluster management and job scheduler system.

The job scheduler allows you to specify computational tasks, known as **jobs**, that are going to be automatically distributed across the available and most suitable resources. A job can be whatever task you want it to be as long as it respects specific [restrictions], for example it could be a specific section of your preprocessing analysis, or the whole analysis of a single subject.

[restrictions]: #job-restrictions-and-defaults

## How to think about jobs

If you are using Cecile, you would most likely need to run multiple parallel jobs, the key to achieve that is to create tasks that are suitable for a job. This means that your code should be written with jobs in mind. 

- Create parallelizable scripts, for example: a script for a given step of your analysis should take as input only one subject, in this way you can run this step for each subject in parallel, thus having as many jobs as the number of your subjects. 
- Chunk your analysis in steps, in this way, especially if you are new to slurm, you have more control on your analysis and you avoid running extremely long job. This does not prevent you from running all the steps together as long as you are running one subject per job.

## How to run a job in Slurm

There are different types of jobs in slurm, you just need to choose the most suitable for your case. But before running your jobs there are a few things your need to do:

1. Activate the [software stack] environment or load the module that you need for your code.
   [software stack]: ../software/##how-to-use-the-stacks
2. Make sure that the code you would like to use for your jobs works properly. Especially at the beginning it might difficult to identify the reason why a job is failing, thus by making sure that your scripts runs as intended you can restrict your error space.
3. Make sure your scripts (e.g. in bash) are executable. If you have never done that make sure to do the following:
   
   Add the so called **shebang** on top of your bash script, it will tell the system to use the bash interpreter to run the code:
   ```bash
   #!/bin/bash

   ```
   Once you have done that, run the following command to make your script exacutable:
   ```bash
   $ chmod +x <script.sh>
   ``` 
   For python use the following shebang in the first line of your script:
   ```python
   #!/usr/bin/env python3
   ```
   And then make it exacutable from the command line:
   ```bash
   $ chmod +x <script.py>
   ```

4. Create, if you do not have it yet, a folder called `slurm_logs` (or you can give it another meaningful name), in the same directory as the scripts are, where slurm can dump the logs reporting error files and logs. 

In order to specify a job in Slurm, you need to make a few decisions and provide a few essential information to the system beforehand. 

!!! note "Ask yourself the following questions before setting up your job"

    1. How long does your job need to run?</b>  
    2. How many CPUs does your job need?</b>  
    3. How much memory does your job need?</b>  
    4. Does your job require any special hardware feature?



The following parameters are not all mandatory, but we **strongly recommend** to define all of them. If you do not define them properly your job might not run as expected, might stop or even fail and you might involuntarily take resources away from other users.

Here it is how you turn the previous questions in parameters for slurm.

```bash
#!/bin/sh

NOTE: Keep in mind that for your slurm script the # before slurm commands is not interpreted as a shell comment 

#SBATCH --cpus-per-task=1                     # number of cpu you request for a task
#SBATCH --nodes=1                             # number of requested nodes, keep it to 1, it is fine for our kind of analyses
#SBATCH --mem-per-cpu=1g                      # amount of memory you require per cpu
#SBATCH --time=01:00:00                       # maximum duration you assign to a job (D-HH:MM:SS)
#SBATCH --output=slurm_logs/output-%A-%a.out  # job printed output
#SBATCH --error=slurm_logs/error-%A-%a.err    # job related errors 

```

=== "Single job"

    It allowes to run only a single job.</b>  
    **Use case:** It is ideal when you want to test a specific analysis and test it on one subject, or when you need to run just one analysis like a group statistical analysis.

    ```bash
    #!/bin/sh


    #SBATCH --mail-user=<name.lastname>@ovgu.de
    #SBATCH --mail-type=BEGIN,END,FAIL
    #SBATCH --job-name=my_job
    #SBATCH --cpus-per-task=1
    #SBATCH --nodes=1
    #SBATCH --mem-per-cpu=1g
    #SBATCH --time=01:00:00
    #SBATCH --output=slurm_logs/output-%A-%a.out
    #SBATCH --error=slurm_logs/error-%A-%a.err

    ./my_script.sh 
    ```

    You can now run `squeue` to see the status of your job, let's understand the output:


    **A running job**
    ```
                JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            217379_0       all fmriprep    user   R       1:10      1 compute05
    ```

    **A pending job**
    ```
                JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            217379_0       all fmriprep    user  PD       1:10      1 compute05
    ```

    **A idled job**
    ```
                JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            217379_5       all bad_job     user   IDLE    0:00      1 compute05
    ```


    - `JOBID`: Indentification number of your job
    - `PARTITION`: Name of partition your jobs are currently using 
    - `NAME`: Name you gave to your job
    - `USER`: Your username
    - `ST`: Status of your job:
    - `R`: running job
    - `PD`: pending job, it is waiting for resources to be freed
    - `IDLE`: idled job, there might be multiple reasons for an idled job, check the error logs and your slurm scripts.
    - `TIME`: Time since the job started running
    - `NODES`: Number of nodes
    - `NODELIST` : Nodes name 


    #### Single job with Matlab

    ```bash
    #!/bin/bash
    #SBATCH --job-name=matlab        # job name, you give it a name you like
    #SBATCH --nodes=1                # number of nodes
    #SBATCH --ntasks=1               # number of tasks
    #SBATCH --cpus-per-task=1        # cpu per task
    #SBATCH --mem-per-cpu=1G         # memory per cpu
    #SBATCH --time=00:01:00          # max amount of time (D:HH:MM:SS)
    #SBATCH --output=slurm_logs/output-%A-%a.out   # printed output
    #SBATCH --error=slurm_logs/error-%A-%a.err     # errors

    ## you need to load matlab even if you have loaded already all the modules
    ## from the software stack 
    module load matlab

    subject=01

    ## run matlab: you pass just the name of the script without .m in my case matlab_script
    matlab -singleCompThread -nodisplay -nosplash -r "matlab_script(${subject})"
    ```


    #### Single job with python

    ```bash
    #!/bin/bash
    #SBATCH --job-name=python        # job name, you give it a name you like
    #SBATCH --nodes=1                # number of nodes
    #SBATCH --ntasks=1               # number of tasks
    #SBATCH --cpus-per-task=1        # cpu per task
    #SBATCH --mem-per-cpu=1G         # memory per cpu
    #SBATCH --time=00:01:00          # max amount of time (D:HH:MM:SS)
    #SBATCH --output=slurm_logs/output-%A-%a.out   # printed output
    #SBATCH --error=slurm_logs/error-%A-%a.err     # errors

    ## you need to load python even if you have loaded already all the modules
    ## from the software stack 
    module load python3
    subject=01

    python -u python_script ${subject}
    ```

    In case you made your python code executable you can remove the line `module load python` and call the script as follows: `./python_script ${subject}` without prepending `python -u`

=== "Array job"

    An array job allows you to run multiple jobs in parallel.</b> 

    **Use case:** It is ideal when you need to run similar tasks multiple times. For example, the same analysis for many subjects, or you need to test different parameters for the same analysis.


    In the array job the new crucial parameter is `--array` which specifies the number of jobs we would like to run. Usually you define the array of jobs indicating the range of the jobs, e.g. `1-5`

    ```bash
    #!/bin/sh


    #SBATCH --mail-user=<name.lastname>@ovgu.de
    #SBATCH --mail-type=BEGIN,END,FAIL
    #SBATCH --job-name=my_job
    #SBATCH --cpus-per-task=1
    #SBATCH --nodes=1
    #SBATCH --mem-per-cpu=1g
    #SBATCH --time=01:00:00
    #SBATCH --output=slurm_logs/output-%A-%a.out
    #SBATCH --error=slurm_logs/error-%A-%a.err
    #SBATCH --array 1-5
    ```

    #### Example of an array job

    After setting up all the parameters you can simply provide the analysis script as in the example below.
    In case you need to pass some parameters to your script, you must use the slurm variable `SLURM_ARRAY_TASK_ID` which keeps track of the job number. As we do in the example, you can use `SLURM_ARRAY_TASK_ID` as an index, `idx`, to access the single parameters specified in `parameters` and then pass them to `my_script.sh`. As you might notice, we subtract `-1` from `SLURM_ARRAY_TASK_ID`, this is because the array starts counting from 1, `--array 1-5`, but many programming languages, like bash in this case, start counting from 0.

    ```bash
    #!/bin/sh


    #SBATCH --mail-user=<name.lastname>@ovgu.de
    #SBATCH --mail-type=BEGIN,END,FAIL
    #SBATCH --job-name=my_job
    #SBATCH --cpus-per-task=1
    #SBATCH --nodes=1
    #SBATCH --mem-per-cpu=1g
    #SBATCH --time=01:00:00
    #SBATCH --output=slurm_logs/output-%A-%a.out
    #SBATCH --error=slurm_logs/error-%A-%a.err
    #SBATCH --array 0-4  ## 5 jobs

    idx=$((SLURM_ARRAY_TASK_ID))

    parameters=(0.4 0.6 0.8 1 1.2)

    ./my_script.sh ${parameters[idx]}


    ```

    Now that your array job script is ready you can run your jobs using the `sbatch` command as follows:

    ```bash
    $ sbatch my_array_job.slurm
    ```

    Check whether your jobs are actually running via the command `squeue` or alternatively with `sinfo`:

    ```bash
    $ squeue
    ```

    You should see a similar output:

    ```
                JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            217379_[6]       all fmriprep    user  PD       0:00      1 (Resources)
            217379_0       all fmriprep    user   R       1:10      1 compute05
            217379_1       all fmriprep    user   R       1:10      1 compute05
            217379_2       all fmriprep    user   R       1:10      1 compute05
            217379_3       all fmriprep    user   R       1:10      1 compute05
            217379_4       all fmriprep    user   R       1:10      1 compute05
            217379_5       all bad_job     user   IDLE    0:00      1 compute05


    ```

    Let's understand `squeue` output:

    - `JOBID`: Indentification number of your job
    - `PARTITION`: Name of partition your jobs are currently using 
    - `NAME`: Name you gave to your job
    - `USER`: Your username
    - `ST`: Status of your job:
    - `R`: running job
    - `PD`: pending job, it is waiting for resources to be freed
    - `IDLE`: idled job, something might be wrong with your scripts or your in the scirpt for submitting jobs check the error logs.
    - `TIME`: Time since the job started running
    - `NODES`: Number of nodes
    - `NODELIST` : Nodes name 



    #### Job array with Matlab

    ```bash
    #!/bin/bash

    #SBATCH --job-name=matlab        # job name, you give it a name you like
    #SBATCH --nodes=1                # number of nodes
    #SBATCH --ntasks=1               # number of tasks
    #SBATCH --cpus-per-task=1        # cpu per task
    #SBATCH --mem-per-cpu=1G         # memory per cpu
    #SBATCH --time=00:01:00          # max amount of time (D:HH:MM:SS)
    #SBATCH --output=slurm_logs/output-%A-%a.out   # printed output
    #SBATCH --error=slurm_logs/error-%A-%a.err     # errors
    #SBATCH --array 0-4

    ## you need to load matlab even if you have loaded already all the modules
    ## from the software stack 
    module load matlab

    idx=$((SLURM_ARRAY_TASK_ID))

    subjects=(01 02 03 04 05)

    ## run matlab: you pass just the name of the script without .m in my case matlab_script
    matlab -singleCompThread -nodisplay -nosplash -r "matlab_script(${subjects[idx]})"
    ```

    #### Job array with python

    ```bash
    #!/bin/bash
    #SBATCH --job-name=python        # job name, you give it a name you like
    #SBATCH --nodes=1                # number of nodes
    #SBATCH --ntasks=1               # number of tasks
    #SBATCH --cpus-per-task=1        # cpu per task
    #SBATCH --mem-per-cpu=1G         # memory per cpu
    #SBATCH --time=00:01:00          # max amount of time (D:HH:MM:SS)
    #SBATCH --output=slurm_logs/output-%A-%a.out   # printed output
    #SBATCH --error=slurm_logs/error-%A-%a.err     # errors
    #SBATCH --array 0-4

    ## you need to load python even if you have loaded already all the modules
    ## from the software stack 
    module load python3

    idx=$((SLURM_ARRAY_TASK_ID))

    subjects=(01 02 03 04 05)

    python -u python_script ${subjects[idx]}

    ```

    In case you made your python code executable you can remove the line `module load python` and call the script as follows: `./python_script ${subjects[idx]}` without prepending `python -u`

=== "Interactive job"

    This approach allows you to work interactively inside a given node.</b>   
    **Use case:** It is ideal if you need to do some testing, or you just need to run some analyses interactively. Using an interactive job you avoid to work on the head node and take precious resources that are shared among all users.

    ```bash
    $ srun --mem=1G --time=01:00:00 --pty bash -i
    ```
    - `--mem`: memory requested (it can be specified in K|G)
    - `--time`: time requested for the interactive job
    - `--pty`: `bash -i` allows you use the terminal

    In case you do not specify time and memory requirements, by default the time will be **24 hours** and the **memory 4G**

    !!! Warning "Closing a session" 
        If you close your session on the cluster the interactive job will be ended as well


## How to define the amount of resources and time for your jobs

The choice of resources for a job must be decided on a case by case basis, there is no precise rule for choosing them given that too many variables can influence the resources and time required. We will give you a few general principles which should work well anough in many instances. 

Please do not underestimate this step, your choices influence your own jobs and other users.

#### One job to rule them all:
As a general rule, when your are setting up new slurm jobs, test that everything works as intended using **only one job**. It is superfluous to say that this approach saves time for you and time and resources for other users, and importantly it reduces the amount of information to consider.

#### If you are clueless about your job resource requirements, you could adopt the following approaches:

- **Trial and error:** Set up one job using a very liberal estimate for the resources (e.g. `--mem-per-cpu`, `--time`). Once the job is over, you can use the following command to see the actual resources used and implement them in your future jobs:

```
# job-id is the id of your job
$ seff <job-id>
```

If you run `seff` when your job is still ongoing, it will give you an unreliable estimate. 

- **Hunt for information:** In case you are using a standard tool (e.g. library, toolbox) you might find in the documentation some indications about minimal resource requirements, alternatively you could ask a more expert member of your group or other users (on Mattermost) who might have used a similar tool or pipeline. Both these approaches could be good starting estimates for your jobs. Online resources like [Neurostars](https://neurostars.org/) might be very helpful as well

- **Run your computation without slurm:** If possible, run your computation (e.g. analysis, simulation) on your local machine and check the resources usage. If you assume that your computation is not particularly heavy you could run it on your `home` on Cecile and check the resource requirements using `htop`. Keep in mind that this process, if computationally heavy, might create issues for other users.

## Job restrictions and defaults

As for data storage, also jobs need some restrictions in order to guarantee a fair usage for everybody.
- **Time requested:** One job can have a **miximum duration of 24 hours**. This time duration could be extended by the clutser administrator upon reasonable request. Keep in mind that slurm will stop your job as soons as it reach the maximum time duration requested.
- **Maximum number of array jobs:** The maximum number of array jobs you can set are **10000**


## Slurm basic commands

```bash
sinfo # shows you current information about the system
```

```bash
squeue # shows the current jobs status

squeue -U <username> # it provides job status for a given user
```

```bash
sbatch <job_script># launch a job or an array job 
```

```bash
scancel <job_ID> # it cancels jobs

scancel -U <username> # all jobs belonging to that user will be cancelled
```

```bash
$ scontrol show partition compute # allows you to see partitions present and limitations 
```