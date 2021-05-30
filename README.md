# Bioinformatics analysis of E3 ubiquitin ligase family
## Prediction of E3 ubiquitin ligases’ binding sites



Proteolysis-targeting chimeras (PROTACs) induce targeted protein degradation by the ubiquitin-proteasome system. They represent a new therapeutic modality and are the focus of great interest, however the progress is hindered by the low efficiency of protein crystallography that provides E3 ubiquitin ligases' 3D structures required in the initial steps of PROTAC development. The automated in silico modeling tool could assist in expanding the number of enzymes available for development of targeted protein degradation systems. Binding site prediction on E3 ligase is another challenge in PROTAC designing. In this work we built a pipeline that executes molecular modeling of target protein and detected binding site predictions for three existing E3 ubiquitin ligase structures. 

In order to predict binding sites we ran a full-atom molecular dynamics simulation in water for three ligases (TRIM25, HUWE1, FBXW7) via GROMACS tool, predicted binding sites across the obtained trajectory using BiteNet tool, clustered the obtained predictions and finally selected the most promising ones.

### Objects:
Е3 ligase ТRIM25 (PDB structure: [5FER](https://www.rcsb.org/structure/5FER))

Е3 ligase HUWE1 (PDB structure: [3H1D](https://www.rcsb.org/structure/3H1D))

Е3 ligase FBXW7 (PDB structure: [2OVP](https://www.rcsb.org/structure/2OVP))

### Pipeline:
  1. Run full-atom molecular dynamics simulation in water
      
      a) Gromacs modelling
      ```bash
      sudo apt-get install gromacs # install gromacs 2018 from repository
      
      
      ```
      b) Extraction of various conformations of ligase
  2. Predict binding sites
  3. Cluster the obtained predictions 
    
