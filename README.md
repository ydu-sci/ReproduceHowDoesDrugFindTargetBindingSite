# ReproduceHowDoesDrugFindTargetBindingSite
MD exercise-DE Shaw 2011 JACS paper-How Does a Drug Molecule Find Its Target Binding Site

1) Protein and ligand preparation - only dasatinib 
  a. Download the dasatinib structure file(Structure3D_CID_3062316.sdf) from PubChem and 
     download the 1y57 pdb file form PDB.
  b. Although the paper used the Maestro to process the protein and ligand. I use Sybyl to
     convert sdf file to mol2 file and prepare the Src kinase. For detailed procedure,
     please refer to the Sybyl manual.
  c. Ligand:
     #antechamber -fi mol2 -fo gcrt -i dasatinib-sybyl.mol2  -o dasatinib-sybyl.gau
     #g09 dasatinib-sybyl.gau dasatinib-sybyl.out
     #antechamber -fi gout -fo prepi -i dasatinib-sybyl.out -o dasatinib-sybyl.prepi -c 
     #resp -j 4 -at gaff -rn DAS
     antechamber -fi mol2 -fo prepi -i dasatinib-sybyl.mol2 -o dasatinib-sybyl.prepi -c 
     bcc -j 5 -at gaff -rn DAS
     parmchk2 -i dasatinib-sybyl.prepi -f prepi -o frcmod
     #ADD THE para into gaff.dat, give it another name gaff.dat.das
     run -FROM desmond ff_amber_to_viparr.py -p gaff.dat -t dasatinib-sybyl.prepi DAS -f ligand
     
  d. Protein:
     Delete all water, hetatm and residues from 82 to 308. Side chain, cap termini, add H,
     fix sidechain amides
  f. Use Maestro to build the dasanitib and Src system, I get folder desmond_setup_dasatinib
  e. run -FROM desmond viparr.py -f amber99SB-ILDN -f tip3p -d DAS desmond_setup_dasatinib-out.cms 
     amber-out.cms
