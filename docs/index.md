## module changes

The modules to be deprecated are
```python
schrodinger.trajectory
schrodinger.infra.desmond
schrodinger.application.desmond.destro
schrodinger.application.desmond.periodicfix
schrodinger.application.desmond.framesettools
schrodinger.application.desmond.generictrajectory
```

The replacements are
```python
schrodinger.application.desmond.packages.topo
schrodinger.application.desmond.packages.traj
schrodinger.application.desmond.packages.analysis
(the new destro)
```

## existing analyzers in the new trajectory infrastructure
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
