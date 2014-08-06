---
layout: default
title: DataFlow
---

## DataFlow
Author: Jim Woodcock


The data flow machine transforms its sequence of values in its input channel into a sequence of computations in its output channel. It shows how the use of parallel processes can speed up the computation. An important idiom is introduced: the use of ghost variables and a ghost process to demonstrate the correctness of the computation. A second important idiom is the use of data independence in model checking: the behaviour of the data flow machine does not depend on decisions taken on the basis of its inputs, it merely transforms the data. This means that checks for deadlock and livelock freedom of the network of processes in the machine can be carried out with synchronisations rather than communications. In this case, the infinite state data flow machine can be easily model checked with only 24 states.


| Properties | Values          |
| :------------ | :---------- |
|Language Version:| cml|


### DataFlow.cml

{% raw %}
~~~
channels

  left, left1, left2, mid, right : nat
  diverge : nat
  
values 
  
  a : nat = 2
  
  b : nat = 5
  
process DataFlow = begin

state

  inseq : seq of nat := []
  
  outseq : seq of nat := []
  
functions

StateInv: seq of nat * seq of nat -> bool
StateInv(inlist,outlist) ==
  len inlist >= len outlist and
  outlist = [a*inlist(i) + b * inlist(i+1) | i in set inds outlist] 
  --Func(inlist,outlist)
  
Func: seq of nat * seq of nat -> seq of nat
Func(inlist,outlist) ==
  [a*inlist(i) + b * inlist(i+1) | i in set inds outlist]

actions

  X21 = left1?x -> mid!(a * x) -> X21
  
  X22 = left2?y -> mid?z -> (dcl r : nat := z + b*y @ right!r -> X22)
  
  X23 = left?x -> left1!x -> X24
  
  X24 = left?x -> left2!x -> left1!x -> X24
  
  GHOST = (left?x -> inseq:= inseq ^[x]; GHOST
       [] right?r -> outseq := outseq ^[r]; GHOST
       [] INVBROKEN)
       
  INVBROKEN = [not StateInv(inseq,outseq)] & Diverge 
@ 

  (X23 [| {| left1,left2 |} |]
    ( X21 [| {| mid |} |] X22 ) ) [| {| left, right |} |] GHOST

end
~~~
{% endraw %}

