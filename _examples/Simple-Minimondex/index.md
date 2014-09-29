---
layout: default
title: Simple-Minimondex
---

## Simple-Minimondex
Author: Jim Woodcock


This CML model has been developed by Jim Woodcock inspired by the original work done with the Mondex secure card system (see [Stepney&00,Woodcock&08a]. In 1994, National Westminster Bank developed an electronic
purse (or smart card) system, called Mondex. An
electronic purse is a card-sized device intended to replace "real"
coins with electronic cash. In contrast to a credit or debit card,
an electronic purse stores its balance in itself, thus does not 
necessarily require any network access to update a remote database
during a transaction. So, electronic purses can be used in small
stores or shops, such as bakeries, where small amounts of money
are involved. In the CML model here however, the different potential faults that could happen in these transactions are not taken into account.

When debugging the MiniMondex example one will get an option to select between ten different cards that wish to transfer an amount of money to another card. Here you can experiment with transferring money between cards and see how it can communicate either over the accept or the reject channel. Experiment with sending money from a card to itself and with transferring more money than the amount left on a particular Card.


### simpler_minimondex.cml

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

