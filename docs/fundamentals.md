## introduction

The new trajectory infrastructure will do better with handling

* virtual sites (pseudo atoms)
* triclinic lattice
* unwrapping around periodic boundary condition automatically

Roughly speaking, here are the correspondences:

old | new
--- | ---
`DesmondSimulation` or `ChorusSimulation` object | `Cms` object and trajectory object
`_DesmondFrame` object | `traj.Frame` object
`schrodinger.infra.desmond.Trajectory` or `framesettools.Frameset` | python list of `traj.Frame` objects

Note that

* The new frame object is different from the old one.

## minimum examples

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

The `results` is a list of the analyzers' output.
In the above example, `results` is a list of 2 items, corresponding to the output of the two analyzers.

The analyzers' output format varies.
Typically, the output is a list of frame-wise results.

## atom AIDs and atom GIDs

In MD simulation, there are two types of atoms

* physical atoms
* pseudo atoms (virtual sites): They are used to model forcefield better.

In Maestro, the pseudo atoms are not displayed.

To keep track of the atoms, there are two types of IDs

* AID stands for Atom ID, defined as the atom index in the full system ct (it is the same as `cms_model.atom.index`).
* GID stands for Global ID, defined as the particle index in the Desmond internal particle array.

Note that pseudo atoms do not have AID.

To access a particle's coordinate in the trajectory (i.e., the `traj.Frame` object), you have to use GID. We have functions to map AID to GID in the `topo` module. For example,

```python
import schrodinger.application.desmond.packages.topo as topo
import schrodinger.application.desmond.packages.traj as traj

_, cms_model = topo.read_cms(cms_filename)
protein_aids = cms_model.select_atom(protein_asl)
protein_gids = topo.aids2gids(cms_model, protein_aids,
                              include_pseudoatoms=False)
tr = traj.read_traj(trajectory_directory)
for fr in tr:
    fr.pos(protein_gids)
    fr.vel(protein_gids)
```
Both `traj.Frame.pos()` and `traj.Frame.vel()` return Nx3 numpy arrays. Without argument input, they return the positions or velocities of all atoms.

Note that

* `vel()` may not exist. Default MD simulation does not save velocities, also the XTC format trajectories do not contain velocities.
* If you get positions this way, then you need to worry about unwrapping around PBC yourself. If you don't want to unwrap yourself, try use the existing mechanisms in the infrastructure. See paradigms below.

ASL and SMARTS evaluations only return AIDs. In other words, they only select physical atoms.


