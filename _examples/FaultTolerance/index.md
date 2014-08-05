---
layout: default
title: FaultTolerance
---

## FaultTolerance
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### FTConcepts.cml

{% raw %}
~~~
types
     Unwanted = <Fault1> | <Error1> | <Failure1>
    
channels
       a,
       b
       unwanted: Unwanted

chansets
       H = {| |}
       Alpha_LFTSimple = {| a,b |}
       Alpha_NFTSimple = {| a,b |}
       Alpha_DLSimple = {| a,b |}
       Alpha_FFTSimple = {| a,b |}

process Limit =
begin
    actions
        Init = unwanted.<Fault1> -> Init
    @ Init
end

process Limit_LFTSimple = Limit
process Limit_NFTSimple = Limit
process Limit_DLTSimple = Limit
process Limit_FFTSimple = Limit

-- Deadlocked system
process DLSimple =
begin
        actions
              NOMINAL_DLSimple = a -> ((b -> Stop) [] FAULT_DLSimple)
              FAULT_DLSimple = unwanted.<Fault1> ->
                   ((b -> Stop) [] (unwanted.<Error1> -> Skip))
       @ NOMINAL_DLSimple
end

-- Full fault tolerant
process FFTSimple =
begin
        actions
              NOMINAL_FFTSimple = a -> ((b -> NOMINAL_FFTSimple) [] FAULT_FFTSimple)
              FAULT_FFTSimple = unwanted.<Fault1> -> b -> NOMINAL_FFTSimple
       @ NOMINAL_FFTSimple
end

-- Fault tolerant (limit).
process LFTSimple =
begin
        actions
              NOMINAL_LFTSimple = a -> ((b -> NOMINAL_LFTSimple) [] FAULT_LFTSimple)
              FAULT_LFTSimple = unwanted.<Fault1> ->
                   ((b -> NOMINAL_LFTSimple) [] (unwanted.<Error1> -> Skip))
       @ NOMINAL_LFTSimple
end

-- Not fault tolerant
process NFTSimple =
begin
        actions
              NOMINAL_NFTSimple = a -> ((b -> NOMINAL_NFTSimple) [] FAULT_NFTSimple)
              FAULT_NFTSimple = unwanted.<Fault1> ->
                   (NOMINAL_NFTSimple [] (unwanted.<Error1> -> Skip))
       @ NOMINAL_NFTSimple
end
~~~
{% endraw %}

