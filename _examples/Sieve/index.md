---
layout: default
title: Sieve
---

## Sieve
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### Sieve.cml

{% raw %}
~~~
-- Optimised sieve
--
-- Input: an integer n > 1
--
-- Let A be an array of boolean values indexed by integers 2 to n,
-- initially all set to true.
--
-- for i = 2,3,4,..., not exceeding sqrt(n)
--   if A[i] is true then
--     for j = i^2, i^2+1, i^2+2i,..., not exceeding n
--       A[j] := false
--
-- Output: all i such that A[i] is true





channels 
  ch: nat * nat
  newch: nat
  atfilter: nat * nat *nat
  atsieve: nat * nat

process Primes =
begin
  actions
   Chans = i: nat @ newch!i -> Chans(i+1)
  
    Filter = p,i,o: nat @
    atfilter!p!i!o ->
      while true do
        ch.i?x ->
          if x mod p <> 0
          then
            ch.o!x -> Skip
            
    Sieve = p,c: nat @
    atsieve!p!c ->
      ch.c!p ->
        newch?d ->
          ( ( Sieve(p+1,d) [|{|ch.d|}|] Filter(p,d,c) ) \\ {|ch.d|} )
@
  Sieve(2,1) [|{|newch|}|] Chans(2)
end


~~~
{% endraw %}

