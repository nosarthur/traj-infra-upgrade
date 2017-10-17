# traj-infra
Examples of transition to new trajectory infrastructure.

 \ | old | new 
 --- | --- | --- 
access atoms | <pre>cms_st = cmsstructure.read_cms(model_fname) <br>for a in cms_st.atom:<br>    print(a.index) </pre> | <pre>_, cms_model = topo.read_cms(model_fname) <br>for a in cms_model.atom:<br>    print(a.index)</pre>
  |  |
  |  |


