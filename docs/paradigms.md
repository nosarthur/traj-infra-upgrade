## load both cms file and trajectory in case trajectory path is not specified

Ideally, the user should specify the paths for both the cms file and the trajectory file.
In case the trajectory path is missing, one should make sure

1. the cms file and trajectory file are in the same directory
1. the `s_chorus_trajectory_file` property in the cms file points to the relative path of the trajectory file

```python
import schrodinger.application.desmond.packages.topo as topo
import schrodinger.application.desmond.packages.traj as traj

msys_model, cms_model = topo.read_cms(args.cms_file)
trj_path = topo.find_traj_path(cms_model, os.path.dirname(args.cms_file))
if trj_path is None:
    parser.error('Could not locate a trajectory directory for given CMS file')
try:
    trj = traj.read_traj(trj_path)
except Exception as e:
    parser.error('Cannot load trajectory file: %s' % e)
```

For GUI, often times one only needs to get the trajectory path from the cms file path, then the following code can be used

```python
import schrodinger.application.desmond.packages.topo as topo

trj_path = topo.find_traj_path_from_cms_path(cms_path)
```

Note one should still check whether `trj_path` is `None` and do `try except` for `traj.read_traj`.

## extract structure once and per frame update coordinates instead of per frame update full system ct and extract structure
Note that although one can get the full system ct per frame in new trajectory infrastructure, it is likely the inefficient approach.
In most cases, the user does not need to track the full system ct over the frames.
Instead, only specific group of atoms or molecules needs to be tracked over the frames.
In these situations, it is more efficient to

* extract the atoms or molecules into a structure once
* keep updating (or maybe only read) the coordinates of the selected atoms or molecules frame by frame

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

## try avoid unwrap coordinates around periodic boundary conditions yourself

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

## analyze all together instead of one by one
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

## testing

The new trajectory modules are in the desmond package.
Thus if you are working on a non-desmond repo and uses trajectory stuff, test has to be written in the following way

* use `test_marker.require_product('desmond')`
* lazy import the trajectory handling modules


```python
import pytest

# Lazy import, in case desmond is not installed
mf = None

@pytest.fixture(scope='module', autouse=True)
def import_my_file():
    global mf
    import my_file as mf
```
