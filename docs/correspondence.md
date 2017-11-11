## access structure
old
```python
import schrodinger.trajectory.cmsstructure as cmsstructure
import schrodinger.structutils.analyze as analyze

cms_st = cmsstructure.read_cms(model_fname)
for a in cms_st.atom:
    print(a.index)
ligand_aids = analyze.evaluate_asl(cms_st, ligand_asl)
```
new
```python
import schrodinger.application.desmond.packages.topo as topo

_, cms_model = topo.read_cms(model_fname)
for a in cms_model.atom:
    print(a.index)
ligand_aids = cms_model.select_atom(ligand_asl)
```

Note that `analyze.evaluate_asl` does not respect PBC whereas `cms_model.select_atom` does.

## access trajectory

old
```python
from schrodinger.trajectory.desmondsimulation import ChorusSimulation
from schrodinger.infra import desmond

tr = desmond.generic_trajectory(trajectory_directory)
dt = tr.frame_time(1) - tr.frame_time(0)

csim = ChorusSimulation(cms_filename, trajectory_directory)
for frame_index in xrange(csim.total_frame):
    fr = csim.getFrame(frame_index)
    st = fr.getStructure()
```

new
```python
import schrodinger.application.desmond.packages.traj as traj
import schrodinger.application.desmond.packages.topo as topo

_, cms_model = topo.read_cms(model_fname)
tr = traj.read_traj(trajectory_directory)
dt = tr[1].time - tr[0].time
for fr in tr:
    st = topo.update_fsys_ct(cms_model, fr).fsys_ct
    # you can use st = topo.update_fsys_ct(cms_model, fr) as well
    # since the properties of cms_model and its fsys_ct are in sync

```
Note that

* although there is a correspondence of `st = fr.getStructure()`, it is likely not the way to go. See [this paradigm](/paradigms/#extract-structure-once-and-per-frame-update-coordinates-instead-of-per-frame-update-full-system-ct-and-extract-structure).
* there are two ways to update the `cms_model`
```python
updated_cms = topo.update_fsys_ct(cms_model, fr)
updated_cms = topo.update_cms(cms_model, fr)
```

The difference is that `update_cms` not only updates full system ct, but also component cts.
