Aloi Davide - PhD Student (University of Birmingham - Centre for Human Brain Health)

# Analysis of the relationship between tDCS-induced current density and tDCS-induced effective connectivity changes in the motor network.

The goal of this study was to assess the relationship between current density, as simulated from MRI anatomical scans using the [ROAST](https://github.com/andypotatohy/roast#5-outputs-of-roast-software) pipeline (Huang et al., 2019), and effective connectivity changes within the motor network, derived using DCM and parametric empirical bayes (PEB). Anatomical and effective connectivity data were gathered from three different fMRI datasets, published in [Aloi et al., 2021](https://www.sciencedirect.com/science/article/pii/S1053811921010533?via%3Dihub) and [Aloi et al., 2023](https://www.sciencedirect.com/science/article/pii/S1053811923002963?via%3Dihub).

## Analysis Pipeline
![Analysis pipeline](https://github.com/davide-aloi/wp1_2_roast/blob/main/figures/exp%20diagram%20wp1%20-%20Page%201.png)


The three dataset analysed come from three different experiments: 
- wp1a and wp1b (i.e., experiment 1 and 2): participants received one session of anodal, cathodal and sham stimulations, in a counterbalanced order. In wp1a, the target of the stimulation was the left motor cortex. In wp1b, the target was the right cerebellum. Connectivity in the motor network was estimated using dynamic causal modelling (DCM) both before and after stimulation, while participants performed a motor task (simple thumb movement).The datasets are the same of [Aloi et al. (2021)](https://www.sciencedirect.com/science/article/pii/S1053811921010533?via%3Dihub).
- wp2a (i.e., experiment 3): same design as above but the stimulation was administered coupled with passive mobilisation of the thumb. The target of the stimulation was the left motor cortex. Participants received 5 stimulations per week, with one week gap between each stimulation condition, and were scanned on Day-1 and Day-5 of each week. (The publication is now accessible as preprint https://doi.org/10.1101/2022.11.22.517479). 

For all datasets we first used DCM to estimate effective connectivity before / after stimulation in each fMRI session. Then, we used a hierarchical approach to estimate pairwise interactions between Time (pre < post stimulation) and Polarity (Anodal vs sham for wp1a and wp2a, and cathodal vs sham for wp1b). For this, we ran a 1st level PEB with contrast pre < post for all 3 stimulation conditions, and then a 2nd level PEB coding the interaction stimulation x time (i.e. anodal pre < anodal post > sham pre < sham post OR cathodal pre < cathodal post > sham pre < sham post).

Of the resulting DCM matrices, we focused only on the connections between M1 and TH (including self connectivities). We took these values and correlated them with current density metrics calculated from MRI-derived models (Max and Median current density values for M1, TH and CB (wp1b only)). This approach is similar to that of Indahlastari et al., 2021 (however here we're using effective and not functional connectivity)

## Snapshot of results 

Current density metrics of each stimulation protocol.
![Current density](CD_medians_allstudies.png)

Canonical Correlation Analysis outcome for Experiment 3 (M1 as target of the stimulation)
<img src="https://github.com/davide-aloi/wp1_2_roast/blob/main/figures/ccca_updated.PNG?raw=true" width="696" height="419">


## Example of Electric field maps
![e-field_figure](https://user-images.githubusercontent.com/4202630/149754221-386e4582-4a39-4723-8e4f-cd94f999f839.png)
(nb the electric field maps were converted to current density prior to the CCA and voxel based analyses)

## Analysis steps and scripts

1) [Rename files](https://github.com/Davi93/wp1_2_roast/blob/main/wp2a_roast_1_rename_scans.py): this renames the anatomical scans of each participant (i.e. sub-01_T1.nii etc) of dataset wp2a. 
2) Roast simulations: in brief, ROAST outputs the following scans for each subject, while also using SPM routines for tissue segmentation: Voltage ("subjName_simulationTag_v.nii", unit in mV), E-field ("subjName_simulationTag_e.nii", unit in V/m) and E-field magnitude ("subjName_simulationTag_emag.nii", unit in V/m). This was done for all three datasets. The coordinates of the electrodes were visually derived using MRIcroGL. [wp2a](https://github.com/Davi93/wp1_2_roast/blob/main/wp2a_roast_2_roast_simulation.m), [wp1a](https://github.com/Davi93/wp1_2_roast/blob/main/wp1a_roast_2_roast_simulation.m), [wp1b](https://github.com/Davi93/wp1_2_roast/blob/main/wp1b_roast_2_roast_simulation.m). 

3) Post ROAST preprocessing: ROAST outputs are in ROAST model space. These scripts move ROAST results back to the MRI space, coregisters and normalises the electric field maps generated by ROAST. The script also normalise the T1 scan and all the masks. NB. ROAST creates a mask in which it combines all spm maks, after applying morphological operations and heuristics on SPM masks to remove holes (unassigned voxels). This mask will be used to calculate current density starting from electric field magnitude. The script was adapted from Luke Andrews's MsC thesis. Scripts: [wp2a](https://github.com/Davi93/wp1_2_roast/blob/main/wp2a_roast_3_post_roast_preprocessing.m), [wp1a](https://github.com/Davi93/wp1_2_roast/blob/main/wp1a_roast_3_post_roast_preprocessing.m), [wp1b](https://github.com/Davi93/wp1_2_roast/blob/main/wp1b_roast_3_post_roast_preprocessing.m). All these three scripts refer to the same [job](https://github.com/Davi93/wp1_2_roast/blob/main/wp2a_roast_3_post_roast_preprocessing_job.m) file.
4) Starting from the DCM estimations for each fMRI session, we ran a 1st level PEB to estimate pre vs post changes (pre < post). I already had the pre vs post results for wp2a, whereas for wp1a and b I used [this](https://github.com/Davi93/wp1_2_roast/blob/main/wp1a_roast_4_1_run_and_extract_single_dcms.m) script. This script takes the GCM files within wp1a_DCMfiles and wp1b_DCMfiles and runs 3 pebs per subjects (pre < post for each polarity).
5) 2nd level PEB: this is done to run 3 pairwise interactions: Anodal vs Sham, Anodal vs Cathodal, Cathodal vs Sham. For wp1a and wp2a I'm interested in the first pairwise interaction, for wp1b I'm interested in the last one. For wp2a, I already had these pairwise interactions, both for day 1 and day 5. For wp1a and wp1b I had to run them using these scripts ([1](https://github.com/Davi93/wp1_2_roast/blob/main/wp1a_roast_4_3_run_pairwise_int.m), [2](https://github.com/Davi93/wp1_2_roast/blob/main/wp1b_roast_4_3_run_pairwise_int.m)) starting from the results of the previous PEB. 
6) DCM values extraction: this script extracts from the files "..all_EPvalues_pairwiseint.mat" - present in each DCMfiles folder - all the DCM values I'm interested in (Anod vs Sham or Cath vs Sham). Results are saved in numpy files in the folder all_dcm_results. [Script](https://github.com/Davi93/wp1_2_roast/blob/main/wp_all_4_3_DCM_matrices_extraction_pairwiseint.ipynb).
7) Current density calculation: the script calculates current density for each subject of each dataset, using the function current_density_efield (see below). Results are saved in 3 different 4d scans (one per dataset, with 1 volume per subject). [Script](https://github.com/Davi93/wp1_2_roast/blob/main/wp_all_6_current_density_calculation.ipynb).
8) Current density metrics: the script calculates median and max current density values for each subject and dataset, for the three ROIs left M1, left thalamus and right cerebellum. Results are saved in three .CSV files. [Script](https://github.com/Davi93/wp1_2_roast/blob/main/wp_all_7_current_density_metrics.ipynb).
9) Canonical correlation analysis between current density metrics and DCM metrics corresponding to 3 pairwise interactions: Anodal vs Sham (A-S) (wp1a), A-S (wp2a) and CS (wp1b). [Script](https://github.com/Davi93/wp1_2_roast/blob/main/wp_all_8_current_density_cor_pairwise_interactions.ipynb)
10) Voxel based correlation analysis: correlation between each voxel of each roi and DCM metrics (and multiple comparison correction) [Script](https://github.com/Davi93/wp1_2_roast/blob/main/wp_all_cor_cd_dcm.ipynb)

## Functions
Functions are contained in folder [custom_functions](https://github.com/Davi93/wp1_2_roast/tree/main/custom_functions)

-- [Functions](https://github.com/Davi93/wp1_2_roast/blob/main/custom_functions/maps_functions.py): these are functions that perform operations on scans.
- current_density_efield: calculates current density starting from electric field magnitude and a brain mask. 

-- [Plotting functions](https://github.com/Davi93/wp1_2_roast/blob/main/custom_functions/plotting_functions.py)
- roast_vector_sim: plots electric field directions and intensities. 


## References:
1) Huang, Y., Datta, A., Bikson, M., & Parra, L. C. (2019). Realistic volumetric-approach to simulate transcranial electric stimulation—ROAST—a fully automated open-source pipeline. Journal of Neural Engineering, 16(5), 056006. https://doi.org/10.1088/1741-2552/ab208d
2) Indahlastari, A., Albizu, A., Kraft, J. N., O’Shea, A., Nissim, N. R., Dunn, A. L., Carballo, D., Gordon, M. P., Taank, S., Kahn, A. T., Hernandez, C., Zucker, W. M., & Woods, A. J. (2021). Individualized tDCS modeling predicts functional connectivity changes within the working memory network in older adults. Brain Stimulation, 14(5), 1205–1215. https://doi.org/10.1016/j.brs.2021.08.003
3) Albizu, A., Fang, R., Indahlastari, A., O’Shea, A., Stolte, S. E., See, K. B., Boutzoukas, E. M., Kraft, J. N., Nissim, N. R., & Woods, A. J. (2020). Machine learning and individual variability in electric field characteristics predict tDCS treatment response. Brain Stimulation, 13(6), 1753–1764. https://doi.org/10.1016/j.brs.2020.10.001

