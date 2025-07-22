# Multi-step jobs

Many steps in a complex workflow will only run on a single thread regardless of whether you've asked for more. This leads to a waste of resources. You can submit separate jobs by writing down the commands in separate shell scripts, then submit them as individual jobs using sbatch with different resource requirements:

**`launchscript.sh`**
```
#!/bin/bash

set -euo pipefail

# Submit the first job step and capture its job ID
step1_jobid=$(sbatch step1_script.sh | awk '{print $4}')

# Submit the second job, ensuring it runs only after the first job completes successfully
step2_jobid=$(sbatch --dependency=afterok:$step1_jobid step2_script.sh | awk '{print $4}')

# Submit the third/last job, ensuring it runs only after the second job completes successfully
sbatch --dependency=afterok:$step2_jobid step3_script.sh
```

Types of Dependencies:
 - `afterok`: The dependent job runs if the first job completes successfully (exit code 0).
 - `afternotok`: The dependent job runs if the first job fails (non-zero exit code).
 - `afterany`: The dependent job runs after the first job completes, regardless of success or failure.
 - `after:<job_id>`: The dependent job starts when the first job begins execution.

In this case, using `--dependency=afterok` ensures that the second job will only start if the first job finishes without errors.

Submit using bash not sbatch.

This can be repeated as many times as necessary. Any arguments can be passed on to the shell scripts in the exact same way as when invoking them using `bash script -i "some option" -o "some other option"` as usual.