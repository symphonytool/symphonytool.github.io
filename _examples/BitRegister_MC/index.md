---
layout: default
title: BitRegister_MC
---

## BitRegister_MC
Author: Jim Woodcock


This simple bit register example has been provided in VDM by a student called Andreas Mueller from Austria. 
It has been translated to CML by Jim Woodcock and used for testing the Symphony tool.

This example defines the main operators you can conduct on a very simple calculator where you can have 
registers you can store and load values to and from, and then add and subtract. The original version handles 
bytes up to 255. This cut-down version was produced by Adalberto Cajueiro to handle bytes up to 4 so it is 
possible to see the effect of overflows and underflows in the model checker.

The deadlock found by the model checker produced the same trace in the animator (except for the tau-events).
A good exercise it to run these two plugins to observe the results. 

The original version uses an input as the value to be incremented/decremented. This version uses a value 
that can be changed in each execution/analysis.


### simpler-register.cml

{% raw %}
~~~
--The original value for MAX is 255. Due to performance questions we use 4
--The value of increment establishes on how the reg variable is updated (incremented or decremented).
-- This can caus under or overflows. 
values
    MAX : nat = 4
    increment : nat = 1
    
functions
   oflow : int*int -> bool
   oflow(i,j) == i+j > MAX

   uflow : int*int -> bool
   uflow(i,j) == i-j < 0

types
  MYINT = nat
    inv i == i in set {increment}
    
channels
   init, overflow, underflow
   inc, dec : MYINT

process RegisterProc = 
 begin
	
  state 
  reg : int	:= 0

  operations  
   INIT : () ==> ()
   INIT() == reg := MAX - 1

   ADD: int ==> ()
   ADD(i) == reg := reg + i

   SUB: int ==> ()
   SUB(i) == reg := reg - i
   
  actions
   
   REG = 
     (inc.increment -> [not oflow(reg,increment)] & ADD(increment);REG)
     []
     (dec.increment -> [not uflow(reg,increment)] & SUB(increment);REG)
   @
     init -> INIT(); REG
 end
~~~
{% endraw %}

