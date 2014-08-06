---
layout: default
title: MiniMondex
---

## MiniMondex
Author: 


This project needs a description.


### minimondex.cml

{% raw %}
~~~
values
 N: nat = 10
 V: nat = 10
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
  state value: nat
  
  operations
    Init: () ==> ()
    Init() == value := V

    Credit: nat ==> ()
    Credit(n) == value := value + n

    Debit: nat ==> ()
    Debit(n) == value := value - n
  
  actions
    Transfer = pay.i?j?n ->
    ( [n > value] & reject!i -> Skip
      []
      [n <= value] & transfer.i.j!n -> accept!i -> Debit(n) )
      
    Receive = transfer?j.i?n -> Credit(n)
    
    Cycle = ( Transfer [] Receive ); Cycle
    
@ Init(); Cycle

end

process Cards =
  || i in set {1,...,N} @ 
  		[ {| pay.i,transfer.i,accept.i, reject.i |} union 
  		  {| transfer.j.i.n | j in set {1,...,N}, n in set {0,...,M}|} ] Card(i)

--the original description uses {| pay.i,transfer.i,accept.i, reject.i |} union {transfer.j.i.n | j <- Index, n <- Money}
-- but the construct var <- expression is not accepted by the parser

process Network = Cards \\ {|transfer|}
~~~
{% endraw %}

