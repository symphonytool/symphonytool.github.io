---
layout: default
title: IncubatorController
---

## IncubatorController
Author: Quentin Charatan and Aaron Kans


This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model an incubator controller at a very abstract level. The CML model of this has been produced by Jim Woodcock. The basic incubation controller example is simply a device that continually monitors and controls a temperature that either can be increased with an Increment operation or decreased with a Decrement operation. This example illustrate the use of guards since there is a minimum and a maximum temperature. Whenever the inc and dec channels are used the temperature represented with the state component temp is incremented and decremented by one. Pre and post-conditions are also incorporated in this example.

The IncubatorController process can be debugged but no input is expected from the user at all (except for choosing to go up or down in temperature). Note that you also can set breakpoints and inspect the temp variable when the simulation is stopped at such breakpoints. Note also that it is easy to reach a deadlock by giving a value outside the state invariant. Try it and find out how to repair this.


| Properties | Values          |
| :------------ | :---------- |
|Language Version:| cml|


### IncubatorController.cml

{% raw %}
~~~
values
	MAX  : int = 10
	MIN  : int = -10
	INIT : int = 5

types
  Signal = <INCREASE> | <DECREASE> | <IDLE>

channels
  init
  request, getActual, getrequest : int
  increment, decrement
  action : Signal

process IncubatorController =
begin
  functions
  inRange: int -> bool
  inRange(v) ==
    MIN <= v and v <= MAX
      
  PreSetInitTemp: int * bool -> bool
  PreSetInitTemp(v,a) ==
    inRange(v) and not a
      
  PreIncrement: int * int * bool * bool -> bool
  PreIncrement(actualT,requestT,r,a) ==  
    actualT < requestT and r and a
    
  PreDecrement: int * int * bool * bool -> bool
  PreDecrement(actualT,requestT,r,a) ==  
    actualT > requestT and r and a
    
  PreRequestChange: int * bool -> bool
  PreRequestChange(v,a) ==
    inRange(v) and not a
    
  state
    requestTemp  : int := INIT
    actualTemp   : int := INIT
    requestS     : bool := false
    actual       : bool := false
  inv
    (requestS => inRange(requestTemp)) and (actual => inRange(actualTemp))
    
  operations
    SetInitTemp : int ==> ()
    SetInitTemp(v) ==
      -- frame wr actualTemp : int
      actualTemp := v
    pre PreSetInitTemp(v,actual) 
      
    RequestChange : int ==> Signal
    RequestChange(v) ==
      requestTemp := v;
      return
        if v > actualTemp then
          <INCREASE>
        elseif v < actualTemp then
            <DECREASE>
        else
          <IDLE>
      pre PreRequestChange(v,actual) 
      
    Increment : () ==> Signal
    Increment() ==
      actualTemp := actualTemp + 1;
      return <INCREASE>
    pre PreIncrement(actualTemp,requestTemp,requestS,actual)
      
    Decrement : () ==> Signal
    Decrement() ==
      actualTemp := actualTemp - 1;
      return <DECREASE>
    pre PreDecrement(actualTemp, requestTemp, requestS, actual)
          
  actions
    Cycle =
      ( request?v -> [PreRequestChange(v,actual)] &
                     (dcl c : Signal @ c := RequestChange(v); action!c -> Skip)
        []
        [PreIncrement(actualTemp,requestTemp,requestS,actual)] & increment -> Increment()
        []
        [PreDecrement(actualTemp, requestTemp, requestS, actual)] & decrement -> Decrement()
        []
        [requestS] & getrequest!requestTemp -> Skip 
        []
        [actual] & getActual!actualTemp -> Skip ) ; Cycle 
  @
    init -> SetInitTemp(INIT); Cycle
end
~~~
{% endraw %}

