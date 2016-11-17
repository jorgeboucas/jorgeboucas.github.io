---
published: false
title: daily Slurm commands
layout: post
---
```bash

# show the partitions 
sinfo 

# show information on nodes 
sinfo -N -O partitionname,nodehost,cpus,cpusload,freemem,memory

# show the queue
squeue 

# show the queue for a user
squeue -u \<user name\>

# show information on a job
scontrol show job \<job id\>

# start and interactive bash session
srun --pty bash <script>

# submit a job
sbatch -p  --cpus-per-task=\<n cpus\> --mem=\<n\>gb -t \<hours\>:\<minutes\>:\<seconds\> -o \<stdout file\> \<script\>  

# submit a job3 after job1 and job2 are successfully ready
job1=$(sbatch  2>&1 | awk '{print $(4)}')
job2=$(sbatch  2>&1 | awk '{print $(4)}')
sbatch -d afterok:${job1}:${job2} 

# cancel a job
scancel 

# cancel all jobs for user
scancel -u 
```