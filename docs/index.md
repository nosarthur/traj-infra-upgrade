## highlights

Compared to the old trajectory infrastructure,
the new one performs better in the following aspects

* virtual sites (pseudo atoms) tracking
* support for triclinic systems and orthorhombic systems
* coordinates unwrapping around periodic boundary condition (PBC)
  either automatically or manually
* various bug fixes and speedup

## module changes

The following modules are deprecated
```python
schrodinger.trajectory
schrodinger.infra.desmond
schrodinger.application.desmond.destro
schrodinger.application.desmond.periodicfix
schrodinger.application.desmond.framesettools
schrodinger.application.desmond.generictrajectory
```

And the replacements are
```python
schrodinger.application.desmond.packages.topo
schrodinger.application.desmond.packages.traj
schrodinger.application.desmond.packages.traj_util
schrodinger.application.desmond.packages.analysis
schrodinger.application.desmond.packages.destro
schrodinger.application.desmond.packages.staf
```

* **topo**: `Cms` object manipulation, AID/GID conversion
* **traj** and **traj_util**: definition of the new trajectory `Frame`, trajectory read/write
* **analysis**: definition of various trajectory analyzers (see below)
* **destro**: the maeff class enables low-level mae file IO (try to avoid using it if you can, it bypasses all python-level and swig-level APIs)
* **staf**: abstract base classes for trajectory analyzers

## analyzers in the new trajectory infrastructure

* basic geometric operations
    * `Angle`
    * `Distance`
    * `PlanarAngle`
    * `Torsion`: dihedral angle
    * `Vector`
* `CenterOf`: weighted geometric center of a group of atoms (PBC aware)
    * `Centroid`
    * `CoC`: center of charge
    * `Com`: center of mass
* `Dipole`
* `Gyradius`: radius of gyration
* `HydrogenBondFinder`
* `MassAvgVel`: mass-averaged velocity
* `MolecularSurfaceArea`
* `MomentOfInertia`
* `OrderParameter`
    * the following analyzers are inputs to `OrderParameter`
        * `AxisDirector`
        * `DipoleDirector`
        * `LipidDirector`
            * sn1
            * sn2
            * all
        * `MomentOfInertiaDirector`
        * `SmartsDirector`
        * `SystemDipoleDirector`
* `PolarSurfaceArea`
* `PosAlign`: position alignment
    * `RMSD`: Root mean square deviation
        * `LigandRMSD`: further includes symmetry consideration
    * `RMSF`: Root mean square fluctuation
        * `ProteinRMSF`
    * ~~`CovarianceMatrix`~~
* `PosTrack`: track positions of selected atoms in a trajectory
* `ProtLigInter`: Protein-ligand interactions
    * `HydrophobicInter`
    * `MetalInter`
    * `ProtLigHbondInter`
    * `ProtLigPiInter`
    * `ProtLigPolarInter`
    * `WaterBridges`
    * `WatLigFragDistance`
* `ProtProtInter`: Protein-protein interactions
    * `ProtProtHbondInter`
    * `ProtProtPiInter`
    * `ProtProtSaltBridges`
* `Rdf`: Radial distribution function
* `Ramachandran`
* `SecondaryStructure`
* `SaltBridgeFinder`
* `SolventAccessibleSurfaceArea`
* `SolventAccessibleSurfaceAreaByResidue`
* `VolumeMapper`

