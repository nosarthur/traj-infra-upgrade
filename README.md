# traj-infra
Examples of transition to new trajectory infrastructure.

 | | old | new 
| --- | --- | --- 
| access structure | <pre>import schrodinger.trajectory.cmsstructure as cmsstructure<br>import schrodinger.structutils.analyze as analyze<br><br>cms_st = cmsstructure.read_cms(model_fname) <br>for a in cms_st.atom:<br>    print(a.index)<br>ligand_aids = analyze.evaluate_asl(cms_st, ligand_asl) </pre> | <pre>import schrodinger.application.desmond.packages.topo as topo<br><br>_, cms_model = topo.read_cms(model_fname) <br>for a in cms_model.atom:<br>    print(a.index)<br>ligand_aids = cms_model.select_atom(ligand_asl)</pre>
| access trajectory | <pre>from schrodinger.trajectory.desmondsimulation import ChorusSimulation<br><br>tr = ChorusSimulation(cms_filename, trajectory_directory)<br>for frame_index in xrange(tr.total_frame):<br>    fr = tr.getFrame(frame_index) </pre> | <pre>import schrodinger.application.desmond.packages.topo as topo<br>import schrodinger.application.desmond.packages.traj as traj<br><br>tr = traj.read_traj(trajectory_directory)<br>for fr in tr:<br>    fr </pre>
|  |  |


