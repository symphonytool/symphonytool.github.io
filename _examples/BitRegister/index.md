---
layout: default
title: BitRegister
---

## BitRegister
Author: Jim Woodcock


This simple bit register example has been provided in VDM by a student called Andreas Mueller from Austria. It has been translated to CML by Jim Woodcock and used for testing the Symphony tool.

This example defines the main operators you can conduct on a very simple calculator where you can have registers you can store and load values to and from, and then add and subtract. Note that it only handles bytes up to 255 so it is possible to see the effect of overflows and underflows.

Just like for the Stack example the RegisterProc process can be debugged and it is possible to load values into the register and try to add and subtract them from each other. Here the natural behaviour of the bit register (including underflow and overflow scenarios) can be exercised. Note that you also can set breakpoints and inspect the different variables when the simulation is stopped at such breakpoints.

In the current version there are a number of proof obligations generated that you will not be able to discharge. Please update the argument types for some of the operations so this can be improved.



### BitRegisterProc.cml

{% raw %}
~~~
  types
    public Byte = int
    inv n == (n >= 0) and (n <= 255)

  functions
   oflow : Byte * Byte -> bool
   oflow(i,j) == i+j > 255

   uflow : Byte * Byte -> bool
   uflow(i,j) == i-j < 0


  channels 
   init, overflow, underflow
   read, load, add, sub : Byte

  chansets
  I = {| init, overflow, underflow, read, load, add, sub |}

 
process RegisterProc = 
 begin

  state 
  reg : Byte	

  operations  
   INIT : () ==> ()
   INIT() == reg := 0

   LOAD : int ==> ()
   LOAD(i) == reg := i
   
   READ: () ==>  int
   READ() == return reg

   ADD: int ==> ()
   ADD(i) == reg := reg + i
   pre not oflow(reg, i)
   post reg = reg~ + i

   SUB: int ==> ()
   SUB(i) == reg := reg - i
   //frame wr reg : int
   pre not uflow(reg,i)
   post reg = reg~ - i
  
  actions
   
   REG = 
     (load?i -> LOAD(i) ; REG)
     []
     (dcl j: int @ j := READ(); read!j -> REG)
     [] 
     (add?i -> ( ([oflow(reg,i)] & overflow -> INIT(); REG)
       	     	 [] 
                 ([not oflow(reg,i)] & ADD(i);REG)))
     [] 
     (sub?i -> ( ([uflow(reg,i)] & underflow -> INIT(); REG)
       	     	 [] 
                 ([not uflow(reg,i)] & SUB(i); REG)))
   @
     init -> INIT(); REG
 end
~~~
{% endraw %}

