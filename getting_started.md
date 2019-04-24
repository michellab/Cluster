# Getting started

### Introduction

The section9 computing cluster uses the [slurm scheduler](https://slurm.schedmd.com/documentation.html) to manage job submissions.  Slurm stands for Simple Linux Utility for Resource Management and was mainly chosen because it is still actively being developed and has good support for GPUs.

If you are having any problems, something isn't working or you would like a package installed, email Antonia: [antonia.mey@ed.ac.uk](mailto:antonia.mey@ed.ac.uk) or Jenke: [jscheen@ed.ac.uk](mailto:jscheen@ed.ac.uk).

## Basics

Log in to the head node to run your jobs:
```
ssh username@section9.chem.ed.ac.uk
```

Load the slurm scheduler:

<sup>*(add this to your section9 bashrc for ease)*</sup>

```
module load slurm
```

Slurm has a set of simple commands. The most useful ones are the following:

<sub>*(each slurm command has its own manual which can be accessed by running "man "command"")*</sub>

```
sbatch - submit a job script (which must be executable - this can be achieved with chmod u+x myscript.sh), e.g. sbatch myscript.sh

scancel - cancel a job, e.g. scancel 123 to cancel the running job with ID "123"
scancel -u username - cancel all "username"'s jobs (permissions are set so that you can only cancel your own)

sinfo - show available partitions and nodes, e.g. sinfo -l
squeue - show jobs in queue, e.g. squeue -u myusername
```

When submitting a job, the submission file will typically include a header with job variables called by lines starting with "#SBATCH" as outlined below. See the sbatch manual for a more complete outline.

```
#SBATCH -n 1 			# Number of cores
#SBATCH -N 1 			# Ensure that all cores are on one machine
#SBATCH -t 0-00:05 		# Runtime in D-HH:MM
#SBATCH -p serial 		# Partition to submit to e.g serial, GTX, Tesla
#SBATCH --mem=100 		# Memory pool for all cores (see also --mem-per-cpu)
#SBATCH -o hostname.out # File to which STDOUT will be written
#SBATCH -e hostname.err # File to which STDERR will be written
#SBATCH --job-name=name # Job will be named "name" 
```



## Simple Serial CPU job submission script

A simple example of a serial CPU script using Sire looks like this:

```
#!/bin/bash
#SBATCH -o somd_CPU.out
#SBATCH -p serial
#SBATCH -n 1
#SBATCH -t 24:00:00

source /etc/profile.d/module.sh 	# makes sure that the modules are available to the job environment
module load cuda/7.5 				#load the required modules for sire, i.e. openMM and 										cuda
module load sire/17.1.0_no_avx
module load openmm/6.3


srun somd-freenrg -C input.cfg -l 0.00 -p Reference #srun is needed for slurm.
```

## CPU Job Array submission script

Slurm also has the capability of using job arrays. The simplest example of using job arrays is by adding a single line to the submission script, e.g.:

```
#!/bin/bash
#...						#use all the desired #SBATCH options here

#SBATCH --array=1-100 		#This will submit a task array using array ID's from 1-100 and 								then run in each directory the program
cd dir.$SLURM_ARRAY_TASK_ID
srun program
```

 A more concrete example for this using different values for lambda:

```
#!/bin/bash
#SBATCH -o somd-array-cpu.%a.out
#SBATCH -p serial
#SBATCH -n 1
#SBATCH --time 24:00:00
#SBATCH --array=0-200

source /etc/profile.d/module.sh
module load cuda/7.5
module load sire/17.1.0_no_avx
module load openmm/6.3


lamvals=(0.000 0.100 0.200 0.300 0.400 0.500 0.600 0.700 0.800 0.900 1.000 )
lam=${lamvals[SLURM_ARRAY_TASK_ID]}

echo "lambda is: " $lam

mkdir lambda-$lam
cd lambda-$lam
srun somd-freenrg -C input.cfg -l $lam -p Reference
cd ..
```

## Simple GPU job submission script

In case you want to parallelise your computations to GPUs instead of computing on CPUs, this can be easily communicated to slurm:

```
#!/bin/bash
#SBATCH -o somd_GPU.out
#SBATCH -p GPU 							#now choose the GPU partition (e.g. GTX, Tesla)
#SBATCH -n 1
#SBATCH --gres=gpu:1 					#requests one GPU
#SBATCH --time 24:00:00

source /etc/profile.d/module.sh
module load cuda/7.0
module load openmm/6.3
module load sire/dev

echo "CUDA DEVICES:" $CUDA_VISIBLE_DEVICES

srun somd-freenrg -C input.cfg -l 0.00 -p CUDA
```

## Job chaining

Sometimes it is useful to start a series of jobs were the new job depends on the successful completion of a previous job. You could for instance start a simulation only when another has finished, or repeat the simulation if it has failed.

With slurm this can be achieved using dependencies. The man page gives the following information on dependencies:

```
-d, --dependency=<dependency_list>
	  Defer the start of this job until the specified  dependencies  have  been  
	  satisfied  completed.   <dependency_list>  is  of  the  form
	  <type:job_id[:job_id][,type:job_id[:job_id]]>.   
	  Many  jobs  can  share the same dependency and these jobs may even belong 
	  to different users. The  value may be changed after job submission using 
	  the scontrol command.

	  after:job_id[:jobid...]
			 This job can begin execution after the specified jobs have begun 
			 execution.

	  afterany:job_id[:jobid...]
			 This job can begin execution after the specified jobs have terminated.

	  afternotok:job_id[:jobid...]
			 This job can begin execution after the specified jobs have terminated
			 in some failed state (non-zero exit  code,  node  failure, timed out, etc).

	  afterok:job_id[:jobid...]
			 This  job can begin execution after the specified jobs have successfully 
			 executed (ran to completion with an exit code of zero).

	  expand:job_id
			 Resources allocated to this job should be used to expand the specified 
			 job.  The job to expand must share the same QOS  (Quality of Service) 
			 and partition.  Gang scheduling of resources in the partition is also 
			 not supported.

	  singleton
			 This job can begin execution after any previously launched jobs sharing 
			 the same job name and user have terminated.
```

A simple job submission using the dependency option can look like this:

```
user@node009:/home/user$ sbatch -p serial script.sh
Submitted batch job 36805
[user@node009:/home/user$ sbatch --dependency=afterany:36805 -p serial script.sh 
Submitted batch job 36806
[user@node009:/home/user/$ sbatch --dependency=afterok:36805 -p serial script.sh 
Submitted batch job 36807
```