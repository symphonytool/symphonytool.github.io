---
layout: default
title: IncubatorMonitor
---

## IncubatorMonitor
Author: 


This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model an incubator monitor at a very abstract level. The CML model of this has been produced by Jim Woodcock. The basic incubation monitor example is simply a device that continually monitors a temperature that either can be increased with an Increment operation or decreased with a Decrement operation. This example illustrate the use of guards since there is a minimum and a maximum temperature controlled by the PreInc and PreDec functions. Whenever the inc and dec channels are used the temperature represented with the state component temp is incremented and decremented by one. Pre and post-conditions are also incorporated in this example.

The IncubatorController process can be debugged but no input is expected from the user at all (except for choosing to go up or down in temperature). Note that you also can set breakpoints and inspect the temp variable when the simulation is stopped at such breakpoints. Note also that it is easy to reach a deadlock by giving a value outside the state invariant. Try it and find out how to repair this.


### IncubatorMonitor.cml

{% raw %}
~~~
values
	MAX  : int = 10
	MIN  : int = -10
	INIT : int = 5

channels
    init, inc, dec

process IncubatorMonitor = 
begin

	state
		temp : int
	inv
		MIN <= temp and temp <= MAX
		
    functions
    
    PreInc: int -> bool
    PreInc(t) ==
      t < MAX
      
    PreDec: int -> bool
    PreDec(t) ==
      t > MIN
      
	operations
		Init: () ==> ()
		Init() ==
			temp := INIT
			post temp = INIT
	
		Increment: () ==> ()
		Increment() ==
			temp := temp + 1
			pre  PreInc(temp)
			post temp = temp~ + 1
			
		Decrement: () ==> ()
		Decrement() ==
			temp := temp - 1
			pre  PreDec(temp)
			post temp = temp~ - 1
			
		getTemp: () ==> int
		getTemp() == 
			return temp
			
	actions
		Cycle = (([PreInc(temp)] & inc -> Increment()) 
		         [] 
		         ([PreDec(temp)] & dec -> Decrement())) ; Cycle
			
@

	init -> Init() ; Cycle

end
~~~
{% endraw %}

