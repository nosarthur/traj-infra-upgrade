
## overview

This section is for developers who want to write customized trajectory analyzers.
It can be skipped if you are satisfied as a user of the existing analyzers.

The main reason for writing one's own analyzer is code decoupling.

As an introductory example, let's filter a trajectory according to a user
selected distance cutoff between two atoms.


```python
# file loading is omitted 
dist_ana = analysis.Distance(msys_model, cms_model, 1, 50)  # atom 1 and 50 are chosen
distances = analysis.analyze(tr, dist_ana)

out_tr = []
for fr, d in zip(tr, distances):
    if d < 4.0:  # 4.0 is the chosen cutoff
        out_tr.append(fr)
```

The following class is an example of customized analyzer for the same task,

```python
class Spliter(staf.GeomAnalyzerBase):
    def __init__(self, msys_model, cms_model, atom1, atom2, cutoff=1):
        self.gids = topo.aids2gids([atom1, atom2], include_pseudoatoms=False)
    
    def _precalc(self, calc):
        calc.addDistance(self.gids[0], self.gids[1])

    def _postcalc(self, calc, pbc, fr):
        d = calc.getDistance(self.gids[0], self.gids[1])
        if d < self.cutoff:
            self.result = fr
```

With this customized analyzer, the usage code becomes

```python
# file loading is omitted 
spliter = Spliter(msys_model, cms_model, 1, 50, cutoff=4.0)
out_tr = analysis.analyze(tr, spliter)
```

## design

At high level, the trajectory analysis process can be summarized by the following pseudo-code

``` python
calc = GeomCalc()

# each analyzer is an instance of a derived class of GeomAnalyzerBase
for ana in analyzers:
    # inside these _precalc() functions, elementary calculations are registered to
    # calc._custom which is a _CustomCalc object
    # these results are cached and shared among analyzers
    ana._precalc(calc)

for fr in tr:
    # registered elementary calculations are performed by this call
    calc()
    # further analyzer specific calculations
    for ana in analyzers:
        ana._postcalc(calc, fr)

# this reduce process does not exist for all analyzers
for ana in analyzers:
    ana.reduce()
```


The design of trajectory analysis framework is a combination of **strategy pattern** and **template pattern** centered around the following three classes

* `class GeomCalc(object)`
* `class GeomAnalyzerBase(object)`
* `class _CustomCalc(OrderedDict)`

`_CustomCalc` is a dictionary (specifically `collections::OrderedDict`) that caches all the elementary calculations for the analyzers.
Typically its keys are function calls and the values are dictionaries whose key value pairs correspond to function call inputs (e.g., `gids`) and outputs.
Its member function `_CustomCalc.calc(pbc, fr)` carries out all cached calculations for each frame.

`GeomCalc` is a callable that contains a `_CustomCalc` instance for calculations.
It supports four basic geometric calculations which only require `gids` as inputs

* vector
* distance
* angle
* torsion

These calculations take care of the unwrapping of the periodic boundary condition.
There are also four analyzer classes to provide direct access to these calculations.


`GeomAnalyzerBase` is the abstract base class for all trajectory analyzer class.
It is a callable that returns the calculation result.

All other analyzers are registered to the `GeomCalc` instance using `GeomCalc.addAnalyzer(analyzer)`.
In `GeomAnalyzerBase._precalc(calc)`, one should use `GeomCalc.addCustom(cid, key, default)` to register the elementary calculation.
In `GeomAnalyzerBase._postcalc(calc, pbc, fr)`, one should retrieve the results of the elementary calculation and perform further calculations if needed.

## analyzer prototypes

There are four analyzer prototypes

* `class GeomAnalyzerBase(object)`
* `class _MaestroAnalysis(GeomAnalyzerBase)`
* `class _CompositeAnalyzer(GeomAnalyzerBase)`
* `class CustomMaestroAnalysis(CustomMaestroAnalysis)`
* `class _DynamicAslAnalyzer(GeomAnalyzerBase)`

Here `GeomAnalyzerBase` is an **abstract base class**. Here is an example that
implement analyzer .

```python
class FooAnalyzer(GeomAnalyzerBase):
    def __init__(msys_model, cms_model, lig_asl):
        """
        @type msys_model: C{msys.System}
        @type  cms_model: C{schrodinger.structure.Structure}
        @type    lig_asl: C{str}
        @param   lig_asl: ASL expression to select ligand atoms
        """
        lig_aids = cms_model.select_atom(lig_asl)
        lig_gids = topo.aids2gids(cms_model, lig_aids, include_pseudoatoms=False)

    def _precalc(self, calc):
        """
        @type calc: L{GeomCalc}
        """
        # to do some real work, use calc.addCustom() to register function calls
        pass


    def _postcalc(self, calc, pbc, fr):
        """
        @type calc: L{GeomCalc}
        @type  pbc: L{Pbc}
        @type   fr: L{traj.Frame}
        """
        # fr is the current frame 
        # one could use calc.getCustom() to retrieve intermediate calculation results
        pass
```


The intention of `_MaestroAnalysis` is to provide intermediate data (a frame and a full-system CT) with all the solute atoms centered.

```python
class FooMaestroAnalyzer(_MaestroAnalysis):
    def __init__(msys_model, cms_model, lig_asl):
        """
        @type msys_model: C{msys.System}
        @type  cms_model: C{schrodinger.structure.Structure}
        @type    lig_asl: C{str}
        @param   lig_asl: ASL expression to select ligand atoms
        """
        _MaestroAnalysis.__init__(self, msys_model, cms_model)
        lig_aids = cms_model.select_atom(lig_asl)
        lig_gids = topo.aids2gids(cms_model, lig_aids, include_pseudoatoms=False)

    def _precalc(self, calc):
        """
        @type calc: L{GeomCalc}
        """
        # to do some work beyond getting centered data,
        # use calc.addCustom() to register function calls
        pass

    def _postcalc(self, calc, pbc, fr):
        """
        @type calc: L{GeomCalc}
        @type  pbc: L{Pbc}
        @type   fr: L{traj.Frame}
        """
        # fr is the current frame 
        # one could use calc.getCustom() to retrieve intermediate calculation results
        centered_fr, centered_cms_model = self._getCentered(calc)
        pass
```

The intention of `_CompositeAnalyzer` is to simplify the implementation when one analyzer needs other analyzers as its helpers, i.e., the **composition pattern**.

## reduce mechanism

The design of the trajectory analysis classes aims to share auxiliary
calculation results between analyzers for each frame. 
However, no information is shared between frames.
The reduce mechanism uses the results from all frames as input for trajectory-wise
analysis, for example, performing statistics, etc.


## more advanced topics

### a tale of three IDs

There are three types of IDs in use

* AID: index used by the maestro which starts from 1
    * cms_model
    * ct
* GID: index used by the msys_model which starts from 0
    * msys_model
    * frame objects
* XID
    * frame objects

### positer mechanism

The positer mechanism enables uniform treatment of 

* atom
* centroid of a group of atoms
* center of mass of a group of atoms
* center of charge of a group of atoms

It has **side effect**: extra `gids` will be created in the frame object.

`class Positer(object)`

Each class instance works with two  `GeomCalc` objects

* an internal `GeomCalc` object that computes the results of the `CenterOf` analyzers
* an external `GeomCalc` object that keeps track of the index of the extra `gids`


``` python
positer = Positer(, )
ext_calc.addPosition(positer, num_pos)

```

### Using the `DynamicAslAnalyzer`

The 
