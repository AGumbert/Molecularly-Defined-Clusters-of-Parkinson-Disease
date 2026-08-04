
=============================================================================================
====================== Overview of Repository and Processing Pipelines ======================
=============================================================================================

This repository contains code files to support analysis of transcriptomic and proteomic data from PPMI. After processing the transcriptomic and proteomic data, files in this repository perform clustering with the iClusterPlus package and then test the resulting clusters for clinical and biological implications.

Here is a brief overview of the current files in the processing pipelines in this repository.


Files used to define global functions used across downstream analyses:

1. Get_UPDRS_Overview_By_Groups.Rmd


The main processing pipeline is designed to select features from the transcriptomic and proteomic data within the cohort of sporadic PD based on analysis of UPDRS outcomes. The recommended order of running files in this pipeline is:

1. PPMI_UPDRS_Cluster_Creation.Rmd
2. PPMI_Data_Exploration_Transcriptomics_based_on_UPDRS.Rmd
3. PPMI_Data_Exploration_Proteomics_based_on_UPDRS.Rmd
4. PPMI_Molecular_Cluster_Creation_UPDRS_features.Rmd


This file performs gene set enrichment and cell-type deconvolution analysis on the transcriptomic data across clusters from the previous analyses:

1. PPMI_Transcriptomic_GSEA.Rmd


See below for more detailed descriptions of files.


=============================================================================================
============================== Descriptions of Individual Files =============================
=============================================================================================


IMPORTANT NOTE: Each file requires access to certain data for input, and some files result in the creation of output data in .txt or .csv format. In the following descriptions, all of these items for input and output are located in the "main_directory" unless otherwise noted. This "main_directory" variable is defined at the start of each analysis.


---------------------------------------------------------------------------------------------
----------- Files used to define global functions used across downstream analyses -----------
---------------------------------------------------------------------------------------------


***** Get_UPDRS_Overview_By_Groups.Rmd *****

Description:

This file defines a function that generates line plots and statistical comparison results from UPDRS data corresponding to two distinct groups of patients. Both UPDRS raw scores and UPDRS changes from baseline are plotted and compared by group at years 0, 1, 2, and 3. The function takes two vectors of patient IDs corresponding to the two groups as input. Two additional parameters are optional and can specify the respective names of each group as strings. This function is used to generate plots and statistical comparisons in other files in this repository.

This file requires access to:

- PPMI_Curated_Data_Cut_Public_20251112.xlsx.

This file results in:

- The "get_UPDRS_overview_by_groups" function is defined.



---------------------------------------------------------------------------------------------
------------ The processing pipeline with feature selection based on UPDRS scores -----------
---------------------------------------------------------------------------------------------


***** PPMI_UPDRS_Cluster_Creation.Rmd *****

Description:

Creates clusters of sporadic PD cases based on UPDRS data from the PPMI curated data set. Uses longitudinal k-means clustering from the kml package. The clusters are based on UPDRS total scores from the first three years after baseline. The clusters generated in this file are used for downstream analyses in PPMI_Data_Exploration_Transcriptomics_based_on_UPDRS.Rmd and PPMI_Data_Exploration_Proteomics_based_on_UPDRS.Rmd. After creating clusters, this file visualizes descriptive and summary statistics relevant to each cluster.

This file requires access to:

- PPMI_Curated_Data_Cut_Public_20251112.xlsx.
- Downstream analyses use the function defined in Get_UPDRS_Overview_By_Groups.Rmd.

This file results in:

- Clusters A and B are defined based on UPDRS scores.
- Cluster A PATNOs are stored in "cluster_A_patients_overall" vector.
- Cluster B PATNOs are stored in "cluster_B_patients_overall" vector.



***** PPMI_Data_Exploration_Transcriptomics_based_on_UPDRS.Rmd *****

Description:

This file uses the UPDRS-based clusters created in PPMI_UPDRS_Cluster_Creation.Rmd. The file also uses the PPMI transcriptomic data. The transcriptomic data is derived from Illumina technology applied to whole blood samples. The file performs differential expression analysis with DESeq2 to identify differentially expressed genes by UPDRS-based cluster. The resulting differentially expressed features are saved in text files with and without the decimal value that specifies the version number, enabling further analyses in other files. 

This file requires access to:

- meta_data.11192021.csv.
- iu_genetic_consensus_20250515_18Jul2025.csv.
- The PPMI_RNAseq_IR3_Analysis folder with bundled transcriptomic data.
- Clusters generated in PPMI_UPDRS_Cluster_Creation.Rmd (defined as global vectors).

This file results in:

