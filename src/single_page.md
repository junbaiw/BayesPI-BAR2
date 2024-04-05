
# BayesPI-BAR2: a new Python package for predicting functional non-coding mutations in cancer patient cohorts
<img src="dnk.jpg" alt="This work was supported by the Norwegian Cancer Society" title="This work was supported by the Norwegian Cancer Society" width="150" align="middle"/>  <img src="OUS_logo_eng.jpg" align="middle" alt="Oslo University Hospital" title="Oslo University Hospital" hspace="20" width="150" />  <img src="helse-sor-ost.jpg" alt="This work was supported by the South-Eastern Norway Regional Health Authority" title="This work was supported by the South-Eastern Norway Regional Health Authority" align="middle" width="150"/>

## Authors 
[Kirill Batmanov](mailto:batmanov.kn@gmail.com)^1^, Jan Delabie^2^, and [Junbai Wang](mailto:junbai.wang@rr-research.no)^1^

^1^Department of Pathology, Norwegian Radium Hospital, PO Box 4953 Nydalen, 0424 Oslo, Norway

^2^Department of Pathology, University Health Network, Toronto, Ontario, Canada


## Introduction 
**BayesPI-BAR2** [4] is a package designed to predict how non-coding somatic mutations in cancer samples affect protein-DNA binding at the mutated place. Changes in binding of transcription factors to mutated regulatory sequences can lead to disrupted gene regulation, which may promote tumorigenesis. **BayesPI-BAR2** takes into account the possibility for several nearby mutations to affect binding of the same protein. The predicted effects are tested for significance in the given patient cohort, and only those that appear in patient samples more frequently than expected by chance are reported.


## Getting started 

**BayesPI-BAR2** is written in Python 2. It includes our **BayesPI2** [1][2] software in binary form, which is available for Linux and OS X operating systems. Here is the full list of dependencies:

