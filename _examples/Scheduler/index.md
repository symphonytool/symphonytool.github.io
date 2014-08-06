---
layout: default
title: Scheduler
---

## Scheduler
Author: Jim Woodcock


This system schedules a collection of processes to ensure that only one is in a critical region for using some resource. The system shows how to build a network of processes and run the execution engine to test the result. A refinement is to require that the processes are permitted to enter the critical region in the order in which they became ready to do so. The idiom used here is to strengthen the behaviour of the system by adding a process in parallel that constrains behaviour as required. The result is a formal refinement. This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. 


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