- Creation of newline-separated list of selected features in "selected_transcriptomic_features_from_UPDRS_A_vs_B.txt", useful for exporting to other files for analysis, including PPMI_Molecular_Cluster_Creation_UPDRS_features.Rmd.
- Creation of comma-separated list of selected features in "selected_transcriptomic_features_from_UPDRS_A_vs_B_comma_separated.txt", useful for copying to analyses in AMP. 
- Creation of newline-separated list of selected features without the version number in "selected_transcriptomic_features_from_UPDRS_A_vs_B_no_version.txt", useful for external gene set analysis.


***** PPMI_Data_Exploration_Proteomics_based_on_UPDRS.Rmd *****

Description:

This file  considers the UPDRS-based clusters from PPMI_UPDRS_Cluster_Creation.Rmd. The file also considers plasma, CSF, and urine proteomics data from Olink, SomaScan, and mass spectrometry. The code in this file acquires the numbers of cases with usable data from each type of proteomic modality. The file finds proteins that have relatively high differential expression by UPDRS cluster to identify candidate features for the creation of molecularly defined clusters. The resulting differentially expressed features are saved in text files, enabling further analyses and cluster creation. 

The file requires access to:

- PPMI plasma, urine, and CSF proteomic data files in the "main_directory/Proteomic_Analysis".
- Clusters generated in PPMI_UPDRS_Cluster_Creation.Rmd (defined as global vectors).


This file results in:

- Creation of newline-separated list of selected urine features in "selected_proteomic_features_urine_from_UPDRS_A_vs_B.txt", useful for exporting to other files for analysis, including PPMI_Molecular_Cluster_Creation_UPDRS_features.Rmd.
- Creation of newline-separated list of selected CSF features in "selected_proteomic_features_CSF_from_UPDRS_A_vs_B.txt", useful for exporting to other files for analysis, including PPMI_Molecular_Cluster_Creation_UPDRS_features.Rmd.


***** PPMI_Molecular_Cluster_Creation_UPDRS_features.Rmd *****

Description:

This file performs clustering analysis with the selected multi-omics features from PPMI_Data_Exploration_Transcriptomics_based_on_UPDRS.Rmd and PPMI_Data_Exploration_Proteomics_based_on_UPDRS.Rmd. The file considers the full cohort of sporadic cases, even those that did not have sufficient UPDRS data to qualify for a UPDRS-based cluster assignment. Accordingly, this program loads and processes the omics data directly from the raw files to capture all sporadic cases. With the clustering results, this file tests the clusters for clinical implications and differences with respect to covariates. 

This file requires access to:

- PPMI_Curated_Data_Cut_Public_20251112.xlsx.
- All omics data files used in PPMI_Data_Exploration_Transcriptomics_based_on_UPDRS.Rmd and PPMI_Data_Exploration_Proteomics_based_on_UPDRS.Rmd.
- Clusters generated in PPMI_UPDRS_Cluster_Creation.Rmd (defined as global vectors).
- List of selected transcriptomic features in "selected_transcriptomic_features_from_UPDRS_A_vs_B.txt".
- List of selected urine proteomic features in "selected_proteomic_features_urine_from_UPDRS_A_vs_B.txt".
- List of selected CSF proteomic features in "selected_proteomic_features_CSF_from_UPDRS_A_vs_B.txt".
- Downstream analyses use the function defined in Get_UPDRS_Overview_By_Groups.Rmd.
- Downstream analyses use Genomic data in PPMI_Project_9001_20250624_21Oct2025.csv (in "main_directory/Genomic_Analysis").

This file results in:

- Creation of molecularly defined clusters tested for clinical implications, neurological implications, and covariates.

---------------------------------------------------------------------------------------------
-------------------- Analyses that are performed after cluster creation  --------------------
---------------------------------------------------------------------------------------------

***** PPMI_Transcriptomic_GSEA.Rmd *****

Description:

This file takes two vectors of participant ids as input in order to specify two groups to compare. The file performs differential expression of transcriptomic data across the two groups. Given the ranked differentially expressed genes across the two groups, the file performs gene set enrichment analysis with the gseGO function from the clusterProfiler package. The file then visualizes the results to indicate the most differentially expressed gene sets across the groups. This file can also support cell-type deconvolution across the two groups or GSEA based on beta values from iClusterBayes to characterize the latent space dimensions used in clustering.

This file requires access to:

- Two vectors of participant ids to compare.
- meta_data.11192021.csv.
- iu_genetic_consensus_20250515_18Jul2025.csv.
- The PPMI_RNAseq_IR3_Analysis folder with bundled transcriptomic data.
- (optional) ranked beta values from iClusterBayes to enable GSEA according to the latent space dimensions.

This file results in:

- Calculates the most differentially expressed gene sets across the two groups and visualizes the results.
- Can also perform cell-type deconvolution analysis to identify cell types associated with the two groups.
- Can also identify gene sets related to the iClusterBayes latent space according to the ranked beta values.


