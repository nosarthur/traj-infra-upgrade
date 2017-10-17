# traj-infra
Examples of transition to new trajectory infrastructure.

| old | new
--- | --- | ---
access atoms | ```cms_st = cmsstructure.read_cms(model_fname) ``` | ```_, cms_model = topo.read_cms(model_fname)```
