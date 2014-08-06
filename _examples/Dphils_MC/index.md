---
layout: default
title: Dphils_MC
---

## Dphils_MC
Author: 


This project needs a description.


### SimpleDphils.cml

{% raw %}
~~~
channels
get00,get10,get01,get11,puts00,puts01,puts10,puts11,eat0,eat1
 
process Dphils = 
begin
 actions
   PHIL0 = get00 -> get01 -> eat0 -> puts00 -> puts01 -> PHIL0
   PHIL1 = get11 -> get10 -> eat1 -> puts11 -> puts10 -> PHIL1
   
   FORK0 = (get00 -> puts00 -> FORK0) [] (get10 -> puts10 -> FORK0)
   FORK1 = (get01 -> puts01 -> FORK1) [] (get11 -> puts11 -> FORK1)
   
   PHILS = PHIL0 ||| PHIL1
   FORKS = FORK0 ||| FORK1
   
   @ PHILS [| {get00,get10,get01,get11,puts00,puts01,puts10,puts11} |] FORKS 


end
~~~
{% endraw %}

