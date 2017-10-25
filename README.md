# traj-infra
Examples of transition to new trajectory infrastructure. 
I will keep updating this page as more code is refactored.

## introduction

The new trajectory infrastructure will do better on

* virtual sites (pseudo atoms)
* triclinic lattice



Roughly speaking, here are the correspondences:

old | new 
--- | --- 
`DesmondSimulation` or `ChorusSimulation` object | `Cms` object and trajectory object
`_DesmondFrame` object | `traj.Frame` object
desmond.generic_trajectory | python list of `traj.Frame` objects

Note that 

* The new frame object is different from the old one. 


## examples 

### access structure
| old | new 
| --- | --- 
| <pre>import schrodinger.trajectory.cmsstructure as cmsstructure<br>import schrodinger.structutils.analyze as analyze<br><br>cms_st = cmsstructure.read_cms(model_fname)<br>for a in cms_st.atom:<br>    print(a.index)<br>ligand_aids = analyze.evaluate_asl(cms_st, ligand_asl) </pre> | <pre>import schrodinger.application.desmond.packages.topo as topo<br><br>_, cms_model = topo.read_cms(model_fname)<br>for a in cms_model.atom:<br>    print(a.index)<br>ligand_aids = cms_model.select_atom(ligand_asl)</pre>
 
### access trajectory 

|old | new
| --- | --- 
|<pre>from schrodinger.trajectory.desmondsimulation import ChorusSimulation<br><br>csim = ChorusSimulation(cms_filename, trajectory_directory)<br>for frame_index in xrange(csim.total_frame):<br>    fr = csim.getFrame(frame_index)<br>    st = fr.getStructure()</pre> | <pre>import schrodinger.application.desmond.packages.traj as traj<br>import schrodinger.application.desmond.packages.topo as topo<br><br>_, cms_model = topo.read_cms(model_fname)<br>tr = traj.read_traj(trajectory_directory)<br>for fr in tr:<br>    st = topo.update_fsys_ct(cms_model, fr).fsys_ct </pre>
|<pre>from schrodinger.infra import desmond<br><br>tr = desmond.generic_trajectory(TRJ_FILE)<br>dt = tr.frame_time(1) - tr.frame_time(0)</pre>| <pre>import schrodinger.application.desmond.packages.traj as traj<br><br>tr = traj.read_traj(trajectory_directory)<br>dt = tr[1].time - tr[0].time</pre>

## paradigms

### extract structure once and per frame update coordinates (instead of per frame extract structure and update coordinates)
Note that although one can get the full system ct per frame in new trajectory infrastructure, it is likely the inefficient approach. In most cases, the real application is not to track the full system ct over the frames, but track some specific group of atoms or molecules over the frames. In these situations, it is more efficient to 

* extract the structure once
* keep updating the coordinates of the structure frame by frame

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

If you have to unwrap yourself, use `analysis.Pbc` class.


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
