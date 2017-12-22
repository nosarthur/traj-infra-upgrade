## introduction

Roughly speaking, here are the correspondences:

old | new
--- | ---
`DesmondSimulation` or `ChorusSimulation` object | `Cms` object and trajectory object
`_DesmondFrame` object | `traj.Frame` object
`schrodinger.infra.desmond.Trajectory` or `framesettools.Frameset` | python list of `traj.Frame` objects

Note that

* The new frame object is different from the old one.
* The following properties and function calls are guaranteed for a frame object
    * `fr.natoms`
    * `fr.pos()`
    * `fr.time`
    * `fr.box`
* The function call `fr.vel()` is not guaranteed. XTC data will not have this, DTR data may not have it either.
* If the data is DTR, you may be able to pull out more (private) information such as `fr._frame().temperature`. Use with your own risk.

## minimum examples

To read the CMS input file
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
* pseudo atoms (virtual sites): They have fractional charge and 0 mass. They are used to model forcefield better.

In Maestro, the pseudo atoms are not displayed.

To keep track of the atoms, there are two types of IDs

* AID stands for Atom ID, defined as the atom index in the full system ct (it is the same as `cms_model.atom.index`). It starts with 1.
* GID stands for Global ID, defined as the particle index in the Desmond internal particle array. It starts with 0.

Note that

* pseudo atoms do not have AID.
* sometimes `AID = GID + 1`, but it is not always the case

To access a particle's coordinate in the trajectory (i.e., the `traj.Frame` object), you have to use GID.
We have functions to map AID to GID in the `topo` module. For example,

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
Both `traj.Frame.pos()` and `traj.Frame.vel()` return Nx3 numpy arrays,
Without argument input, they return the positions or velocities of all atoms.
With GIDs input, the order of the output follows the order of the GID input.
For example 

```
In [8]: fr0.pos([2,3])
Out[8]:
array([[-4.77608204,  3.04622388,  1.31021202],
       [-5.66208792,  1.88501239,  0.96774006]], dtype=float32)

In [9]: fr0.pos([3,2])
Out[9]:
array([[-5.66208792,  1.88501239,  0.96774006],
       [-4.77608204,  3.04622388,  1.31021202]], dtype=float32)
```

One can use the GID directly to get the single atom coordinate too
```
In [7]: fr0.pos(0)
Out[7]: array([-0.76461941, -2.64227247, -0.62481445], dtype=float32)
```


Note that

* `vel()` may not exist. Default MD simulation does not save velocities, also the XTC format trajectories do not contain velocities.
* If you get positions this way, then you need to worry about unwrapping around PBC yourself. If you don't want to unwrap yourself, try use the existing mechanisms in the infrastructure. See [this paradigm](/paradigms/#try-avoid-unwrap-coordinates-around-periodic-boundary-conditions-yourself).

ASL and SMARTS evaluations only return AIDs. In other words, they only select physical atoms.
There are two helper functions in the topo module to select GIDS:
```
topo.asl2gids()
topo.aids2gids()
```
