# AutoMD
Fast to molecular dynamics simulation.  [中文文档](https://zhuanlan.zhihu.com/p/595698129)

Installtion
----
### 1. Install the Desmond package.  
*   Firstly, download the academic edition of [Desmond package](https://www.deshawresearch.com/resources.html) and install it on you HPC or PC. Of course, you can install the [SCHRODINGER package](https://www.schrodinger.com/downloads/releases) instead the academic edition of Desmond. Then, change the directory to the installtion path of Desmond or SCHRODINGER, and run this command to set the `$Desmond`:  
```
echo "export Desmond=${PWD}/" >> ~/.bashrc
```
*   Then, configure the `${Desmond}/schrodinger.hosts` following this paper: [How do I configure my schrodinger.hosts file for Desmond GPU jobs?
](https://www.schrodinger.com/kb/1844).   
### 2. Install the viparr and msys by pip.  
*   Firstly, download the msys and viparr.   
```
wget https://github.com/DEShawResearch/viparr/releases/download/4.7.35/viparr-4.7.35-cp38-cp38-manylinux2014_x86_64.whl
wget https://github.com/DEShawResearch/msys/releases/download/1.7.337/msys-1.7.337-cp38-cp38-manylinux2014_x86_64.whl
```
*   Secondly, run these commands to create a new virtual environment for viparr.
```
${Desmond}/run schrodinger_virtualenv.py schrodinger.ve
source schrodinger.ve/bin/activate
pip install --upgrade pip
```
*   Then, install the msys and viparr by Schrödinger pip.
```
pip install msys-1.7.337-cp38-cp38-manylinux2014_x86_64.whl
pip install viparr-4.7.35-cp38-cp38-manylinux2014_x86_64.whl
echo "export viparr=${PWD}/schrodinger.ve/bin" >> ~/.bashrc
```
*   Then, clone the public viparr parameters, and set the environment variable.  
```
git clone git://github.com/DEShawResearch/viparr-ffpublic.git
echo "export VIPARR_FFPATH=${PWD}/viparr-ffpublic/ff" >> ~/.bashrc
```

### 3. Download AutoMD and additional force fields.
*   Download AutoMD and set the environment variable.
```
git clone https://github.com/Wang-Lin-boop/AutoMD
cd AutoMD
echo "alias AutoMD=${PWD}/AutoMD" >> ~/.bashrc
chmod +x AutoMD
source ~/.bashrc
```
*   Copy the mreged Amber force field to `${VIPARR_FFPATH}`.
```
cp -r ff/* ${VIPARR_FFPATH}/
```

Usage
----
__Options:__  
*   Input parameter: Use Mae or CMS as input, that, MAE can contain pre-defined membrane locations.   
```
  -i    Use a file name (Multiple files are wrapped in "", and split by ' ') "*.mae" or "*.cms" ;  
            or regular expression to represent your input file, default is *.mae.  
```
*   Solution Builder parameter: INC or OUC, this often depends on where the protein is located. 
```
  -S    System Build Mode: <INC>  
        INC: System in cell, salt buffer is 0.15M KCl, water is SPC. Add K to neutralize system.  
        OUC: System out of cell, salt buffer is 0.15M NaCl, water is SPC. Add Na to neutralize system.    
        Custom Instruct: Such as: "TIP4P:Cl:0.15-Na-Cl+0.02-Fe2-Cl+0.02-Mg2-Cl"    
            Interactive addition of salt. Add Cl to neutralize system.  
                for positive_ion: Na, Li, K, Rb, Cs, Fe2, Fe3, Mg2, Ca2, Zn2 are predefined.  
                for nagative_ion: F, Cl, Br, I are predefined.  
                for water: SPC, TIP3P, TIP4P, TIP5P, DMSO, METHANOL are predefined.  
```
*   Membrane builder parameter: Typically, POPC membranes for eukaryotes, whereas POPE membrane for prokaryotes.
```
  -l    Lipid type for membrane box. Use this option will build membrane box. <None>  
            Lipid types: POPC, POPE, DPPC, DMPC.  
```
*   Simulation box builder parameters: box shape and size.
```
  -b    Define a boxshape for your systems. <cubic>  
            box types: dodecahedron_hexagon, cubic, orthorhombic, triclinic  
  -s    Define a boxsize for your systems.  <15.0>  
            for dodecahedron_hexagon and cubic, defulat is 15.0;  
            for orthorhombic or triclinic box, defulat is [15.0 15.0 15.0];  
            If you want use Orthorhombic or Triclinic box, your parameter should be like "15.0 15.0 15.0"  
```
*   Force fields parameters: 
```
  -R    Redistribute the mass of heavy atoms to bonded hydrogen atoms to slow-down high frequency motions.  
  -F    Define a force field to build your systems. <OPLS_2005>  
            OPLS_2005, S-OPLS are recommended to receptor-ligand systems.  
            Amber, Charmm, DES-Amber are recommended to other systems.
            Use the "Custom" to load parameters from input .cms file.  
            
The current force fields support in AutoMD:
    S-OPLS:
        The force fields in Schrödinger packages, recommended to ligand-protein complex.
    OPLS_2005:
        The default force field of Desmond package.
    Amber:
        Recommended to protein, DNA, RNA, lipid and other systems.
        Amber-ff19SB for protein, Amber-ffncaa for non-canonical aa, Amber-ffptm for
        post-translational modifications, amber1jc ion parameters adapt with spce,
        tip3p or tip4pew, Amber-bsc1 for DNA, Amber-tan2018 for RNA.
    Charmm:
        Recommended to protein, DNA, RNA, lipid, carbohydrate and other systems.
        Charmm36m for protein, Charmm36 for carbohydrate, ions, lipid and nucleic acid.
    DES-Amber:
        Recommended to protein-protein complex.
        DES-Amber for protein-protein complex.
```
*   Simulation control parameter:  
```
  -m    Enter the maximum simulation time for the Brownian motion simulation, in ps. <100>  
  -r    The relaxation protocol before MD, "Membrane" or "Solute". <Solute>  
  -e    The ensemble class in MD stage, "NPT", "NVT", "NPgT". <NPT>  
  -t    Enter the Molecular dynamics simulation time for the product simulation, in ns. <100>  
  -T    Specify the temperature to be used, in kelvin. <310>  
  -N    Number of Repeat simulation with different random numbers. <1>  
  -P    Define a ASL to receptor, such as "protein".  
  -L    Define a ASL to ligand and run interaction analysis, such as "res.ptype UNK".  
  -u    Turn off md simulation, only system build.  
  -C    Set constraint to an ASL, such as "chain.name A AND backbone"  
  -f    Set constraint force, default is 10.  
  -o    Specify the approximate number of frames in the trajectory.  <1000>  
        This value is coupled with the recording interval for the trajectory and the simulation time:  
        the number of frames times the trajectory recording interval is the total simulation time.  
        If you adjust the number of frames, the recording interval will be modified.  
```
__Example:__   
  1) MD for cytoplasmic protein-ligand complex:  
```
AutoMD -i "*.mae" -S INC -P "chain.name A" -L "res.ptype UNK" -F "OPLS_2005"  
```
  2) MD for plasma protein-protein complex:  
```
AutoMD -i "*.mae" -S OUC -F "DES-Amber"  
```
  3) MD for DNA/RNA-protein complex:  
