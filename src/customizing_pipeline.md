﻿# Customizing the default BayesPI-BAR2 pipeline
The `bayespi_bar2_pipeline.py` script is the starting point for users wishing to use **BayesPI-BAR2** to process their own datasets. It shows a reasonable sequence of execution of the main package tools to obtain a final result. The source code in the file is simple, well-commented and should be easy to modify. The pipeline, as it is implemented in this script, can be logically divided into 8 steps, each step being a call to a tool from the package:

1.  Compute differentially expressed genes between patient group and the normal cell group (here we use gene expression in melanocytes as the normal group). Calls `differential_expression.py`.
2.  Select _regions of interest_, here defined as regions near transcription start sites of differentially expressed genes. Calls `gene_regions.py`.
3.  Compute mutation blocks in the regions of interest using **MuSSD**. Calls `mussd.py`.
4.  Select those blocks that are mutated more than average. The melanoma dataset contains large number of mutations, leading to many identified mutation blocks. This step removes 90% of blocks that have relatively few mutations. In cases when there are few mutation blocks identified by **MuSSD**, this step is probably unnecessary. Calls `highly_mutated_blocks.py`.
5.  Compute affinity changes for the foreground (patient) mutation blocks using **BayesPI-BAR**. Calls `bayespi_bar.py`.
6.  Compute the random background model for affinity changes, matching its parameters to the chosen foreground blocks. Calls `background_affinity_changes.py`. There is a choice of background models:
	- A tumor-derived background model will randomly samples patients' tumor somatic mutations to build the background model. It assumes that most mutations have no effect. This model most closely matches various features of the tumor mutation biases, including location-specific ones. However it requires a large mutation sample to generate a good background model. At least a few thousand mutations is needed.
	- A mutation signature model uses k-mer probability distribution specified by the user to randomly generate mutations. These distributions for many known cancer subtypes can be obtained from COSMIC, or generated by other means. The mutation locations are chosen randomly.
	- A simplest uninformed background model will generate mutations uniformly, in both location and nucleotide replacement. False positives due to cancer-specific mutation biases are to be expected with this model.
7.  Run the affinity change significance tests for all foreground blocks vs. the background model. Calls `affinity_change_significance_test.py`.
8.  Remove the transcription factors that are not expressed in patient samples from the final predictions. This is implemented as an auxiliary tool from the demo folder because it relies on the specific naming convention of the supplied PWM database.