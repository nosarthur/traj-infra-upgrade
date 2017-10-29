# traj-infra
Examples of transition to new trajectory infrastructure. 
I will keep updating this page as more code is ported.

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

## introduction

The new trajectory infrastructure will do better with handling

* virtual sites (pseudo atoms)
* triclinic lattice

Roughly speaking, here are the correspondences:

old | new 
--- | --- 
`DesmondSimulation` or `ChorusSimulation` object | `Cms` object and trajectory object
`_DesmondFrame` object | `traj.Frame` object
`schrodinger.infra.desmond.Trajectory` or `framesettools.Frameset` | python list of `traj.Frame` objects

Note that 

* The new frame object is different from the old one. 

### minimum examples

To read the cms input file
```python
import schrodinger.application.desmond.packages.topo as topo

msys_model, cms_model = topo.read_cms(cms_file_name)
```

To read a trajectory
```python
import schrodinger.application.desmond.packages.traj as traj

tr = traj.read_traj(trajectory_directory)
```

To do some analysis using existing analyzers

```python
from schrodinger.application.desmond.packages import analysis, traj, topo                        

# load data                                                                        
msys_model, cms_model = topo.read_cms(FNAME)                                       
tr = traj.read_traj(TRJ_FNAME) 

# define analyzers                                                                 
analyzer1 = analysis.Com(msys_model, cms_model, asl='m.n 1')                      
analyzer2 = analysis.ProtLigInter(msys_model, cms_model, 'protein', 'm.n 2')
                                                                                   
# compute result                                                                   
results = analysis.analyze(tr, analyzer1, analyzer2, )  
```

The `results` is a list of the analyzer's output. 
Each output is a list of results for each trajectory frame.

### atom AIDs and atom GIDs

In MD simulation, there are two types of atoms

* physical atoms
* pseudo atoms (virtual sites): They are used to model forcefield better.

In Maestro, the pseudo atoms are not displayed.

To keep track of the atoms, there are two types of IDs

* AID stands for Atom ID, defined as the atom index in the full system ct (it is the same as `cms_model.atom.index`).
* GID stands for Global ID, defined as the particle index in the Desmond internal particle array.

Note that pseudo atoms do not have AID.

To access a particle's coordinate in the trajectory (i.e., the `traj.Frame` object), you have to use GID. We have functions to map AID to GID in the `topo` module.

ASL and SMARTS evaluations only return AIDs. In other words, they only select physical atoms.

## examples 

### access structure
| old | new 
| --- | --- 
| <pre>import schrodinger.trajectory.cmsstructure as cmsstructure<br>import schrodinger.structutils.analyze as analyze<br><br>cms_st = cmsstructure.read_cms(model_fname)<br>for a in cms_st.atom:<br>    print(a.index)<br>ligand_aids = analyze.evaluate_asl(cms_st, ligand_asl) </pre> | <pre>import schrodinger.application.desmond.packages.topo as topo<br><br>_, cms_model = topo.read_cms(model_fname)<br>for a in cms_model.atom:<br>    print(a.index)<br>ligand_aids = cms_model.select_atom(ligand_asl)</pre>
 
### access trajectory 

|old | new
| --- | --- 
|<pre>from schrodinger.trajectory.desmondsimulation import ChorusSimulation<br><br>csim = ChorusSimulation(cms_filename, trajectory_directory)<br>for frame_index in xrange(csim.total_frame):<br>    fr = csim.getFrame(frame_index)<br>    st = fr.getStructure()</pre> | <pre>import schrodinger.application.desmond.packages.traj as traj<br>import schrodinger.application.desmond.packages.topo as topo<br><br>_, cms_model = topo.read_cms(model_fname)<br>tr = traj.read_traj(trajectory_directory)<br>for fr in tr:<br>    st = topo.update_fsys_ct(cms_model, fr).fsys_ct<br>    # you can use st = topo.update_fsys_ct(cms_model, fr) as well<br>    # the properties of cms_model and its fsys_ct are in sync  </pre>
|<pre>from schrodinger.infra import desmond<br><br>tr = desmond.generic_trajectory(TRJ_FILE)<br>dt = tr.frame_time(1) - tr.frame_time(0)</pre>| <pre>import schrodinger.application.desmond.packages.traj as traj<br><br>tr = traj.read_traj(trajectory_directory)<br>dt = tr[1].time - tr[0].time</pre>

