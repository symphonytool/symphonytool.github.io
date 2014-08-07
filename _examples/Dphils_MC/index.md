---
layout: default
title: Dphils_MC
---

## Dphils_MC
Author: Adalberto Cajueiro


This simple dining philosophers models the classical example considering two philosofers.

The deadlock found by the model checker produced the same trace in the animator (except for the tau-events).
A good exercise it to run these two plugins to observe the results.  


### SimpleDphils.cml

{% raw %}
~~~
channels
get00,get10,get01,get11,puts00,puts01,puts10,puts11,eat0,eat1

process PHILS = 
begin
 actions
   PHIL0 = get00 -> get01 -> eat0 -> puts00 -> puts01 -> PHIL0
   PHIL1 = get11 -> get10 -> eat1 -> puts11 -> puts10 -> PHIL1
   
   @ PHIL0 ||| PHIL1 
end

process FORKS = 
begin
 actions
   FORK0 = (get00 -> puts00 -> FORK0) [] (get10 -> puts10 -> FORK0)
   FORK1 = (get01 -> puts01 -> FORK1) [] (get11 -> puts11 -> FORK1)
   @ FORK0 ||| FORK1 
end

process Dphils = PHILS [| {get00,get10,get01,get11,puts00,puts01,puts10,puts11} |] FORKS
~~~
{% endraw %}

