# BayesPI-BAR2 pipeline tools

The **BayesPI-BAR2** package is a set of command line tools residing in the `python` folder. Run `python <tool_name.py> --help` command to see the full usage information for a particular tool. The purpose of each tool and the data involved in the processing are described below:

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
    _Input:_ output of `mussd.py` and `bayespi_bar.py`, optionally a mutation signature file or a set of background mutations to use
    _Output:_ Bash script which runs `background_affinity_changes.py` with chosen parameters
-   `background_affinity_changes.py`: selects random mutation blocks for the background model and computes the background model using `bayespi_bar.py`.  
    _Input:_ BED file with regions of interest, a folder with PWM files, optionally a mutation signature file or a set of background mutations to use
    _Output:_ δdbA values for all PWMs and randomly generated variants (same as `bayespi_bar.py`)
-   `affinity_change_significance_test.py`: performs the Wilcoxon rank-sum testing, comparing the patient affinity changes in mutation blocks to the background model.  
    _Input:_ a pair of outputs of `bayespi_bar.py`, for the background model and a mutation block  
    _Output:_ a table of PWMs which are affected in the foreground block significantly more than expected according to the background model
-   `plot_result.py`: produces heatmaps displaying predicted affinity changes across transcription factors and patient samples.  
    _Input:_ output of `affinity_change_significance_test.py` and `bayespi_bar.py`  
    _Output:_ PNG file
