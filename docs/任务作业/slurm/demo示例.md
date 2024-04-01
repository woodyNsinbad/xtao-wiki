# sbatch 示例
```
#!/bin/bash
#SBATCH --job-name=my_job                # Job name
#SBATCH --output=my_job.out             # Standard output file name
#SBATCH --error=my_job.err              # Standard error file name
#SBATCH --partition=compute             # Partition name (must)
#SBATCH --account=my_account            # Accounting account (must)
#SBATCH --nodes=1                       # Request 1 node
#SBATCH --ntasks=1                      # Request 1 task
#SBATCH --cpus-per-task=1               # Allocate 1 CPU cores per task （default）
#SBATCH --mem=4G                        # Request 4GB memory （default）
#SBATCH --time=1:00:00                  # Expected run time is 1 hour（options）
#SBATCH --mail-user=your_email@example.com   # Specify the email address to receive emails（options）
#SBATCH --mail-type=ALL                 # Receive all types of email notifications（options）
#SBATCH --reservation=my_reservation    # Reservation name（options）
#SBATCH --dependency=afterok:1234       # Dependency job ID（options）
#SBATCH --chdir=/path/to/working/directory    # Change working directory（options）
#SBATCH --export=ALL                    # Export all environment variables（options）
#SBATCH --nice=10                       # Set job priority
#SBATCH --comment="My comment"          # Add a comment to the job（options）
#SBATCH --gres=gpu:1                    # Request 1 GPU（options）

# Load required modules (if any)
module load my_module

# Additional environment setup (if needed)
export PATH=$PATH:/path/to/additional/binaries

# Run any pre-job setup commands (if needed)
echo "Preparing for job execution..."

# Execute the main job command
echo "Executing main job command..."
my_command arg1 arg2

# Run any post-job cleanup commands (if needed)
echo "Cleaning up after job execution..."

# End of script
echo "Job execution completed."
# #SBATCH --exclusive                     # Request exclusive node access
```