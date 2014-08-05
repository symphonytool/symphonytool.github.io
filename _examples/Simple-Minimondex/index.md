---
layout: default
title: Simple-Minimondex
---

## Simple-Minimondex
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### simpler-minimondex.cml

{% raw %}
~~~
values
 N: nat = 2
 V: nat = 1
 M: nat = N*V

types
  Index = nat 
    inv i == i in set {1,...,N}

  Money = nat
    inv m == m in set {0,...,M}
    

channels
  pay, transfer: Index * Index * Money
  accept , reject: Index

process Card = val i: Index @
begin
  state value: nat := 0
  
  operations
    Init: () ==> ()
    Init() == value := V

    Credit: nat ==> ()
    Credit(n) == value := value + n

    Debit: nat ==> ()
    Debit(n) == value := value - n
    pre value >= n
  
  actions
    Transfer = pay.i?j?n ->
    ( [n > value] & reject!i -> Skip
      []
      [n <= value] & transfer.i.j!n -> accept!i -> Debit(n) )
      
    Receive = transfer?j.i?n -> Credit(n)
    
    Cycle = ( Transfer [] Receive ); Cycle
    
@ Init(); Cycle

end

process OneCard = Card(1) 

--THE FOLLOWING PROCESSES MAKE THE ANALYSIS VERY SLOW AND THE MODEL CHECKER 
--STILL HAS PROBLEMS TO HANDLE IT. 
process Cards =
  || i: Index @ 
  		[ {| pay.i,transfer.i,accept.i, reject.i |} union {| transfer.j.i.n | j:Index,n:Money|} ] Card(i)

--the original description uses {| pay.i,transfer.i,accept.i, reject.i |} union {transfer.j.i.n | j <- Index, n <- Money}
-- but the construct var <- expression is not accepted by the parser

process Network = Cards \\ {|transfer|}
~~~
{% endraw %}

