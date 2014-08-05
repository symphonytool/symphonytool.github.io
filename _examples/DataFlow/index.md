---
layout: default
title: DataFlow
---

## DataFlow
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


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

