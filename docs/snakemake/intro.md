# Introduction to Snakemake for HPC Users

![snake-img](media/bashvsnake.png)

## What is Snakemake?
<p>Author: ChatGPT  | Reviewed by: <a href="mailto:semoda@bio.aau.dk">Sebastian Mølvang Dall</a>    |   Review Date: 18-12-2023</p>
Snakemake is a workflow management system that simplifies the definition and execution of complex data analysis pipelines. It's an open-source tool, designed to scale from simple to complex workflows, providing both flexibility and reproducibility. Snakemake workflows are defined in a human-readable, Python-based language, making them both accessible and powerful.

## Key Features of Snakemake

1. **Reproducibility and Transparency:** Snakemake ensures that every step of your workflow is reproducible, documenting how each output file is created and what inputs and parameters are used.
2. **Scalability:** It is designed to scale from local execution on a laptop to cluster, grid, or cloud environments, seamlessly integrating with SLURM and other job schedulers.
3. **Flexibility:** Users can integrate code written in any language (Python, R, Shell, etc.) and can also encapsulate steps in software containers like Conda, Docker, or Singularity for consistent environments.
4. **Dependency Resolution:** Snakemake automatically resolves dependencies between steps and executes them in the correct order. It also allows for parallel execution of independent steps.
5. **Efficiency:** It checks the necessity of re-execution and only reruns steps when their inputs, code, or parameters have changed.

## Integrating Snakemake with SLURM

Snakemake and SLURM are a powerful combination for HPC environments. Snakemake's ability to submit jobs to a SLURM scheduler allows for efficient resource utilization and management of complex workflows. It automatically translates the workflow steps into SLURM jobs, managing the submission process, and monitoring job completion.

## The purpose of this tutorial

Snakemake offers great documentation at [snakemake.readthedocs.io](https://snakemake.readthedocs.io/en/stable/) for getting started, however the purpose of this tutorial is to focus on establishing snakemake workflows and preparing them for running on biocloud SLURM system. As with bash scripts and other programming languages, there are many ways to accomplish the same task, and this will be no exception. The goal is to provide a basic understanding of snakemake and how to use it to create workflows that can be run on biocloud.
