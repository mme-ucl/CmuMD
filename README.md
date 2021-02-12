# This is the CmuMD repository

This repository hosts the implementation for PLUMED 2 of the CmuMD method, which allows to mimic open boundary conditions in NVT molecular simulations involving solid/fluid interfaces, and enables the simulation of concentration-driven fluxes through porous membranes. 

The method, with an application to crystal growth problems is described in the paper: 

__Molecular dynamics simulations of solutions at constant chemical potential__ 
C Perego, M Salvalaglio, M Parrinello, [J Chem Phys (14), 144113, 2015](https://moodle.ucl.ac.uk/course/view.php?id=1191), [arxiv](https://arxiv.org/abs/1501.07825)

The adaptation of CmuMD to model concentration driven membrane fluxes is described in the paper: 

__Concentration gradient driven molecular dynamics: a new method for simulations of membrane permeation and separation__
A Ozcan, C Perego, M Salvalaglio, M Parrinello, O Yazaydin
Chem Sci 8 (5), 3858-3865

The version hosted here has been developed by [Dr. Claudio Perego](https://scholar.google.co.uk/citations?user=TwqxhpUAAAAJ) and is maintained by the [Salvalaglio](http://www.ucl.ac.uk/molecular-modelling) and [Yazaydin](https://www.ucl.ac.uk/~ucecoya/) groups at [UCL Chemical Engineering](https://www.ucl.ac.uk/chemical-engineering/). 

The method has been further developed into a "cannibalistic" variant, and to model crystal nucleation at constant supersaturation by [Dr. Tarak Karmakar](https://scholar.google.co.uk/citations?hl=en&user=LWBFC34AAAAJ). 


__Usage__ 

CMUMD is defined as a PLUMED 2 Collective Variable (CV), the purpose of which is that of performing CmuMD.
CMUMD computes the number density of a group of atoms in a planar shell. 
The computed density can be restrained through PLUMED-2 RESTRAINT keyword (performing CmuMD), or  monitored (see example below).

__NB:__ _CMUMD is defined as a CV but IT IS NOT an actual CV, the derivatives are formally wrong for any typical CV usage._

The method as it is can be applied only to a planar geometry. 

__To use, copy CMUMD.cpp in the $PLUMED_DIR/src/colvar directory and compile plumed__

Follows a brief explanation of the available keywords based on an example. The reader should already be familiar with the usage of PLUMED 2.


_SYSTEM_:

Consider a Urea slab in water-urea solution. We define 2 groups:

`<GROUP ATOMS=1-9599:1 LABEL=all   #all urea + water atoms>`
`<GROUP ATOMS=5001-9599:1 LABEL=water   #all water atoms>`


_MONITORING THE SOLVENT (water) CR DENSITY:

`<CMUMD LABEL=n_water GROUP=water NSV=3 DCR=0.0715 CRSIZE=0.1215 WF=0.00000715 NINT=10.0 NZ=188>`

This line computes the water number density in a CR located at distance DCR from both sides of the crystal slab. The CR is composed by two equal regions placed at both sides (left and right) of the crystal slab.

keywords:
- GROUP  =  the solvent atom group
- NSV    =  number of atoms per solvent molecule
- DCR    =  distance of the CR inner boundary from the left and right interfaces of the crystal slab (all lengths are in units of Z box size!!!)
- CRSIZE =  thickness of the CR (on each side of the crystal).
- WF     =  Fermi function length at the CR boundaries
- NINT   =  Number density at the interface (nm^-3)
- NZ 	 =  Number of bins for the histogram of the interface-finding algorithm

__Observations__:

1------
The algorithm loops on the molecules contained in GROUP and check if they belong to the CR. A combination of Fermi switching functions is used to count 1 for the molecules contained in the CR and count 0 for the others. At the boundaries of the CR intermediate values are possible, depending on the Fermi switching function length considered.

Two possible Fermi function lengths are defined, WIN and WOUT. WIN is relative to the inner boundaries of the CR, those facing the crystal.
WOUT is relative to the outer boundaries, those facing the reservoir region.
A 3rd length, WF, is that of the localizing function G, Eq. (4) of the ref. paper. If no WIN and WOUT are defined (as in the example line) they all are automatically set equal to WF (which is a compulsory keyword). Generally the boundary of the CR can be sharp, therefore WIN and WOUT can be small, while WF cannot be too small in order to contain the magnitude of the external forces restraining the concentration (see Ref. paper and supporting info). In this case WF is small since the variable is used only for monitoring purposes.
The Fermi functions are by default cut-off at 20.0 lengths, if one wants to change this settings the keywords COF, COIN, COOUT (one for each lenght) can be used. The default setting is COF=COIN=COOUT=20.0.

2------
The code computes the center of mass of the molecules to define their position. It is possible to change this setting through the COMSV keyword:
COMSV = -1 (default) use the center of mass
COMSV = 0 use the first atom of the molecule to define its position
COMSV = n use the n-th atom of the molecule to define its position (the order is that of the list provided in the GROUP, in this case is O-H1-H2)

NOTE THAT:
A: of course it is possible to provide plumed only with the atoms that define the molecule position, e.g. all the Oxygen atoms. Then one should put NSV=1, and the result would be the same as using COMSV=0.
B: If one wants to use the center of mass of the molecules it is also possible to define a GROUP formed by the centers of mass of all the solvent molecules, see plumed 2 COM documentation. This (with NSV = 1) would correspond to the default COMSV=-1.
Both A and B are preferable, since they exploit the PLUMED 2 features and are therefore more reliable and probably faster.

3-------
The two regions composing the CR are placed at a fixed distance from both interfaces of the crystal. To this purpose the code has a simple algorithm that locates the two interfaces of the crystal slab. In order to do that one has to provide NINT and NZ. The interface is located by building an histogram of the local number density of the solvent molecules. This approach is based on the idea that the crystal is not formed by solvent molecules, if that's not the case the algorithm should be modified.
This histogram is built dividing the box in NZ bins (in the z direction which is the one of the crystal growth). Then the solvent density in each bin is computed and the interfaces are located  where the density crosses the threshold value NINT.
In this example we divide the box in 188 bins and locate the interfaces where the density of the solvent crosses 10.0 nm^-3.
The algorithm looks for the slab starting from the center of the box.
The criteria to locate the slab is finding 3 consecutive bins with solvent density less than NINT/3. This is arbitrary, it might need to be changed for different systems.
When the slab is located then the method looks for the 2 interfaces and sets the position of the CR accordingly.

For this algorithm to work properly, it has to account for all the solvent molecules, not just part of them.
It is possible to use the FIXED keyword to place a virtual fixed interface in the box (the same for both the left and right CR regions). The CR will be then defined accordingly and remain fixed no matter what (e.g. FIXED=0.5 fixes the interface in the middle of the box).


*MONITORING THE SOLUTE (urea) CR DENSITY:

CMUMD LABEL=n_urea GROUP=all NSV=3 SOLUTE=5000 NST=8 DCR=0.0715 CRSIZE=0.1215 WF=0.00000715 NINT=10.0 NZ=188

Compared to the previous line this accounts for all the atoms (GROUP=all) and contains 2 new keywords.

SOLUTE=5000 The total number of solute atoms in this case is 625 urea molecules
NST=8 the number of atoms per solute molecules (urea = 8)

The rest is equal to the previous example line.

Observations:

1--------
To compute the solute concentration we provide all the atoms, not just the solute. Moreover the number of atoms per solvent molecules (NSV) is provided as well.
This is because the density criteria to locate the interfaces still uses the solvent distribution.
If one uses FIXED then this can be avoided.

2--------
The same scheme as for the solvent density is used to define the position of the molecules. The keyword to change the center of mass settings is COMST (instead of COMSV)
Also in this case the considerations about using PLUMED center-of-mass groups or directly passing the position-defining atoms to plumed hold.
e.g. in the case of urea providing only the C atoms (with NST =1), which is pretty close to the center of mass, might be the smartest strategy.



*RESTRAINING THE SOLUTE/SOLVENT CR DENSITY:

CMUMD LABEL=n_water GROUP=water NSV=3 DCR=0.0715 DF=0.23 CRSIZE=0.1215 WF=0.02 WOUT=0.00000715 WIN=0.00000715 NINT=10.0 NZ=188
CMUMD LABEL=n_urea GROUP=all NSV=3 SOLUTE=5000 NST=8 DCR=0.0715 DF=0.23 CRSIZE=0.1215 WF=0.02 WOUT=0.00000715 WIN=0.00000715  NINT=10.0 NZ=188

RESTRAINT ARG=n_water AT=29.5 KAPPA=20.0 LABEL=wres	 #water density restrain
RESTRAINT ARG=n_urea AT=1.7 KAPPA=100.0 LABEL=ures	#urea density restrain

The two CMUMD lines compute the densities in the defined CRs, while the two RESTRAINT lines apply an harmonic potential to their value in order to control de concentration (check PLUMED manual for insights on the RESTRAINT keyword).
Actually the force is non-conservative and a potential cannot be defined, the correct formula for the force is Eq.1 of the paper.

Observations:
1---------
A further keyword is used in the two CMUMD lines:
DF = 0.23 is the distance from the interfaces (still in box lengths) at which the force is located at both sides of the crystal. By default DF = DCR+CRSIZE, that is at the outer boundary of the CR. If one sets DF < DCR+CRSIZE the code sets it equal to DCR+CRSIZE by default, the only choice is DF > DCR+CRSIZE. AS explained in the paper this is necessary to decouple the reservoir from the CR.

2---------
We have set WF larger than WIN and WOUT, as explained above in order to contain the module of the applied force.

3---------
The KAPPA of RESTRAINT lines does not correspond to the k of eq.1 of the paper. There should be a factor 2/VCR ( k = KAPPA*2/VCR) where VCR is the volume of the CR (left+right region).

4---------
As explained in the reference paper we tipically restrain only one species and let the barostat action balance the concentration of the other.

5---------
The virial due to the applied CmuMD force is not computed, therefore the contribution of this force to the pressure equilibration is not provided. In the considered case this contribution can be neglected but that is not guaranteed for any kind of system.












 