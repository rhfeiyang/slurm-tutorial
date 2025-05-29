# SLURM Tutorial & Quick Reference

A concise guide to using SLURM (Simple Linux Utility for Resource Management) for high-performance computing clusters.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Common Commands](#common-commands)
- [Resource Allocation](#resource-allocation)
- [Batch Job Submission](#batch-job-submission)
- [Best Practices](#best-practices)

## Prerequisites

Before using SLURM commands, make sure to:
1. Connect to your cluster's login node (usually via SSH)
2. Have proper account permissions set up by your system administrator
3. Know your cluster's partition names and available resources

```bash
ssh username@your-cluster-login-node
```

## Common Commands

### Check Job Status
```bash
squeue                    # View all running jobs
squeue -u username        # View your jobs only
squeue -j jobid           # View specific job
```

### Check Cluster Resources
```bash
sinfo                     # Basic cluster information
sinfo -Nel                # Detailed node information
```

### Custom Resource View
For a more detailed and customed resource overview (by [@rhfeiyang](https://rhfeiyang.top)):
```bash
sinfo -O Nodehost:13,partition:.15,statecompact:.7,Gres:.30,GresUsed:.47,freemem:.10,memory:.10,cpusstate:.15
```

### Job Management
```bash
scancel jobid             # Cancel a job
scancel -u username       # Cancel all your jobs
scontrol show job jobid   # Show job details
```

### Modify Running Jobs
```bash
scontrol update JobId=jobid TimeLimit=10:00:00    # Modify time limit
scontrol update JobId=jobid Partition=newpartition # Change partition
```

## Resource Allocation

### Interactive Jobs (for debugging)

Basic CPU-only allocation:
```bash
salloc -N 1 -c 4 -t 2:00:00 --mem=8G
```

GPU allocation (adjust GPU type and count as needed):
```bash
# Single GPU
salloc -N 1 -c 8 --gres=gpu:1 --mem=32G -t 4:00:00

# Specific GPU type (check with `sinfo -O Gres:13`)
salloc -N 1 -c 8 --gres=gpu:v100:1 --mem=32G -t 4:00:00
salloc -N 1 -c 8 --gres=gpu:a100:1 --mem=64G -t 4:00:00
```

### Parameter Explanations
- `-N 1`: Number of nodes
- `-c 8`: Number of CPU cores
- `--gres=gpu:1`: Generic resource (1 GPU)
- `--mem=32G`: Memory allocation
- `-t 4:00:00`: Time limit (4 hours)
- `-p partition_name`: Specify partition
- `-A account_name`: Specify account

## Batch Job Submission

### Create a SLURM Script

Create a file with `.slurm` or `.sh` extension:

```bash
#!/bin/bash
#SBATCH --job-name=my_job          # Job name
#SBATCH --partition=gpu            # Partition name
#SBATCH --account=your_account     # Account name
#SBATCH --requeue                  # Requeue the job if it fails/preempted
#SBATCH --nodes=1                  # Number of nodes
#SBATCH --cpus-per-task=8          # CPU cores per task
#SBATCH --mem=32G                  # Memory per node
#SBATCH --time=05:00:00            # Time limit (5 hours)
#SBATCH --gres=gpu:1               # GPU allocation
#SBATCH --output=job_%j.out        # Standard output (%j = job ID)
#SBATCH --error=job_%j.err         # Standard error
#SBATCH --mail-type=END,FAIL       # Email notifications
#SBATCH --mail-user=your@email.com # Email address


# Navigate to working directory
cd /path/to/your/project

# Activate environment, change to your own shell and environment name
source /home/user/.bashrc
conda activate your_env

# Run your commands
python your_script.py
echo "Job completed at $(date)"
```

### Submit the Job
```bash
sbatch your_script.slurm
```

### Common SBATCH Options

| Option | Description | Example |
|--------|-------------|---------|
| `--job-name` | Job name | `--job-name=training` |
| `--partition` | Queue/partition | `--partition=gpu` |
| `--nodes` | Number of nodes | `--nodes=2` |
| `--ntasks` | Number of tasks | `--ntasks=4` |
| `--cpus-per-task` | CPUs per task | `--cpus-per-task=8` |
| `--mem` | Memory per node | `--mem=64G` |
| `--mem-per-cpu` | Memory per CPU | `--mem-per-cpu=4G` |
| `--time` | Time limit | `--time=12:00:00` |
| `--gres` | Generic resources | `--gres=gpu:2` |
| `--exclusive` | Exclusive node use | `--exclusive` |
| `--array` | Job arrays | `--array=1-10` |

## Best Practices

### 1. Resource Planning
- Request only the resources you need
- Use appropriate time limits
- Monitor your jobs with `squeue` and `sacct`

### 2. Job Management
- Use meaningful job names
- Redirect output to files
- Set up email notifications for long jobs

### 3. Debugging
- Start with interactive jobs for testing
- Use small test datasets first
- Check your scripts locally when possible

### 4. Efficiency Tips
- Use job arrays for parameter sweeps
- Checkpoint long-running jobs
- Clean up temporary files
- Use `--requeue` for fault tolerance

### 5. Common Troubleshooting
```bash
# Check job details
scontrol show job jobid

# View job history
sacct -j jobid --format=JobID,JobName,State,ExitCode,Start,End

# Check node details
scontrol show node nodename

# View cluster usage
sshare -A account_name
```

## Additional Resources

- [Official SLURM Documentation](https://slurm.schedmd.com/)
- Check your cluster's specific documentation for:
  - Available partitions and QoS settings
  - Installed software modules
  - Local policies and limits
  - Cluster-specific examples

---

> **Note**: Replace placeholder values (account names, partition names, email addresses, etc.) with your cluster-specific information. Contact your system administrator for cluster-specific configurations.
