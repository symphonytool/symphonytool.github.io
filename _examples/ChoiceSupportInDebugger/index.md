---
layout: default
title: ChoiceSupportInDebugger
---

## ChoiceSupportInDebugger
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### Choice.cml

{% raw %}
~~~
channels 
a : int

process A = 
begin

 functions
 rndElement(s : set of int) r : int
 pre card s <> 0
 post r in set s
 
 operations
 
 takeRndElement: () ==> int
 takeRndElement() == (dcl x: int @ x := rndElement(v);v := v \ {x}; return x)
 	
 state
	v : set of int := {1,2,3,4}
	
 actions

 Main = [card v = 0] & Skip
 		[]
 		[card v > 0] & (dcl x : int @ x := takeRndElement(); a.(x) -> Main)		
	
 @ Main
end

~~~
{% endraw %}

