# traj-infra
Examples of transition to new trajectory infrastructure.

 | | old | new 
| --- | --- | --- 
| access structure | <pre>import schrodinger.trajectory.cmsstructure as cmsstructure<br>import schrodinger.structutils.analyze as analyze<br>cms_st = cmsstructure.read_cms(model_fname) <br>for a in cms_st.atom:<br>    print(a.index)<br>ligand_aids = analyze.evaluate_asl(cms_st, ligand_asl) </pre> | <pre>import schrodinger.application.desmond.packages.topo as topo<br>_, cms_model = topo.read_cms(model_fname) <br>for a in cms_model.atom:<br>    print(a.index)<br>ligand_aids = cms_model.select_atom(ligand_asl)</pre>
|  |  |
|  |  |


