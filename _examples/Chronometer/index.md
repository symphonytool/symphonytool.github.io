---
layout: default
title: Chronometer
---

## Chronometer
Author: Alvaro Miyazawa


This simple chronometer example has been taken from the PhD work from Marcel
Oliveira in CSP that has been moved to CML by Alvaro Miyazawa. This example
is particular good to illustrate the capabilities of the CML refinement support
tool. The example contains both an olympian specification of a clock as well as
a a refined version that distributed it in multiple components.


### Chronometer.cml

{% raw %}
~~~
types
	RANGE = nat inv i == i < 60
	
channels
	tick, time
	out: RANGE*RANGE	
	
process AChronometer = begin
state sec: RANGE
	  min: RANGE
	  
actions

AInit = [frame wr sec, min pre true and true post sec = 0 and min = 0]

incSec = [frame wr sec pre true post sec = (sec~ + 1) mod 60]	  

incMin = [frame wr min pre true post min = (min~ + 1) mod 60]

Run = tick -> incSec(); ([sec = 0] & incMin() [] [sec <> 0] & Skip)
      []
      time -> out!min!sec -> Skip

@ AInit; mu X @ (Run; X)
end

channels inc, minsReq
channels ans: RANGE

process CChronometer = begin
state sec: RANGE
	  min: RANGE

actions

SecInit = [frame wr sec pre true post sec = 0]

MinInit = [frame wr min pre true post min = 0]

incSec = [frame wr sec pre true post sec = (sec~ + 1) mod 60]	  

incMin = [frame wr min pre true post min = (min~ + 1) mod 60]

RunSec = tick -> incSec(); ([sec = 0] & incMin() [] [sec <> 0] & Skip)
		 []
		 time -> minsReq -> ans?mins -> out!mins!sec -> Skip

RunMin = inc -> incMin() [] minsReq -> ans!min -> Skip

@ (
	(SecInit(); mu X @ (RunSec; X)) 
		[|{sec}|{|inc, minsReq, ans|}|{min}|]
	(MinInit(); mu X @ (RunMin; X))
)\\{|inc, minsReq, ans|}
end
~~~
{% endraw %}

