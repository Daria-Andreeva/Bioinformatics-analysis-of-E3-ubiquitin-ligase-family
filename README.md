# Bioinformatics analysis of E3 ubiquitin ligase family. 
## Prediction of E3 ubiquitin ligases’ binding sites



Proteolysis-targeting chimeras (PROTACs) induce targeted protein degradation by the ubiquitin-proteasome system. They represent a new therapeutic modality and are the focus of great interest, however the progress is hindered by the low efficiency of protein crystallography that provides E3 ubiquitin ligases' 3D structures required in the initial steps of PROTAC development. The automated in silico modeling tool could assist in expanding the number of enzymes available for development of targeted protein degradation systems. Binding site prediction on E3 ligase is another challenge in PROTAC designing. In this work we built a pipeline that executes molecular modeling of target protein and detected binding site predictions for three existing E3 ubiquitin ligase structures. 

In order to predict binding sites we ran a full-atom molecular dynamics simulation in water for three ligases (TRIM25, HUWE1, FBXW7) via GROMACS tool, predicted binding sites across the obtained trajectory using BiteNet tool, clustered the obtained predictions and finally selected the most promising ones.

### Objects:
Е3 ligase ТRIM25 (PDB structure: [5FER](https://www.rcsb.org/structure/5FER))

Е3 ligase HUWE1 (PDB structure: [3H1D](https://www.rcsb.org/structure/3H1D))

Е3 ligase FBXW7 (PDB structure: [2OVP](https://www.rcsb.org/structure/2OVP))

### Pipeline:
1. Run full-atom molecular dynamics simulation in water

      __a) Gromacs modelling__
      
      ```bash
      # install gromacs 2018 from repository
      apt-get install gromacs 
      
      # convert your pdb file into gro format, leaving only the protein of interest in the configuration (for this you can use grep or any other way). Select field amber03
      gmx pdb2gmx -f protein.pdb -o protein.gro -water spce
      
      # then we place the protein in a cubic cell, center it and set the distance to the cell border at 1 nm
      gmx editconf -f protein.gro -o protein_in_box.gro -c -d 1.0 -bt cubic
      
      # solve your protein in water
      gmx solvate -cp protein_in_box.gro -cs spc216.gro -o protein_in_water.gro -p topol.top
      
      # add ions to your system to balance the charge
      wget http://www.mdtutorials.com/gmx/lysozyme/Files/ions.mdp
      gmx grompp -f ions.mdp -c protein_in_water.gro -p topol.top -o ions.tpr
      gmx genion -s ions.tpr -o protein_in_water.gro -p topol.top -pname NA -nname CL -neutral
       
      # Minimizing system energy
      wget http://www.mdtutorials.com/gmx/lysozyme/Files/minim.mdp
      gmx grompp -f minim.mdp -c protein_in_water.gro -p topol.top -o em.tpr
      gmx mdrun -v -deffnm em
      
      # equilibration
      wget http://www.mdtutorials.com/gmx/lysozyme/Files/nvt.mdp
      gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr
      gmx mdrun -deffnm nvt
      wget http://www.mdtutorials.com/gmx/lysozyme/Files/npt.mdp
      gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr
      gmx mdrun -deffnm nvt
      
      # Production MD run
      wget http://www.mdtutorials.com/gmx/lysozyme/Files/md.mdp
      gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr
      gmx mdrun -deffnm md_0_1
      
      # Check the stability of the protein structure during simulation and calculate rmsd.
      gmx trjconv -s md_0_1.tpr -f md_0_1.xtc -o md_0_1_noPBC.xtc -pbc mol -center
      gmx rms -s md_0_1.tpr -f md_0_1_noPBC.xtc -o rmsd.xvg -tu ns
      ```
      
      RMSD for TRIM25
      
      ![Image alt](https://github.com/Daria-Andreeva/Bioinformatics-analysis-of-E3-ubiquitin-ligase-family/blob/main/pics/rmsd.png)
      
      The deviation reaches a constant level of about 2 angstroms, which means the stability of the structure.
      
      __b) Extraction of various conformations of ligase__
      
      From the resulting trajectory, select the number of frames required for analysis (20 in this project) and convert them to pdb format
      
      ```bash
      gmx trjconv -f md_0_1_noPBC.xtc -s md_0_1.tpr -b [start time] -e [end time] -o frame.pdb
      ```
 __2. Predict binding sites__
  
  We can now predict the most likely protein-peptide binding sites for each frame using [BiteNet](https://sites.skoltech.ru/imolecule/tools/bitenet/)
  As a result, to the mail indicated on the site, you will receive a csv file with predictions. The value of the score parameter (distributed from 0 to 1) reflects the most probable binding sites.
  ![Image alt](https://github.com/Daria-Andreeva/Bioinformatics-analysis-of-E3-ubiquitin-ligase-family/blob/main/pics/prediction_0.png)
__3. Cluster the obtained predictions__ 

To determine the most likely binding site, we average the scores for all 20 frames and choose the site with the highest score(the detailed code is in /supplementary_code/predictions_analyze.ipynb).

The following is visualization of TRIM25 by vmd, the site with the highest score is marked in red and amino acid residues are indicated.

![Image alt](https://github.com/Daria-Andreeva/Bioinformatics-analysis-of-E3-ubiquitin-ligase-family/blob/main/pics/top_site.jpg)

### References
1. Kozlovskii, I., Popov, P. Spatiotemporal identification of druggable binding sites using deep learning. Commun Biol 3, 618 (2020).
2. Schapira, M., Calabrese, M.F., Bullock, A.N. et al. Targeted protein degradation: expanding the toolbox. Nat Rev Drug Discov 18, 949–963 (2019).
    
