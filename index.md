# Notes for Nextflow tutorial 

These are intended to supplement the live tutorial
and may not contain sufficient detail for self-guided learning.


## STILL TODO

1. Polish the final version of this w/flow `git clone --branch nf2023_dev https://github.com/rsuchecki/nextflow-walkthrough.git`
2. Delete intermediate steps and create dedicated branches
3. Update text below, e.g. replace `mv main.nf step1.nf && git checkout bis2021_step_1` with `mv main.nf step1.nf && git checkout nf2023_step_1`


## Basics

1. ssh to petrichor
2. You will work in your scratch space i.e. `/scratch3/$USER`

```sh
cd /scratch3/$USER
```


## Hello world example

```sh
nextflow run rsuchecki/hello -revision master 
```

This executed a minimal NF workflow, but we are still running everything on the login node - no different from executing it on your own laptop.

In the next example, we'd like NF to submit each "hello" task to our cluster's Slurm scheduler as a separate batch job.
We now have accounting on the cluster so for that to work, we need to make sure the scheduler "knows" which project code 

```sh
# compare
nextflow run rsuchecki/hello -revision slurm
```

Want a closer look? `$NXF_HOME/assets/rsuchecki/hello/main.nf`.

For editing and trying things out it is better to get a local copy: 

```sh
git clone --branch slurm  https://github.com/rsuchecki/hello.git 
cd hello
# view/edit (e.g. remove the  "executor 'slurm'" directive, then run
nextflow run main.nf
```

## Example workflow 

We are developing a Nextflow DSL2 workflow (loosely) based on [this bash script](https://github.com/nathanhaigh/snakemake_template/blob/final/analysis.sh)

## Data and env modules prep

```sh
cd /scratch3/$USER
git clone --branch bis2021_step_0 https://github.com/rsuchecki/nextflow-walkthrough.git
cd nextflow-walkthrough
```

We can **either** use

```sh
nextflow run setup_data.nf 
```

**or** just copy the data previously downloaded and made available on the login node.

```sh
cp -r /tmp/NF_WORKSHOP/data ./
```

We can also get the local copy of the Singularity image we will be using. 

```sh
cp -r /tmp/NF_WORKSHOP/...simg ./singularity-images
```

Normally Nextflow would pull the image from the remote, 
but we want to avoid any issues with multiple concurrent pulls in the context of this workshop. 

## Additional files etc.

Note that some additional files are included and may be more complex than necessary for this workflow. 
This content, especially in `nextflow.config` is a mix of settings

*  designed to aid teaching
*  specific to petrichor
*  basics you may find convenient 

I hope to address the important sections, especially the [singularity execution profile](https://github.com/rsuchecki/nextflow-walkthrough/blob/bb83b4b8090f52676a93db825b4fd578e6854c5d/nextflow.config#L36-L51)


## Instructions/steps

We will be covering the following steps, 
note that there are multiple ways to go about it
so the contents on the day may differ.

**We will be editing the `main.nf` script file.**


### First input channel 

1. Use a channel factory to get FASTQ files from `data/raw_reads`
2. Apply an operator to limit the number of files e.g. using `take(params.n)`. The `n` could be set from the command line. 
3. Use `view` operator to display the names of files travelling through the channel. 
4. Use `set` operator to assign the channel to a variable name `ReadsForQcChannel`

Execute `nextflow run main.nf`


**If** the above tasks caused you some un-recoverable issues you can rename or delete your
`main.nf` and check-out a revision where the above steps have been captured,
`mv main.nf step1.nf && git checkout bis2021_step_1`

### FASTQC & MULIQC

1. Add process definitions for `FASTQC` and `MULTIQC`
2. Include `publishDir` directive in `MULTIQC` to copy results to `results/multiqc`
3. Combine them in a workflow, reading from `ReadsForQcChannel`

Execute `nextflow run main.nf -profile singularity,slurm -resume --n 4`, you may also increase `n` (32 for all files to be precessed) but we can also do that later. 

**If** the above tasks caused you some un-recoverable issues you can rename or delete your
`main.nf` and check-out a revision where the above steps have been captured,
`mv main.nf step2.nf && git checkout bis2021_step_2`

### BWA_INDEX

1. (Optional) Use a channel factory to get FASTA file from `data/references/`
2. Add process definition for `BWA_INDEX` 
3. Add `BWA_INDEX` call to workflow 

Execute `nextflow run main.nf -profile singularity,slurm -resume --n 4`

**If** the above tasks caused you some un-recoverable issues you can rename or delete your
`main.nf` and check-out a revision where the above steps have been captured,
`mv main.nf step3.nf && git checkout bis2021_step_3`

### TRIM_PE

1.  Use a channel factory to get FASTQ files as **pairs** from `data/raw_reads/`
2.  (Optional) Use a channel factory to get the adapters file `data/misc/TruSeq3-PE.fa`
3.  Add process definition for TRIM_PE (Trimmomatic)
4.  Add `TRIM_PE` call to workflow 

Execute `nextflow run main.nf -profile singularity,slurm -resume --n 4`

**If** the above tasks caused you some un-recoverable issues you can rename or delete your
`main.nf` and check-out a revision where the above steps have been captured,
`mv main.nf step4.nf && git checkout bis2021_step_4`

### BWA_ALIGN

1. Add process definition for `BWA_ALIGN` 
2. Add `BWA_ALIGN` call to workflow 
   
Execute `nextflow run main.nf -profile singularity,slurm -resume --n 4`, you may also increase `n` (32 for all files to be precessed).

**If** the above tasks caused you some un-recoverable issues you can rename or delete your
`main.nf` and check-out a revision where the above steps have been captured,
`mv main.nf step5.nf && git checkout bis2021_step_5`


### MERGE_BAMS (bonus task)

1. Add `MERGE_BAMS` process such that `samtools merge` is used (with 2 CPU threads) to merge per sample BAMs into one.
2. Add `MERGE_BAMS` call to workflow 

Execute `nextflow run main.nf -profile singularity,slurm -resume --n 16`

**If** the above tasks caused you some un-recoverable issues you can rename or delete your
`main.nf` and check-out a revision where the above steps have been captured,
`mv main.nf step6.nf && git checkout bis2021_step_6`

### Alternative syntax styles in workflow definition

```sh
mv main.nf nextsteps.nf
git checkout bis2021
```

Refer to comments in the `workflow { }` block.