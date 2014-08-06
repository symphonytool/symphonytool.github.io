---
layout: default
title: IncubatorMonitor
---

## IncubatorMonitor
Author: 


This project needs a description.


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

