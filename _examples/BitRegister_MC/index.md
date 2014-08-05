---
layout: default
title: BitRegister_MC
---

## BitRegister_MC
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


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