-   Python 2 interpreter: [https://www.python.org/downloads/](https://www.python.org/downloads/)
-   SciPy library: [https://www.scipy.org/](https://www.scipy.org/)
-   matplotlib, if the plot making function is used: [https://matplotlib.org/](https://matplotlib.org/)
-   bedtools: [http://bedtools.readthedocs.io/en/latest/](http://bedtools.readthedocs.io/en/latest/)

You can use the `pip install scipy matplotlib` command to install the Python libraries.

==The **BayesPI-BAR2** package can be downloaded [here](bayespi_bar2_package.tgz).==

To test the basic functionality, go to the `demo/melanoma_small` folder and run the command `python melanoma_small_pipeline.py` . After downloading the reference human genome, the test pipeline should complete without errors in a few minutes and produce the result file, `data/skin_cancer_small/out/foreground/block_0_5_1295228_1295253/result.tsv` with several ETS factors mentioned in it.

## Package contents

The package has four subfolders:

-   `bin`: the binaries of **BayesPI2**
-   `demo`: the two example pipelines, `melanoma_small` for a quick test and `melanoma_full` for a complete application
-   `data`: the folder from which the demos take their input data and where they put their outputs
-   `python`: the folder with the package Python source code

The main package is a set of command line tools residing in the `python` folder. Run `python <tool_name.py> --help` command to see the full usage information for a particular tool. The purpose of each tool and the data involved in the processing are described below:

-   `differential_expression.py`: computes differentially expressed genes between two groups of samples, usually the patient samples and normal tissue samples. This implements a relatively simple and fast method based on Kolmogorov-Smirnov test comparing two sets of normalized RPKM values. For a more precise analysis, the [baySeq](http://bioconductor.org/packages/release/bioc/html/baySeq.html) R package could be used instead.  
    _Input:_ two sets of read count files from RNA-Seq experiments in the tab-delimited format  
    _Output:_ list of differentially expressed genes
-   `gene_regions.py`: extracts gene regions based on offsets from transcription start sites.  
    _Input:_ gene annotation GTF file  
    _Output:_ BED file with selected regions
-   `intersect_vcf.py`: computes intersection between VCF files.  
    _Input:_ a set of VCF files  
    _Output:_ VCF file
-   `mussd.py`: implements the Mutation filtering based on the Space and Sample Distribution (**MuSSD**) algorithm which finds DNA regions that are highly mutated in multiple patients (_mutation blocks_).  
    _Input:_ a set of VCF files with patient somatic mutations, one per patient, a BED file with regions of interest, optionally a set of VCF files with patient germline mutations
    _Output:_ a pair of FASTA files for each block, containing reference and alternate sequences for each patient, in the format expected by `bayespi_bar.py`
-   `highly_mutated_blocks.py`: a simple method to find which blocks appear to have higher than average mutation rate, used to discard blocks with few mutations.  
    _Input:_ output of `mussd.py`  
    _Output:_ list of highly mutated blocks
-   `bayespi_bar.py`: implements the **BayesPI-BAR** [3] method to compute the protein-DNA binding affinity changes due to sequence variants.  
    _Input:_ two FASTA files with a set of paired reference and alternate sequences, as well as a folder containing PWM models in BayesPI format. The set of 1772 PWMs used in the pipeline is provided in the `data/pwm` folder  
    _Output:_ rankings of PWMs by predicted effect size, δdbA values for all PWMs and variants
-   `choose_background_parameters.py`: selects appropriate parameters (sequence size and mutation count distribution) for the background model based on chosen foreground (i.e. patient) mutation blocks.  
    _Input:_ output of `mussd.py` and `bayespi_bar.py`, optionally a mutation signature file  
    _Output:_ Bash script which runs `background_affinity_changes.py` with chosen parameters
-   `background_affinity_changes.py`: selects random mutation blocks for the background model and computes the background model using `bayespi_bar.py`.  
    _Input:_ BED file with regions of interest, a folder with PWM files, optionally a mutation signature file  
    _Output:_ δdbA values for all PWMs and randomly generated variants (same as `bayespi_bar.py`)
-   `affinity_change_significance_test.py`: performs the Wilcoxon rank-sum testing, comparing the patient affinity changes in mutation blocks to the background model.  
    _Input:_ a pair of outputs of `bayespi_bar.py`, for the background model and a mutation block  
    _Output:_ a table of PWMs which are affected in the foreground block significantly more than expected according to the background model
-   `plot_result.py`: produces heatmaps displaying predicted affinity changes across transcription factors and patient samples.  
    _Input:_ output of `affinity_change_significance_test.py` and `bayespi_bar.py`  
    _Output:_ PNG file

## Skin cancer demonstration pipeline 


The package includes an example analysis pipeline which reproduces the known result about mutations in the _TERT_ gene promoter that create binding sites for ETS family transcription factors. The pipeline calls the main package tools in appropriate sequence, reporting the progress of the computation.

To run the pipeline, go to the `demo/melanoma_full` folder and run the following commands:

1.  `python get_and_preprocess_data.py` to download the input and reference data and preprocess it into the right format.
2.  `python bayespi_bar2_pipeline.py` to execute the main pipeline code. This will take about one full day of computation on a multi-core machine. The computation speed can be greatly improved if you run the pipeline on a cluster which supports the SLURM queue manager. Edit the `parallel_options.txt` file in the same folder to specify the desired parallelization configuration. Check the help of `bayespi_bar.py` from the main package to learn about the parallelization options.
3.  `python make_plots.py` to make the heatmaps for the significantly affected transcription factors in the foreground blocks.

The main pipeline script, `bayespi_bar2_pipeline.py`, is designed to be robust to interruptions. If the pipeline execution was interrupted at any point, simply run the script again, and it will resume calculation from the place it was interrupted. You can see the progress of the computation as well as the main pipeline parameters in the log file, whose location is printed on the screen when the pipeline starts.

`bayespi_bar2_pipeline.py` is supposed to be the starting point for users wishing to use **BayesPI-BAR2** to process their own datasets. It shows a reasonable sequence of execution of the main package tools to obtain a final result. The source code in the file is simple, well-commented and should be easy to modify. The pipeline, as it is implemented in this script, can be logically divided into 8 steps, each step being a call to a tool from the package:

1.  Compute differentially expressed genes between patient group and the normal cell group (here we use gene expression in melanocytes as the normal group). Calls `differential_expression.py`.
2.  Select _regions of interest_, here defined as regions near transcription start sites of differentially expressed genes. Calls `gene_regions.py`.
3.  Compute mutation blocks in the regions of interest using **MuSSD**. Calls `mussd.py`.
4.  Select those blocks that are mutated more than average. The melanoma dataset contains large number of mutations, leading to many identified mutation blocks. This step removes 90% of blocks that have relatively few mutations. In cases when there are few mutation blocks identified by **MuSSD**, this step is probably unnecessary. Calls `highly_mutated_blocks.py`.
5.  Compute affinity changes for the foreground (patient) mutation blocks using **BayesPI-BAR**. Calls `bayespi_bar.py`.
6.  Compute the random background model for affinity changes, matching its parameters to the chosen foreground blocks. Calls `background_affinity_changes.py`.
7.  Run the affinity change significance tests for all foreground blocks vs. the background model. Calls `affinity_change_significance_test.py`.
8.  Remove the transcription factors that are not expressed in patient samples from the final predictions. This is implemented as an auxiliary tool from the demo folder because it relies on the specific naming convention of the supplied PWM database.

The `get_and_preprocess_data.py` script will download about 2 Gb of data necessary for the pipeline. Here is the full list of additional files that will be downloaded:

-   The reference genome hs37d5: [ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/phase2_reference_assembly_sequence/hs37d5.fa.gz](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/phase2_reference_assembly_sequence/hs37d5.fa.gz)
-   The GENCODE gene annotation v.19: [ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_19/gencode.v19.annotation.gtf.gz](ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_19/gencode.v19.annotation.gtf.gz)
-   The skin cancer input data and the normal melanocyte expression (once the paper is accepted for publication then the data set will be available for download): [hosted on this server](skin_cancer_icgc_and_normal_melanocytes_geo_data.tgz)

## References
1. BayesPI - a new model to study protein-DNA interactions: a case study of condition-specific protein binding parameters for Yeast transcription factors. Wang J, Morigen. _BMC Bioinformatics_. 2009 Oct 20;10:345. doi: 10.1186/1471-2105-10-345. [[PMID:19857274]](https://www.ncbi.nlm.nih.gov/pubmed/19857274) 
2. Quality versus accuracy: result of a reanalysis of protein-binding microarrays from the DREAM5 challenge by using BayesPI2 including dinucleotide interdependence. Wang J. _BMC Bioinformatics_. 2014 Aug 27;15:289. doi: 10.1186/1471-2105-15-289. [[PMID:25158938]](https://www.ncbi.nlm.nih.gov/pubmed/25158938)
3. BayesPI-BAR: a new biophysical model for characterization of regulatory sequence variations. Wang J, Batmanov K. _Nucleic Acids Res._ 2015 Dec 2;43(21):e147. doi: 10.1093/nar/gkv733 [[PMID:26202972]](https://www.ncbi.nlm.nih.gov/pubmed/26202972)
4. Integrative whole-genome sequence analysis reveals roles of regulatory mutations in BCL6 and BCL2 in follicular lymphoma. Batmanov K, Wang W, Bjoras M, Delabie J, Wang J. _Sci Rep._ 2017 Aug 1;7(1):7040. doi: 10.1038/s41598-017-07226-4. [[PMID:28765546]](https://www.ncbi.nlm.nih.gov/pubmed/28765546)
5. BayesPI-BAR2: a Python package for predicting functional non-coding mutations in cancer patient cohorts. Kirill Batmanov, Jan Delabie, and Junbai Wang (_submitted_)

  
