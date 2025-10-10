# Jupyter notebook
[Jupyter Notebook](https://jupyter.org/) is a web application for creating and sharing computational documents. It offers a simple, streamlined, document-centric experience. This app will allow you to run a Jupyter Notebook server in a SLURM job and access it directly from your browser.

Before continuing, please first follow the guide to [getting access to the OpenOndemand web portal](../../../access/webportal.md).

## Starting the app
Start by selecting the amount of resources that you expect to use and for how long:

![jupyter resources](img/jupyter_resources.png)

If you need to use a node with specific [features](../../../slurm/jobsubmission.md#requesting-compute-nodes-with-special-features), for example if you need some fast and [local scratch space](../../../storage/local.md), or otherwise need to pass any additional options to the Slurm `sbatch` command used to launch the job, you can enter them in the "additional job options" field. Then click Launch!

## Accessing the app
When you've clicked **Launch** SLURM will immediately start finding a compute node with the requested amount of resources available, and you will see a **Queued** status. When the chosen compute node partition is not fully allocated this usually only takes a few seconds, however if it takes longer, you can check the job status and the reason why it's pending under the [Jobs](../jobqueue.md) menu, or by using [shell commands](../../../slurm/jobcontrol.md#get-job-status-info).

![jupyter queued](img/jupyter_queued.png)

When the job has been granted a resource allocation the server needs a little while to start and you will see the status change to **Starting**

![jupyter starting](img/jupyter_starting.png)

and when it finishes a button will appear to launch the notebook:

![jupyter running](img/jupyter_running.png)

You can now start working:

![jupyter inside](img/jupyter_inside.png)

???+ info "Closing the window"
    If you close the window or browser tab while something is running, it will continue to run in the background. You can always reconnect to it by clicking the **Connect to Jupyter Notebook** button again.

## Stopping the app
When you are done with your work, it's important to stop the app to free up resources for other users. You can do that by either clicking the **Quit** button inside the notebook server in the top right corner:

![jupyter quit](img/jupyter_quit.png)

or click the red **Cancel** button under **My Interactive Sessions**, see the screenshots above.

!!! warning "Always inspect and optimize efficiency for next time!"
    When the job completes, **!!!ALWAYS!!!** inspect the CPU and memory usage of the job in either the notification email received or using [these commands](../../../slurm/accounting.md#job-efficiency-summary) and adjust the next job accordingly! This is essential to avoid wasting resources which other people could used, and to reduce queue time.

## Installing Python packages for Jupyter

### From PyPI
If the exact Python version isn't important to you, you can install packages from pipy using `pip` directly from a notebook using for example `!pip install matplotlib`. However, this will use the default python version installed on the system, which cannot be changed and will be upgraded from time to time, which can result in incompatibilities and break dependencies between packages after each upgrade. To manage both the python version and python packages it's better to use conda environments.

### Using conda environments
You can use [conda environments]() to manage software and python packages and make them available for Jupyter notebooks by installing `ipykernel` into any environment as a separate [kernel](https://docs.jupyter.org/en/latest/projects/kernels.html):

```
mamba activate myproject
mamba install ipykernel
python -m ipykernel install --user --name myproject --display-name myproject
```

This will install the kernel into `${HOME}/.local/share/jupyter/kernels` with the chosen display name and make it visible from Jupyter. Select the new kernel either when creating a new notebook:

![Create new notebook with custom Jupyter kernel](img/jupyter_kernel_new.png)

or when already working on a notebook by changing kernel from the menus:

![Change Jupyter kernel](img/jupyter_kernel_change.png)

In a similar way you can also run R code in Jupyter notebooks by installing the [IRkernel](https://irkernel.github.io/) into an environment:
```
mamba activate myproject
mamba install r-irkernel
R -e "IRkernel::installspec(name = 'myproject-Rkernel', displayname = 'myproject-Rkernel')"
```

You should see the R logo when you've chosen the correct kernel:

![R kernel](img/jupyter_kernel_R.png)

If you need to install R packages, don't run any `install.packages()` commands, you must [install R packages into the conda environment](../../../software/conda.md#r-and-installing-r-packages-within-conda-environments) instead.
