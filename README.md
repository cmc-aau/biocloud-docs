# Documentation website for the BioCloud HPC at AAU
Built using [MkDocs](https://www.mkdocs.org/) with the [material theme](https://squidfunk.github.io/mkdocs-material/getting-started/).

Any changes pushed to GitHub will trigger a GitHub action that will automatically build and publish the website, so don't use `mkdocs build` or `mkdocs gh-deploy`.

This is a public repository and website, so don't publish anything sensitive!

## Testing locally
Everything is markdown so `mkdocs` is not strictly needed locally unless you want to inspect the outcome of the changes before pushing to GitHub. If so you you can use `mkdocs` and `mkdocs-material` in a few different ways:
 
 - Use the development container in VSCode
Install the "Dev Containers" extension and reopen the folder in a container (F1 -> Dev Containers: Reopen in Container).

 - Use pipenv
Install `pipenv` from `pip` or `apt`, then run `pipenv install` to install the required packages.

 - Install `mkdocs` and `mkdocs-material` locally using `pip` or `conda` or any other way you want.

Run `mkdocs serve` or `pipenv run mkdocs serve` if using `pipenv`. This will serve the site locally, any changes are updated live in the local browser. To show both editor and website side-by-side within VSCode hit F1 -> Simple browser: Show -> http://127.0.0.1:8000.

## To-do
github how to add SSH key
CLAAUDIA link to VM's?
https://github.com/NVIDIA/deepops/blob/master/docs/slurm-cluster/slurm-usage.md

make new figure based on figure 1 from here https://slurm.schedmd.com/quickstart.html