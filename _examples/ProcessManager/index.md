---
layout: default
title: ProcessManager
---

## ProcessManager
Author: 


This project needs a description.


| Properties | Values          |
| :------------ | :---------- |
|Language Version:| cml|


### ProcessManager.cml

{% raw %}
~~~
values

  N : nat1 = 10
  
types
  Status = <READY> | <BLOCKED>
  Process :: id: nat
             status : Status

channels
  admit, wakeup, dispatch : nat
  init, timeout, block, terminate

process ProcessManager =
begin
  state
    running : [nat] := nil
    waiting : seq of Process := []
    inv
      ( running <> nil => forall i in set inds waiting @ waiting(i).id <> running )
      and
      ( forall i,j in set inds waiting @ i <> j => waiting(i).id <> waiting(j).id )
      
  functions
    findPos(q: seq of Process,id: nat) pos : nat
      pre  exists p in set (elems q) @ p.id = id
      post pos in set elems [i | i in set inds q @ q(i).id = id]
      
    findNext: seq of Process -> nat1
    findNext(q) ==
      hd [i | i in set inds q @ q(i).status = <READY>]
      pre exists p in set elems q @ p.status = <READY>
      post 
        q(RESULT).status = <READY>
        and
        forall i in set {1,...,RESULT-1} @ q(i).status = <READY>
        
    remove: seq of Process * nat -> seq of Process
    remove(q, pos) ==
      q(1,...,pos-1) ^ q(pos+1,...,len q)
    pre  pos in set inds q
    post RESULT = q(1,...,pos-1) ^ q(pos+1,...,len q)
      
    NonNil: [nat] -> bool
    NonNil(n) ==
      n <> nil
      
    PreAdmit: nat * [nat] * seq of Process -> bool
    PreAdmit(i,r,w) ==
      (r <> nil => i <> r)
      and
      forall p in set elems w @ p.id <> i

    PreDispatch: [nat] * seq of Process -> bool
    PreDispatch(r,w) ==
      r = nil
      and
      exists p in set elems w @ p.status = <READY>
            
    PreWakeup: nat * seq of Process -> bool
    PreWakeup(i,w) ==
      w(findPos(w,i)).status = <BLOCKED>
      
  operations
    Init : () ==> ()
    Init() ==
      running := nil ; waiting := []

    Admit: nat ==> ()
    Admit(id) ==
      waiting := waiting ^ [mk_Process(id,<READY>)]
    pre   PreAdmit(id,running,waiting)
    post  waiting = waiting~ ^ [mk_Process(id,<READY>)]    

    Dispatch: nat ==> ()
    Dispatch(id) ==
      running := waiting(findNext(waiting)).id;
      waiting := remove(waiting,findNext(waiting))
    pre   PreDispatch(running,waiting)
    post  running = waiting~(findNext(waiting~)).id
          and
          waiting = remove(waiting~,findNext(waiting~))

    Timeout()
      frame wr running : [nat]
            wr waiting : seq of Process
      pre   NonNil(running)
      post  waiting = waiting~ ^ [mk_Process(running~,<READY>)]
            and
            running = nil

    Block()
      frame wr running : [nat]
            wr waiting : seq of Process
      pre   NonNil(running)
      post  waiting = waiting~ ^ [mk_Process(running~,<BLOCKED>)]
            and
            running = nil

    Wakeup: nat ==> ()
    Wakeup(id) ==
      waiting := waiting ++ {findPos(waiting,id) |-> mk_Process(id,<READY>)}
    pre   PreWakeup(id,waiting)
    post  waiting = waiting~ ++ {findPos(waiting~,id) |-> mk_Process(id,<READY>)}

    Terminate: () ==> ()
    Terminate() ==
      running := nil
    pre   NonNil(running)
    post  running = nil

  actions
    Cycle =
      ( 
        [] s in set {1,...,N} @ 
            [PreAdmit(s,running,waiting)] & admit.s -> Admit(s)
        []
        [PreDispatch(running,waiting)] & dispatch?s -> Dispatch(s)
        []
        [NonNil(running)] & timeout -> Timeout()
        []
        [NonNil(running)] & block -> Block()
        []
        wakeup?s -> [PreWakeup(s,waiting)] & Wakeup(s) -- what else?
        []
        [NonNil(running)] & terminate -> Terminate() ) ; Cycle
@
  init -> Init(); Cycle
end
~~~
{% endraw %}

