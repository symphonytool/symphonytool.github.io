---
layout: default
title: Frogs
---

## Frogs
Author: Jim Woodcock


This example is the frog jumping puzzle that was used in the 
introductionary CSP training of CSP at the first COMPASS meeting.
This example illustrates a lot of invariants but no concurrency.
The CML model is written by Jim Woodcock.



### Frogs.cml

{% raw %}
~~~
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
values

  POset = {1,...,7}
  
types
  POS = nat
    inv p == p in set POset
  ML_1 = nat
    inv ml_1 == ml_1 in set {2,...,7}
  ML_2 = nat
    inv ml_2 == ml_2 in set {3,...,7}
  MR_1 = nat
    inv mr_1 == mr_1 in set {1,...,6}
  MR_2 = nat
    inv mr_2 == mr_2 in set {1,...,5}

channels
  move_left_1 : ML_1
  move_left_2 : ML_2
  move_right_1: MR_1
  move_right_2: MR_2
  done

process Frogs = 
begin
  state
    F: set of POS := {1,2,3}
    E: POS := 4
    M: set of POS := {5,6,7}
      inv
        not (E in set F)
        and not (E in set M)
        and F inter M = {}
        and F union {E} union M = {1,...,7} 
  functions
    final_state: set of POS * POS * set of POS -> bool
    final_state(F,E,M) == (F={5,6,7}) and (E=4) and (M={1,2,3})
  actions
    FrogMarch =
      ( ( [] i in set POset @ [i in set F and E = i+1] & move_right_1!i -> atomic(F := (F \ {i}) union {i+1}; E := i) )
        []
        ( [] i in set POset @ [i in set F and E = i+2] & move_right_2!i -> atomic(F := (F \ {i}) union {i+2}; E := i) )
        []
        ( [] i in set POset @ [i in set M and E = i-1] & move_left_1!i  -> atomic(M := (M \ {i}) union {i-1}; E := i) )
        []
        ( [] i in set POset @ [i in set M and E = i-2] & move_left_2!i  -> atomic(M := (M \ {i}) union {i-2}; E := i) )
        []
        ([final_state(F,E,M)] & done -> Stop) );
      FrogMarch
@
  FrogMarch
end
         
~~~
{% endraw %}