Note that 

* although there is a correspondence of `st = fr.getStructure()`, it is likely not the way to go. See paradigms below.

## paradigms

### extract structure once and per frame update coordinates instead of per frame update full system ct and extract structure 
Note that although one can get the full system ct per frame in new trajectory infrastructure, it is likely the inefficient approach. In most cases, the demand is not to track the full system ct over the frames, but track some specific group of atoms or molecules over the frames. In these situations, it is more efficient to 

* extract the structure once
* keep updating the coordinates of the structure frame by frame

If the analysis is fully geometric, then the structure extraction can be avoided as well.

Here is an example

```python
import schrodinger.application.desmond.packages.topo as topo
import schrodinger.application.desmond.packages.traj as traj

_, cms_model = topo.read_cms(cms_filename)
tr = traj.read_traj(trajectory_directory)
protein_aids = cms_model.select_atom(protein_asl)
protein_gids = topo.aids2gids(cms_model, protein_aids,
                              include_pseudoatoms=False)
protein_st = cms_model.extract(protein_aids)
for fr in tr:
    protein_st.setXYZ(fr.pos(protein_gids))
    # what needs to be done on the protein structure
    ...
```

This is better than

```python
_, cms_model = topo.read_cms(cms_filename)
tr = traj.read_traj(trajectory_directory)
protein_aids = cms_model.select_atom(protein_asl)
for fr in tr:
    updated_cms = topo.update_fsys_ct(cms_model, fr)
    protein_st = updated_cms.extract(protein_aids)
    # what needs to be done on the protein structure
    ...
```

### try avoid unwrap coordinates around periodic boundary conditions yourself

There are mechanisms in the new trajectory infrastructure to do these unwrapping for you. The relevant analyzer classes are

* basic geometric operations                                                    
    * `Angle`                                                                   
    * `Distance`                                                                
    * `Torsion`                                                                 
    * `Vector`                                                                  
* `CenterOf`                                                                    
    * `Centroid`                                                                
    * `CoC`: center of charge                                                   
    * `Com`: center of mass  
    
    
You can even measure geometric quantity between (among) center of mass objects/atoms (see example below), etc. 

```python
from schrodinger.application.desmond.packages import analysis                      
from schrodinger.application.desmond.packages import traj                          
from schrodinger.application.desmond.packages import topo 

# load data                                                                        
msys_model, cms_model = topo.read_cms(FNAME)                                       
tr = traj.read_traj(TRJ_FNAME) 

# define analyzers                                                                 
analyzer1 = analysis.Com(msys_model, cms_model, asl=my_asl)                      
ana_vector = analysis.Vector(msys_model, cms_model, 1, 10)
analyzer3 = analysis.ProtLigInter(msys_model, cms_model, 'protein', 'm.n 2')
s5 = analysis.Com(self.msys_model, self.cms_model, asl='not atom.num 17')  # this doesn't need to be passed to analysis.analyze
dist_com_atm = analysis.Distance(self.msys_model, self.cms_model, s5, 10)
                                                                                   
# compute result                                                                   
results = analysis.analyze(tr, analyzer1, ana_vector, analyzer3, dist_com_atm)  
```

If you have to unwrap yourself, use the `analysis.Pbc` class. Also note that the simulation box may change from frame to frame, e.g. in NPT systems. Thus the `analysis.Pbc` object needs to be updated frame by frame.

Note that the [circular mean algorithm](https://en.wikipedia.org/wiki/Center_of_mass#Systems_with_periodic_boundary_conditions) is used to calculate geometric centers.
Thus it will fail if both of the following conditions apply
* the selected atoms have a spatial extent comparable to the simulation box
* the selected atoms do not form a blob-like shape (e.g., dumbbell)

### analyze all together instead of one by one
If multiple analyzers can share some intermediate calculations, 
there is a good chance that the new trajectory analysis framework calculates these intermediate results only once.
Thus it is more efficient to call `analysis.analyze` with all analyzers together instead of calling `analysis.analyze` multiple times.

Thus 
```python
results = analysis.analyze(tr, analyzer1, analyzer2, analyzer3)  
```

is better than
```python
result1 = analysis.analyze(tr, analyzer1)
result2 = analysis.analyze(tr, analyzer2)
result3 = analysis.analyze(tr, analyzer3)  
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
    * `HydrophobicInter`, `_HydrophobicInter`                                   
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
