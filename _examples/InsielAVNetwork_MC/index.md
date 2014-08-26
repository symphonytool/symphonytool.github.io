---
layout: default
title: InsielAVNetwork_MC
---

## InsielAVNetwork_MC
Author: Zoe Andrews and Jeremy Bryans


This example was provided by Zoe Andrews and Jeremy Bryans and modified by Adalberto Cajueiro. 
It models the AV network, keeping renderers in synchronisation.  


### InsielAVNetwork_MC.cml

{% raw %}
~~~
-- Keeping renderers in sync in an AV network
-- Inspired by "Clunker" model by Bill Roscoe.

/**
 * restricted for model-checker
 * With version 0.3.5-SNAPSHOT of Symphony.
 */

-- Zoe Andrews and Jeremy Bryans
types
 --no typea are declared
 
channels
 a : nat
 c : nat --* nat -- counter channel

chansets
 AllButC = {|a|}
  
-- inspired by Clunker
-- Counter(id) counts the number of a.id events that have taken place.
process Counter = id : nat @ 
begin
 state
 counter : nat := 0
 
 operations
 incCount:()==>()
 incCount() ==
   (counter := counter + 1)
   
 actions
-- Counting = (c.id.(count) -> a.id -> incCount();Counting) 
 Counting = (a!id -> incCount(); c!counter -> Counting) 
 @ Counting
 end 

-- An auxiliary process to test process Counter
process TestCounter = Counter(1)

-- P is a basic playing process, that plays the frames a.id with no timing constraints
process P = id : nat @ 
begin
  actions
  Play = a.id -> Play
       
  @ Play
 end 

-- An auxiliary process to test P
process TestP = P(1)

-- Renderer represents the final renderers in the AV system, and is a combination of basic playing and counting
process Renderer = id : nat @ P(id) [|{|a|}|] Counter(id) 

-- We have two Renderers in our system
process Renderers = Renderer(1)  ||| Renderer(2) 
 

channels
 maxx : nat 
 minn : nat 


-- process Limiter adapted to cope with only two processes: for model checking
process Limiter = lim:nat @
begin
 state
 count1 : nat := 0
 count2 : nat := 0

 actions
 Limit = [count1-count2=2] & (count2:=count2+1);c!count2 -> Limit    -- changed order of update and channel c communication
         []
         [count2-count1=2] & (count1:=count1+1);c!count1 -> Limit    -- changed order of update and channel c communication
         []
         [-2 < (count2-count1) and (count2-count1) < 2] & 
           (
             count1:=count1+1;c!count1 -> Limit 
             --c!(count1+1) -> (count1:=count1+1);Limit
             --the order was inverted because expressions like (count1+1) are not allowed in outputs (for model checkeing purposes)    
             []
             --c.2!(count2+1) -> (count2:=count2+1);Limit 
             count2:=count2+1; c!count2 -> Limit
           )
  @ Limit
 end 

--Simple process to test th eprocess Limiter
process OneLimiter = Limiter(2) 


-- Limited is the two limited renderers.
process Limited = Renderers [|{|c|}|] Limiter(2)


process SpecificationForTwoProcs =  
 begin
 state 
  -- state variables have to be different from state variables of any other process
  newcount1 : nat := 0
  newcount2 : nat := 0
    
  actions 
  
  Spec = [newcount1-newcount2=2] & (newcount2:=newcount2+1);c!newcount2 -> Spec
         []
         [newcount2-newcount1=2] & (newcount1:=newcount1+1);c!newcount1 -> Spec
         
    @ Spec
  end 
  
-- hypothesis : process DF is deadlock-free
process DF =  SpecificationForTwoProcs  [|{|c|}|] Limited 


 
~~~
{% endraw %}

