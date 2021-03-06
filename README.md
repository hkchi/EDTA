[![install with bioconda](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg?style=flat)](http://bioconda.github.io/recipes/edta/README.html) [![Anaconda-Server Badge](https://anaconda.org/bioconda/edta/badges/platforms.svg)](https://anaconda.org/bioconda/edta) [![Anaconda-Server Badge](https://anaconda.org/bioconda/ltr_retriever/badges/license.svg)](https://github.com/oushujun/EDTA/blob/master/LICENSE) [![Anaconda-Server Badge](https://anaconda.org/bioconda/edta/badges/version.svg)](https://anaconda.org/bioconda/edta)


# The Extensive *de novo* TE Annotator (EDTA)

## Table of Contents

   * [Introduction](#introduction)
   * [Installation](#installation)
      * [Quick installation using conda](#quick-installation-using-conda)
      * [Quick installation using Singularity](#quick-installation-using-singularity-good-for-hpc-users)
      * [Quick installation using Docker](#quick-installation-using-docker-good-for-root-users)
      * [Step by step installation using conda ](#step-by-step-installation-using-conda)
   * [Inputs](#inputs)
   * [Outputs](#outputs)
   * [EDTA usage](#edta-usage)
      * [From head to toe](#from-head-to-toe)
      * [Divide and conquer](#divide-and-conquer)
   * [Benchmarking usage](#benchmarking)
   * [Citations](#citations)
   * [Other resources](#other-resources)
   * [Issues](#issues)
   * [Acknowledgements](#acknowledgements)


## Introduction
This package is developed for automated whole-genome *de-novo* TE annotation and benchmarking the annotation performance of TE libraries.

The EDTA package was designed to filter out false discoveries in raw TE candidates and generate a high-quality non-redundant TE library for whole-genome TE annotations. Selection of initial search programs were based on benckmarkings on the annotation performance using a manually curated TE library in the rice genome.

<img width="600" alt="The EDTA workflow" src="https://github.com/oushujun/EDTA/blob/master/development/EDTA%20workflow.png?raw=true">

For benchmarking of a testing TE library, I have provided the curated TE annotation (v6.9.5) for the rice genome (TIGR7/MSU7 version). You may use the `lib-test.pl` script to compare the annotation performance of your method/library to the methods we have tested (usage shown below).


## Installation

There are four ways to install EDTA. Please choose one.

### Quick installation using conda (Linux64)
    conda install -c bioconda edta

### Quick installation using [Singularity](https://sylabs.io/docs/) (good for HPC users)
Installation:

    singularity build --sandbox EDTA.sif docker://kapeel/edta

Usage:

    singularity exec {path}/EDTA.sif /EDTA/EDTA.pl --genome genome.fa [other parameters]

	{path} is the path you build the EDTA singularity image

### Quick installation using [Docker](https://www.docker.com/) (good for root users)
Installation:

    docker pull kapeel/edta

Usage:

    docker run kapeel/edta --genome genome.fa [other parameters]

### Step by step installation using conda
    conda create -n EDTA
    conda activate EDTA
    conda config --env --add channels anaconda --add channels conda-forge --add channels bioconda
    conda install -n EDTA -y cd-hit repeatmodeler muscle mdust blast openjdk perl perl-text-soundex multiprocess regex tensorflow>=1.14.0 keras>=2.2.4 scikit-learn>=0.19.0 biopython pandas glob2 python>=3.6 tesorter genericrepeatfinder genometools-genometools ltr_retriever ltr_finder
    git clone https://github.com/oushujun/EDTA
    ./EDTA/EDTA.pl


## Inputs
Required: The genome file [FASTA]. Please make sure sequence names are short (<=15 characters) and simple (i.e, letters, numbers, and underscore).

Optional: 
1. Coding sequence of the species or closely related species [FASTA]. This file helps to purge gene sequences in the TE library.
2. Known gene position of this version of the genome assembly [BED]. Coordinates specified in this file will be whitelisted from TE annotation to avoid over-masking.
3. Curated TE library of the species [FASTA]. This file is trusted 100%. Please make sure it's curated. If you only have a couple of curated sequences, that's fine. It doesn't need to be complete.


## Outputs
Expected: A non-redundant TE library: $genome.mod.EDTA.TElib.fa. The curated library is included in this file if provided. TEs are classified into the superfamily level and using the three-letter naming system reported in [Wicker et al. (2007)](https://www.nature.com/articles/nrg2165). Each sequence can be considered as a TE family.

Optional:
1. Novel TE families: $genome.mod.EDTA.TElib.novel.fa. This file contains TE sequences that are not included in the curated library (`--curatedlib` required).
2. Whole-genome TE annotation: $genome.mod.EDTA.TEanno.gff. This file contains both structurally intact and fragmented TE annotations (`--anno 1` required).
3. Summary of whole-genome TE annotation: $genome.mod.EDTA.TEanno.sum (`--anno 1` required).
4. Low-threshold TE masking: $genome.mod.MAKER.masked. This is a genome file with only long TEs (>=1 kb) being masked. You may use this for de novo gene annotations. Annotated gene models should contain TEs and need further filtering (`--anno 1` required).
5. Annotation inconsistency for simple TEs; $genome.mod.EDTA.TE.fa.stat.redun.sum (`--evaluate 1` required).
6. Annotation inconsistency for nested TEs: $genome.mod.EDTA.TE.fa.stat.nested.sum (`--evaluate 1` required).
7. Oveall annotation inconsistency: $genome.mod.EDTA.TE.fa.stat.all.sum (`--evaluate 1` required).


## EDTA Usage

### From head to toe
*You got a genome and you want to get a high-quality TE annotation:*

    perl EDTA.pl [options]
      --genome	[File]	The genome FASTA
      --species [Rice|Maize|others]	Specify the species for identification of TIR candidates. Default: others
      --step	[all|filter|final|anno] Specify which steps you want to run EDTA.
				all: run the entire pipeline (default)
				filter: start from raw TEs to the end.
				final: start from filtered TEs to finalizing the run.
				anno: perform whole-genome annotation/analysis after TE library construction.
      --overwrite	[0|1]	If previous results are found, decide to overwrite (1, rerun) or not (0, default).
      --cds	[File]	Provide a FASTA file containing the coding sequence (no introns, UTRs, nor TEs) of this genome or its close relative.
      --curatedlib	[file]	Provided a curated library to keep consistant naming and classification for known TEs.
				All TEs in this file will be trusted 100%, so please ONLY provide MANUALLY CURATED ones here.
				This option is not mandatory. It's totally OK if no file is provided (default).
      --sensitive	[0|1]	Use RepeatModeler to identify remaining TEs (1) or not (0, default).
				This step is very slow and MAY help to recover some TEs.
      --anno	[0|1]	Perform (1) or not perform (0, default) whole-genome TE annotation after TE library construction.
      --rmout	[File]	Provide your own homology-based TE annotation instead of using the EDTA library for masking. File is in RepeatMasker .out format. This file will be merged with the structural-based TE annotation. (-anno 1 required). Default: use the EDTA library for annotation.
      --evaluate	[0|1]	Evaluate (1) classification consistency of the TE annotation. (-anno 1 required). Default: 0.
				This step is slow and does not affect the annotation result.
      --exclude	[File]	Exclude bed format regions from TE annotation. Default: undef. (-anno 1 required).
      --threads|-t	[int]	Number of theads to run this script (default: 4)
      --help|-h	Display this help info

### Divide and conquer
*Identify intact elements of a paticular TE type*:

1.Get raw libraries from a genome (specify `-type ltr|tir|helitron` in different runs)

    perl EDTA_raw.pl [options]
      --genome	[File]	The genome FASTA
      --species [Rice|Maize|others]	Specify the species for identification of TIR candidates. Default: others
      --type	[ltr|tir|helitron|all]	Specify which type of raw TE candidates you want to get. Default: all
      --overwrite	[0|1]	If previous results are found, decide to overwrite (1, rerun) or not (0, default).
      --threads|-t	[int]	Number of theads to run this script
      --help|-h	Display this help info

2.Finish the rest of the EDTA analysis (specify `-overwrite 0` and it will automatically pick up existing results in the work folder)

    perl EDTA.pl --overwrite 0 [options]


## Benchmarking
If you developed a new TE method/got a TE library and want to compare it's annotation performance to the methods we have tested, you can:

1.annotate the rice genome with your test library:

    RepeatMasker -pa 36 -q -no_is -norna -nolow -div 40 -lib custom.TE.lib.fasta -cutoff 225 rice_genome.fasta

2.Test the annotation performance of a particular TE category.

    perl lib-test.pl -genome genome.fasta -std genome.stdlib.RM.out -tst genome.testlib.RM.out -cat [options]
        -genome	[file]	FASTA format genome sequence
        -std	[file]	RepeatMasker .out file of the standard library
        -tst	[file]	RepeatMasker .out file of the test library
        -cat	[string]	Testing TE category. Use one of LTR|nonLTR|LINE|SINE|TIR|MITE|Helitron|Total|Classified
        -N	[0|1]	Include Ns in total length of the genome. Defaule: 0 (not include Ns).
        -unknown	[0|1]	Include unknown annotations to the testing category. This should be used when
                        the test library has no classification and you assume they all belong to the
                        target category specified by -cat. Default: 0 (not include unknowns)

eg.

    perl lib-test.pl -genome rice_genome.fasta -std ./EDTA/database/Rice_MSU7.fasta.std6.9.5.out -tst rice_genome.fasta.test.out -cat LTR

## Citations
Please cite our paper if you find EDTA is useful:

Ou S., Su W., Liao Y., Chougule K., Agda J. R. A., Hellinga A. J., Lugo C. S. B., Elliott T. A., Ware D., Peterson T., Jiang N.✉, Hirsch C. N.✉ and Hufford M. B.✉ (2019). Benchmarking Transposable Element Annotation Methods for Creation of a Streamlined, Comprehensive Pipeline. [Genome Biol. 20(1): 275.](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1905-y)

Please also cite the software packages that were used in EDTA, listed in the [EDTA/bin](https://github.com/oushujun/EDTA/tree/master/bin) directory.

## Other resources
You may download the [rice genome here](http://rice.plantbiology.msu.edu/pub/data/Eukaryotic_Projects/o_sativa/annotation_dbs/pseudomolecules/version_7.0/all.dir/all.con).

## Issues
If you have any issues with installation and usage, please check if similar issues have been reported in [Issues](https://github.com/oushujun/EDTA/issues) or open a new issue. If you are (looking for) happy users, please read or write successful cases [here](https://github.com/oushujun/EDTA/issues/15).

## Acknowledgements
I want to thank [Jacques Dainat](https://github.com/Juke34) for contribution of the EDTA conda recipe as well as improving the codes. I also want to thank [Qiushi Li](https://github.com/QiushiLi), [Zhigui Bao](https://github.com/baozg), [Philipp Bayer](https://github.com/philippbayer), [Nick Carleson](https://github.com/Neato-Nick), [@aderzelle](https://github.com/aderzelle), [Shanzhen Liu](https://github.com/liu3zhenlab), [Zhougeng Xu](https://github.com/xuzhougeng), [Shun Wang](https://github.com/wangshun1121), and many more others for testing, debugging, and improving the EDTA pipeline.
