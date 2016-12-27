#Reproduce How Does Drug Find Target Binding Site
######Du, Yu writed at 2016-11-12 14:08:29
######MD exercise-DE Shaw 2011 JACS paper-How Does a Drug Molecule Find Its Target Binding Site

1. Download the dasatinib structure file(Structure3D_CID_3062316.sdf) and the pp1 from PubChem and download the 1y57 pdb file form PDB.

2. Although the paper used the Maestro to process the protein and ligand, I use Sybyl to convert sdf file to mol2 file and prepare the Src kinase. For detailed procedure, please refer to the Sybyl manual.

3. Ligand preparation
  * `#antechamber -fi mol2 -fo gcrt -i dasatinib-sybyl.mol2  -o dasatinib-sybyl.gau`
  * `#g09 dasatinib-sybyl.gau dasatinib-sybyl.out`
  * `#antechamber -fi gout -fo prepi -i dasatinib-sybyl.out -o dasatinib-sybyl.prepi -c resp -j 4 -at gaff -rn DAS`
  * `antechamber -fi mol2 -fo prepi -i dasatinib-sybyl.mol2 -o dasatinib-sybyl.prepi -c bcc -j 5 -at gaff -rn DAS`
  * `parmchk2 -i dasatinib-sybyl.prepi -f prepi -o frcmod`
  * Add the frcmod para into gaff.dat, give it another name gaff.dat.das<br />`run -FROM desmond ff_amber_to_viparr.py -p gaff.dat -t dasatinib-sybyl.prepi DAS -f ligand`<br />Note: DAS is a folder that contains the template used by viparr.py
  
4. Protein preparation
  * Delete all water, hetatm and residues **not** from 259 to 533. Side chain, cap termini, add H, fix sidechain amides
  
5. Use Maestro to build the dasanitib and Src system, I get folder desmond_setup_dasatinib

6. `run -FROM desmond viparr.py -f amber99SB-ILDN -f tip3p -d DAS desmond_setup_dasatinib-out.cms amber-out.cms`

7. `run -FROM desmond build_constraints.py amber-out.cms amber-start.cms`

8. Load the amber-start.cms file to the Maestro and configure the Desmond according to the JACS paper, such as 
  * NVT ensemble at 310K using the Nose-Hoover thermostat with a relaxation time of 1.0 ps
  * All bond lengths to H constrained using M-SHAKE
  * Especially add the line backend = { force = { nonbond = { far = { type = gse n_k = [32 32 32] } } } } to the .cfg file
  
9. Write the .cms, .cfg and .msj files to the disk and transfer these files to the GPU node for MD computation. I use the command like the following
  * `env SCHRODINGER_CUDA_VISIBLE_DEVICES="0" multisim -JOBNAME final-md_pp1-3-400ns -HOST gpu-new -maxjob 1 -cpu 1 -m md_pp1-3-400ns.msj -c md_pp1-3-400ns.cfg -description "md_pp1-3-400ns" md_pp1-3-400ns.cms -set 'stage[1].set_family.md.jlaunch_opt=["-gpu"]' -o md_pp1-3-400ns-out.cms`

#####NOTE: 
* I can't figure out how to deal with the weak repulsive force between 6 dasatinib molecules. 
* The manual or guide of gaussian, antechamber and desmond are really helpful, one should never miss these wonderful materials.
* GPU Desmond is tremendously faster than the CPU version. GPU is strongly recommended.
* `jobcontrol` command from schrodinger is useful for job control.
* When merging molecules and proteins in the Maestro, different substructure id and name of molecules should be used in the mol2 file.
* J. Am. Chem. Soc., 2014, 136 (8), pp 3320–3320. Correction to How Does a Drug Molecule Find its Target Binding Site?Only the kinase domain (residues 259− 533) was included in the simulations.
