# CμMD Implementation for PLUMED2

This repository hosts the implementation for [PLUMED 2](https://www.plumed.org) of the CmuMD method, which allows to mimic open boundary conditions in NVT molecular simulations involving solid/fluid interfaces, and enables the simulation of concentration-driven fluxes through porous membranes. 

The method, with an application to crystal growth is described in the paper: 

__[1] Molecular dynamics simulations of solutions at constant chemical potential__  
C Perego, M Salvalaglio, M Parrinello, [J Chem Phys (14), 144113, 2015](https://moodle.ucl.ac.uk/course/view.php?id=1191), [arxiv](https://arxiv.org/abs/1501.07825)

The adaptation of CmuMD to model concentration driven membrane fluxes is described in the paper: 

__[2] Concentration gradient driven molecular dynamics: a new method for simulations of membrane permeation and separation__ 
A Ozcan, C Perego, M Salvalaglio, M Parrinello, O Yazaydin
[Chem Sci 8 (5), 3858-3865, 2017](https://pubs.rsc.org/en/content/articlepdf/2017/sc/c6sc04978h)

The version hosted here has been developed by [Dr. Claudio Perego](https://scholar.google.co.uk/citations?user=TwqxhpUAAAAJ) and is maintained by the [Salvalaglio](http://www.ucl.ac.uk/molecular-modelling) and [Yazaydin](https://www.ucl.ac.uk/~ucecoya/) groups at [UCL Chemical Engineering](https://www.ucl.ac.uk/chemical-engineering/). 

The method has been further developed into a "cannibalistic" variant, and to model crystal nucleation at constant supersaturation by [Dr. Tarak Karmakar](https://scholar.google.co.uk/citations?hl=en&user=LWBFC34AAAAJ). 

### Installation

CMUMD is implemented as a PLUMED 2 Collective Variable (CV), the purpose of which is that of computing the density in a specific region of the simulation box. 
The derivatives of such variable are suitably modified to apply forces at the interface between a physical region of the simulation box and the system' reservoir.   

__NB:__ _CMUMD is defined as a CV but IT IS NOT an actual CV, the derivatives are formally wrong for any typical CV usage._

The forces are then applied by using the `RESTRAINT` keyword in PLUMED 2 (see the examples following). The method as implemented in this repository can be applied only to a planar geometry. 

__In order to install CmuMD:__ 

1. Download the file `Cmumd.cpp` 
2. Copy the source code in the PLUMED source directory:  
`cp CMUMD.cpp $PLUMED_DIR/src/colvar` 
3. [compile PLUMED 2](https://www.plumed.org/doc-v2.6/user-doc/html/_installation.html)


### Example 1: Concentration Driven Flux - methane permeation in a ZIF-8 membrane

This example reports the PLUMED file used to implement the method discussed in [Ozcan et al., Chem Sci, 2017](https://pubs.rsc.org/en/content/articlepdf/2017/sc/c6sc04978h). 
The input files for a typical simulation are available in `Examples/Example1_methane_permeation_in_ZIF-8_membrane`

```
GROUP ATOMS=5361-5785:1 LABEL=solute

CMUMD LABEL=LEFT GROUP=solute NSV=1 FIXED=0.350932603  DCR=0.087733  CRSIZE=0.087733 WF=0.00877 ASYMM=-1
CMUMD LABEL=RIGHT GROUP=solute NSV=1 FIXED=0.649067397  DCR=0.087733  CRSIZE=0.087733 WF=0.00877 ASYMM=1

RESTRAINT ARG=LEFT AT=1.03  KAPPA=8000.0 LABEL=lres
RESTRAINT ARG=RIGHT AT=0.0   KAPPA=500000.0 LABEL=rres

PRINT ARG=LEFT,RIGHT STRIDE=500 FILE=COLVAR_2
```


1. `GROUP` defines the group of atoms that are monitored to bias and monitor the density. More about the syntax of `GROUP` can be found in the [PLUMED 2 manual](https://www.plumed.org/doc-v2.6/user-doc/html/_g_r_o_u_p.html) 

2. `CMUMD LABEL=LEFT GROUP=solute NSV=1 FIXED=0.350932603  DCR=0.087733  CRSIZE=0.087733 WF=0.00877 ASYMM=-1`
	`CMUMD LABEL=RIGHT GROUP=solute NSV=1 FIXED=0.649067397  DCR=0.087733  CRSIZE=0.087733 WF=0.00877 ASYMM=1`
	
	These line computes the methane number density in a CR located at distance DCR from the left and right sides of the membrane slab. 
	
	__Keywords__:  
	* `GROUP`  =  the group of atoms for which the density will be controlled
	* `NSV`    =  (compulsory) number of atoms per solvent molecule
	* `SOLUTE` =  number of atoms per solute molecule
	* `DCR`    =  (compulsory) distance of the CR inner boundary from the left and right interfaces of the crystal slab (all lengths are in units of Z box size!!!). 
	* `CRSIZE`  =  (compulsory) width of the CR.  
	* `WF`      =  (compulsory) Fermi function length at the CR boundaries. 
	* `COF`     =  Cutoff for the Fermi function
	* `WIN`     =  Fermi function truncation distance inside the CR
	* `COIN`    =  Cutoff for the Fermi function inside the CR
	* `WOUT`    =  Fermi function truncation distance outside the CR
	* `COOUT`   =  Cutoff for the Fermi function outside the CR
	* `FIXED`   =  Fix the position of the interface
	* `NZ`      =  Number of histogram bins for the density in z
	* `NINT`    =  Density at the phase boundary (effective boundary condition)
	* `COMST`   =  Use solute centre of mass in the density calculation
	* `COMSV`   =  Use solvent centre of mass in the density calculation
	* `DELTA`   =  Difference in density in two different CR regions when using the asymmetric variant
	* `NOSCALE` =  Use absolute length units
 	* `ASYMM`   =  Indicates that this statement defines the concentration on the left side of the membrane slab, `ASYMM=-1` indicates the left side of the membrane slab, `ASYMM=1` indicates the right side of the membrane slab. 
	
3. `RESTRAINT ARG=LEFT AT=1.03  KAPPA=8000.0 LABEL=lres`
`RESTRAINT ARG=RIGHT AT=0.0   KAPPA=500000.0 LABEL=rres`  

	Impose an harmonic constraint on the concentration, in the left and right volumes. More about the syntax of `RESTRAINT` can be found in the [PLUMED 2 manual](https://www.plumed.org/doc-v2.6/user-doc/html/_r_e_s_t_r_a_i_n_t.html) 
 
4. `PRINT ARG=LEFT,RIGHT STRIDE=500 FILE=COLVAR_2` 
	Print statement. More about the syntax of `RESTRAINT` can be found in the [PLUMED 2 manual](https://www.plumed.org/doc-v2.6/user-doc/html/_p_r_i_n_t.html)


### Example 2: Concentration Driven Flux - methanol-water separation in an MFI membrane

This example illustrates the application of the CGD-MD method for the liquid phase separation of methanol-water mixture. [Madero-Castro et al., 2021](https://t.co/kRg0Jz4czS?amp=1). 
The input files are available in `Examples/Example2_methanol_water_separation_in_MFI_membrane`

### Example 3: Concentration Driven Flux - ternary gas separation in a ZIF-8 membrane (with LAMMPS)

This example illustrates the application of the CGD-MD method for the separation of a ternary gas mixture. [Namsani et al., 2021](https://dx.doi.org/10.1002/adts.201900120). 
The input files are available in `Example3_ternary_gas_separation_in_a_ZIF-8_membrane`


### Example 4: Concentration-dependent features of the NaCl double layer (with GROMACS 2018.6)

This example illustrates the application of the CmuMD method for investigating the structure of the solution / graphite interface. 
The input files are available in `Example4_NaCl_at_graphite`



 
