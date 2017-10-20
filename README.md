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
|<pre>from schrodinger.trajectory.desmondsimulation import ChorusSimulation<br><br>csim = ChorusSimulation(cms_filename, trajectory_directory)<br>for frame_index in xrange(csim.total_frame):<br>    fr = csim.getFrame(frame_index) </pre> | <pre>import schrodinger.application.desmond.packages.traj as traj<br><br>tr = traj.read_traj(trajectory_directory)<br>for fr in tr:<br>    fr </pre>
|<pre>from schrodinger.infra import desmond<br><br>trj = desmond.generic_trajectory(TRJ_FILE)</pre>| 


