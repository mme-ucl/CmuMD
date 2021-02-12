# CmuMD 

This repository hosts the implementation for [PLUMED 2](https://www.plumed.org) of the CmuMD method, which allows to mimic open boundary conditions in NVT molecular simulations involving solid/fluid interfaces, and enables the simulation of concentration-driven fluxes through porous membranes. 

The method, with an application to crystal growth problems is described in the paper: 

* __Molecular dynamics simulations of solutions at constant chemical potential__  
C Perego, M Salvalaglio, M Parrinello, [J Chem Phys (14), 144113, 2015](https://moodle.ucl.ac.uk/course/view.php?id=1191), [arxiv](https://arxiv.org/abs/1501.07825)

The adaptation of CmuMD to model concentration driven membrane fluxes is described in the paper: 

* __Concentration gradient driven molecular dynamics: a new method for simulations of membrane permeation and separation__  
A Ozcan, C Perego, M Salvalaglio, M Parrinello, O Yazaydin [Chem Sci 8 (5), 3858-3865, 2017](https://pubs.rsc.org/en/content/articlehtml/2017/sc/c6sc04978h)

The version hosted here has been developed by [Dr. Claudio Perego](https://scholar.google.co.uk/citations?user=TwqxhpUAAAAJ) and is maintained by the [Salvalaglio](http://www.ucl.ac.uk/molecular-modelling) and [Yazaydin](https://www.ucl.ac.uk/~ucecoya/) groups at [UCL Chemical Engineering](https://www.ucl.ac.uk/chemical-engineering/). 

The method has been further developed in the Parrinello group by [Dr. Tarak Karmakar](https://scholar.google.co.uk/citations?hl=en&user=LWBFC34AAAAJ). Further developments include a "cannibalistic" variant, and a spherical geometry which enables to model crystal nucleation at constant supersaturation. 


## Usage  

CMUMD is defined as a PLUMED 2 Collective Variable (CV), the purpose of which is that of performing CmuMD.
CMUMD is implemented using the RESTRAINT keyword (performing CmuMD), or  monitored. The method as it is implemented here can be applied only to a planar geometry. 

__NB:__ _CMUMD is defined among CVs in PLUMED, but IT IS NOT an actual CV. Derivatives are incorrect for any typical CV use._

__To use CmuMD, copy CMUMD.cpp in the $PLUMED_DIR/src/colvar directory and compile plumed__

Example applications to solid/liquid interfaces and fluxes through porous materials will be added to this repository. 
















 
