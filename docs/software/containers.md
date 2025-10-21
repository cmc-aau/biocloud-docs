# Containers
[Containers](https://www.docker.com/resources/what-container/) provide a convenient and portable way to package and run applications in a completely isolated and self-contained environment, making it easy to manage dependencies and ensure complete reproducibility and portability. Compared to [conda environments](conda.md) or software modules containers are always based on a base operating system image, usually Linux, ensuring that even the operating system is under control. Once a container is built and working as intended, it will run exactly the same forever, whereever, and is therefore the best way to bundle and distribute production-level workflows. By containerizing the application platform and its dependencies, differences in OS distributions and underlying infrastructure are abstracted away completely. Linux containers allow users to:

 - Use software with complicated dependencies and environment requirements
 - Run an application container from the Sylabs Container Library, Docker Hub, or from self-made images from the GitHub container registry
 - Use a package manager (like apt or yum) to install software without changing anything on the host system or require elevated privileges
 - Run an application that was built for a different distribution of Linux than the host OS
 - Run the latest released software built for newer Linux OS versions than that present on HPC systems
 - Archive an analysis for long-term reproducibility and/or publication

## Singularity/Apptainer
[Singularity/Apptainer](https://apptainer.org/docs/user/main/index.html) is a tool for running software containers on HPC systems, but is made specifically with scientific computing in mind. Singularity allows running Docker and any other OCI-based container natively and is a replacement for Docker on HPC systems. Singularity has a few extra advantages:

 - Security: a user in the container is the same user with the same privileges/permissions as the one running the container, so no privilege escalation is possible
 - Ease of deployment: no daemon running as root on each node, a container is simply an executable
 - Ability to run workflows that require MPI and GPU support

### Building container images
Building your own containers using native `apptainer build` or `docker build` commands are not possible on BioCloud due to the specific user authentication and security configuration used. Instead you can build containers using either [cotainr](https://cotainr.readthedocs.io/en/stable/user_guide) or [enroot](https://github.com/nvidia/enroot). Other alternative ways for building containers are:

 - Using your own system (laptop/workstation) where you have root/elevated privileges to install Singularity or Docker and build containers, then transfer the container image file(s) to the BioCloud or publish it to a public container registry
 - Use a free cloud container build service like [https://cloud.sylabs.io](https://cloud.sylabs.io) or [https://hub.docker.com/](https://hub.docker.com/)
 - Publish a `Dockerfile` to a GitHub repository and use GitHub actions to build and publish the container to the GitHub container registry

#### Bundling conda environments in a container
cotainr is perhaps the easiest way to [bundle conda environments inside an apptainer container](https://cotainr.readthedocs.io/en/stable/user_guide/conda_env.html) by simply giving a path to an environment yaml file (see the [conda page](conda.md#creating-an-environment)) and choosing a base image from for example [docker hub](https://hub.docker.com/):

```
cotainr build --base-image docker://ubuntu:22.04 --conda-env condaenv.yml myproject.sif
```

This will produce a single file anyone can use anywhere, regardless of platform and local differences in setup, etc.

### Pre-built container images
Usually it's not needed to build a container yourself unless you want to customize things in detail, since there are plenty of pre-built images already available that work straight of the box. For bioinformatic software the community-driven project [biocontainers.pro](https://biocontainers.pro/) should have anything you need, and if not - you can contribute! If you need a container with multiple tools installed see [multi-package containers](https://github.com/BioContainers/multi-package-containers).

### Running a container
```
# pull a container
$ apptainer pull ubuntu_22.04.sif docker://ubuntu:22.04

# run a container with default options
$ apptainer run ubuntu_22.04.sif yourcommand --someoption somefile

# start an interactive shell within a container
$ apptainer shell ubuntu_22.04.sif
```

### Binding (mounting) folders from the host to the container
You almost always need to bind/mount a folder from the host machine to the container, so that it's available inside the container for input/output to the particular tool you need to use. With Singularity/Apptainer the `/tmp` folder, the current folder, and your home folder are always mounted by default. To mount additional folders use `-B`, for example:
```
# Bind with the same path inside the container as on the host
apptainer run -B /databases ubuntu_22.04.sif yourcommand --someoption somefile

# Bind at a different path inside the container
apptainer run -B /databases:/some/other/path/databases ubuntu_22.04.sif yourcommand --someoption somefile

# Binding multiple folders at once
apptainer run -B /databases -B /raw_data -B /projects ubuntu_22.04.sif yourcommand --someoption somefile

# You can also specify mounts setting the APPTAINER_BIND variable before running a container
export APPTAINER_BIND="/raw_data,/databases,/home,/projects"
apptainer run ubuntu_22.04.sif yourcommand --someoption somefile
```

For additional guidance see the [Apptainer usage guide](https://apptainer.org/docs/user/main/index.html). If you need to use a GPU with apptainer use the `--nvccli` flag, not `--nv`.

## Docker containers
Docker itself is not supported directly for non-admin users due to security and compatibility issues with our user authentication mechanism, but you can instead just run them through apptainer by prepending `docker://` to the container path, see [this page](https://apptainer.org/docs/user/main/docker_and_oci.html).
