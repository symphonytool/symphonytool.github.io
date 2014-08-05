---
layout: default
title: InfComm_MC
---

## InfComm_MC
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### inf.cml

{% raw %}
~~~
// This example show a process that chooses values from a channel that accepts natural 
//numbers and uses them. There are infinite values to be communicated. However there are specific
//values that cause a deadlock and the model checker is able to find them. To achieve this, it is 
//necessary that at least two instances are internally created (chosen) by the model checker.
//This must be set in the Model Checker preferences.
//furthermore, this example shows a system that cannot be analysed by tools like FDR and PAT due to 
//manipulation of infinite events.  
types
    NATA = nat 
    NATB = nat
      
channels 
a:NATA
b:NATB

process P = 
begin 
  state
   n : NATA := 0
  operations
   Double:nat ==> ()
   Double(k) == n := 2*k
  actions
   MAIN = a?x -> Double(x); a?y -> if (n = y and y > 2) then Stop else Skip 
 @ MAIN  
end

-- one instantiation for each type communicated in a and b are enough to find the deadlock
process PRec = 
begin 
  state
   value : NATA := 0
  operations
   Triple:nat ==> ()
   Triple(v) == value := 3*v
  actions
   MAIN = a?x1 -> Triple(x1); b?y1 -> if (value = y1 and y1 > 2) then Stop else MAIN 
 @ MAIN  
end
    
~~~
{% endraw %}

