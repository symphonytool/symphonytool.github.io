---
layout: default
title: Scheduler
---

## Scheduler
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### Scheduler.cml

{% raw %}
~~~
values
  psize = 5
  PIDset = {1,...,psize}
  
types
  PID = nat
    inv p == p in set PIDset

channels
  create, delete, ready, enter, leave : PID

process Proc = p: PID @
begin
  actions
    NewProc = create.p -> RunProc
    RunProc =
      ready.p -> enter.p -> leave.p -> RunProc
      []
      delete.p -> NewProc
@
  NewProc
end

process Mutex =
begin
@
  mu X @ (enter?p -> leave!p -> X) 
end

process Scheduler = 
  ( ||| p in set PIDset @ Proc(p) ) [|{|enter,leave|}|] Mutex
  
process Queue =
begin
  state 
    q: seq of PID := []
      inv len(q) <= psize and card elems q = len q 
  actions
    Q = 
      [len(q) < psize] & ready?p -> q := q^[p]; Q
      []
      [not (q = [])] & enter!(hd(q)) -> q := tl(q); Q
@
  Q
end

process Scheduler1 =
  Scheduler [|{|ready,enter|}|] Queue
~~~
{% endraw %}

