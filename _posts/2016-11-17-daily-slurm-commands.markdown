---
published: true
title: daily Slurm commands
layout: post
---
```bash
# show the partitions 
sinfo 

# show information on nodes 
sinfo -N -O partitionname,nodehost,cpus,cpusload,freemem,memory

# check information on a partition
scontrol show part <partition name>

# show the queue
squeue 

# show the queue for a user
squeue -u <user name>

# show information on a job
scontrol show job <job id>

# start and interactive bash session
srun --pty bash

# submit a job
sbatch -p  --cpus-per-task=<n cpus> --mem=<n>gb -t <hours>:<minutes>:<seconds> -o <stdout file> <script>  

# submit a job3 after job1 and job2 are successfully ready
job1=$(sbatch --parsable <script1>)
job2=$(sbatch --parsable <script2>)
sbatch -d afterok:${job1}:${job2} <script3>

# attach to a running job and run a command
srun --jodib <JOBID> --pty <command>

# cancel a job
scancel <job id>

# cancel all jobs for user
scancel -u <user name>
```