```
AutoMD -i "*.mae" -S "SPC:Cl:0.15-K-Cl+0.02-Mg2-Cl" -F Amber  
```
  4) MD for membrane protein, need to prior place membrane in Meastro.  
```
AutoMD -i "*.mae" -S OUC -l "POPC" -r "Membrane" -F "Charmm"  
```  

Molecule Dictionary
----
_You can retrieve your small molecule in the following database, obtaion the residue name corresponding to your small molecule, and check that this residue name is present in the templates of some kind of force field._    
#### 1. (Recommended) Charmm Small Molecule Library (CSML)  
*   [Charmm-GUI CSML](https://charmm-gui.org/?doc=archive&lib=csml): Small Molecules for Charmm Force fields  

#### 2. Amber Lipid, DNA, RNA and others
*   [Amber Lipid 17](https://ambermd.org/AmberModels_lipids.php)  
*   [Amber Force Fields for DNA, RNA, and others](https://ambermd.org/AmberModels.php)  
*   [AMBER parameter database](http://amber.manchester.ac.uk/):  Small Molecule parameters for Amber Force fields   

Use the following commands to find the force field which contained your small molecule:   
```
grep ": {$" $VIPARR_FFPATH/*/templates | awk -F: '{print $1,$2}' |  sed 's/\/templates//g' | awk -F/ '{print $NF}' > VIPARR_Dictionary.index
grep '<your residue name>' VIPARR_Dictionary.index
```

_**Then, make sure specify the same ff encompassing this residue in AutoMD when you run the MD simulatuons.**_  
_Note that, in AutoMD, the -F Charmm ff is combined by aa.charmm.c36m, misc.charmm.all36, carb.charmm.c36, ethers.charmm.c35, ions.charmm36, lipid.charmm.c36 and na.charmm.c36, the -F Amber ff is combined by aa.amber.19SBmisc, aa.amber.ffncaa, lipid.amber.lipid17, ions.amber1lm_iod.all, ions.amber2ff99.tip3p, na.amber.bsc1 and na.amber.tan2018, the -F DES-Amber is combined by aa.DES-Amber_pe3.2, dna.DES-Amber_pe3.2, rna.DES-Amber_pe3.2 and other force fields in -F Amber._ 

_Alternatively, you can also refer to the method [here for Amber](https://www.protocols.io/view/how-to-assign-amber-parameters-to-desmond-generate-bp2l6bqwkgqe/v1?step=5) or [here for Charmm](https://www.protocols.io/view/how-to-assign-charmm-parameters-to-desmond-generat-q26g78pr8lwz/v1?step=4) using self prepared Amber or Charmm small molecule force field files._   

Disclaimer
----
_This script was developed to speed up my own work, and I put this script here for convenience for sharing to some people who need it. Discussion with me is welcome if you also wish to use it and have some problems, but I do not guarantee that you will solve it. Of note, this script was developed based on a series of software from the [D. E. Shaw Research](https://github.com/DEShawResearch), all credit to D. E. Shaw Research. I declare no competing interest._    

Acknowledgments
---
We would like to express our special thanks to the following individuals and organizations, whose contributions have been invaluable to our research:    
*   Thilo Mast and Dmitry Lupyan for sharing their methods for using alternative force fields in Desmond, which provided us with a valuable foundation for our work;    
*   D E Shaw Research and Schrödinger for their significant contributions to the development and maintenance of the Desmond software, which has allowed us to leverage this powerful platform for our research.   

