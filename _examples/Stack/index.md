---
layout: default
title: Stack
---

## Stack
Author: Peter Gorm Larsen


This is the standard stack example written in CML. If you wish to play with this you can add an invariant to put a limit to the stack size.



| Properties | Values          |
| :------------ | :---------- |
|Language Version:| cml|


### Stack.cml

{% raw %}
~~~
types
  Element = token

channels
  init, empty, nonempty
  push, pop : Element
  
process Stack =
begin
  state
    stack : seq of Element
    
  operations
    Init : () ==> ()
    Init() ==
      stack := []
      
    Push : Element ==> ()
    Push(e) ==
      stack := [e]^stack
      
    Pop : () ==> Element
    Pop() ==
      let tmp = hd(stack)
      in
        (stack := tl(stack);
         return tmp)
    pre stack <> []
    
  functions
  
    isEmpty : seq of Element -> bool
    isEmpty(s) ==
      s = []
      
  actions
    Cycle =
      ( push?e -> Push(e)
        []
        [not isEmpty(stack)] & (dcl v : Element @ v := Pop(); pop!v -> Skip)
        []
        [isEmpty(stack)] & empty -> Skip
        []
        [not isEmpty(stack)] & nonempty -> Skip ) ; Cycle
  @
    init -> Init(); Cycle
end
~~~
{% endraw %}

