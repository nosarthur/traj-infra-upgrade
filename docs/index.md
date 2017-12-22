## highlights

Compared to the old trajectory infrastructure, the new one perform better (faster, more robust) in the following aspects

* tracking virtual sites (pseudo atoms)
* handling triclinic systems and orthorhombic systems whose primitive cell vectors do not align with the axes
* unwrapping coordinates around periodic boundary condition (automatically or manually)

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
```

* topo: `Cms` object manipulation, GID tracking/conversion
* traj and traj_util: definition of the new `Frame` object, trajectory read/write
* analysis: definition of various trajectory analyzers (see below)
* destro: the maeff class enables low-level mae file IO (try to avoid using it if you can)

## analyzers in the new trajectory infrastructure
* basic geometric operations
    * `Angle`
    * `Distance`
    * `Torsion`
    * `Vector`
* `CenterOf`
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
* `PosAlign`: align positions
    * `RMSD`: Root mean square deviation
        * `LigandRMSD`: further includes symmetry consideration
    * `RMSF`: Root mean square fluctuation
        * `ProteinRMSF`
    * ~~`CovarianceMatrix`~~
* `PosTrack`: track positions of selected atoms in a trajectory
* `ProtLigInter`: Protein-ligand interactions
    * `HydrophobicInter`, `_HydrophobicInter
    * `MetalInter`
    * `ProtLigHbondInter`
    * `ProtLigPiInter`
    * `ProtLigPolarInter`, `_ProtLigPolarInter`
    * `WaterBridges`
    * `WatLigFragDistance`, `_WatLigFragDistance`
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

In case you want to write customerized analyzer, I can share another document with you.
