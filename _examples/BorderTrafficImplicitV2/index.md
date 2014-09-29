---
layout: default
title: BorderTrafficImplicitV2
---

## BorderTrafficImplicitV2
Author: 


This example models the traffic flow of a border crossing between two countries in implicit style.


### BorderTrafficImplv2_TP.cml

{% raw %}
~~~
types

  CountryId = token
  ActuatorId = token
  
  Location = int
       
  Actuator :: aId : ActuatorId
              loc : Location
              disp : nat

  --natural number corresponding to sequence position
  Corridor = seq of nat

functions

   -- Function to calculate the distance required for the corridor based upon the target
   -- speed and the current national speed limit of the country.
   calcDistance : nat * nat-> nat
   calcDistance(targetSpeed, speedLimit) ==
   (
      if targetSpeed <= speedLimit
      then floor ((speedLimit-targetSpeed)/maxInterval) + 1 -- not sure if this is best way, ok for now
      else 0
   )
   pre maxInterval > 0
   
   -- Function to calculate the speed target the neighbouring TMS must acheive. THis is based 
   -- upon the starting location of the corridor, the required distance of the corridor, and the
   -- ultimate target speed
   calcNeighbourTarget : int * nat * nat * nat -> nat   
   calcNeighbourTarget(startLoc, target, distance, speedLimit)  ==
   (
     let interval : nat = calcInterval(target, distance, speedLimit) in
     (
        target + (interval * abs(startLoc))
     )
   )
   pre speedLimit > target and distance > 0
   post RESULT > target

  -- Function to determine the interval between actuators
  calcInterval: nat * nat * nat -> nat
  calcInterval(target, distance, speedLimit) == 
      abs(floor((speedLimit-target)/distance))
   pre speedLimit > target and distance > 0   


channels

  --environment channels--
  inIncident : int * nat
  incidentClear: int
  
  --interface channels--
  neighbourRequest : CountryId * CountryId * nat
  neighbourOk : CountryId * CountryId
  
  --act mngr channels--
  actStatus : seq of nat 
  determineCorr : int * nat
  newCorr : Corridor
  createCorr : Corridor * nat
  disableCorr : Corridor

chansets
  -- the main interface between countryTMSs
  interface = {neighbourRequest, neighbourOk}
  external = {inIncident,incidentClear}


process CountryTMS =  i:CountryId, n:CountryId, sl:nat,  
                     aCorr : seq of Actuator @
begin

 state
  -- identifiers for the TMS and its neighbour
  myId : CountryId := i
  nId : CountryId :=n
  nationalSpeedLimit : nat :=sl
  -- an ordering of the actuators in the country - ordered by location
  acts : seq of Actuator := aCorr

  -- all invariants for TMS process
  inv myId <> nId  -- ensures the neighbour id is not its own.
  and forall a1, a2 in set inds acts @ -- establishes a maximum difference between contiguous actuators
         (a1 + 1 = a2 => acts(a1).disp - acts(a2).disp < maxDiff) 
  and forall a in set elems acts @ a.disp <= nationalSpeedLimit -- no actuator is displaying a speed too high
 
 operations
 
   -- Operation to determine which actuators should be present in the speed corridor
   -- Takes the stating location (i.e. where the target speed must be reached) and the 
   -- required distance of the corridor and returns a 'corridor' of actuators which 
   -- will have their displays changed.
   determineSpeedCorridor (startLoc: int, distance:nat) resc: Corridor
   pre len acts > 1 -- the TMS must have actuators
   post elems resc subset inds acts and 
        forall a in set elems resc @ 
          (acts(a).loc >= startLoc and acts(a).loc <= distance+startLoc)
   
   
   
   -- Given the collection of actuators provided, enact the speed corridor up to a 
   -- target speed as given in the operation parameters.
   createSpeedCorridor (pCorr: Corridor , target: nat) 
   pre elems pCorr subset inds acts and
       (len pCorr > 0 => target < nationalSpeedLimit)
   post forall a1, a2 in set elems pCorr @ (a1 <> a2 => 
         (acts(a1).loc > acts(a2).loc => acts(a1).disp >= acts(a2).disp))
   
   
   
   -- Given the speed corridor provided, remove the speed limits and re-enact the country's 
   -- national speed limit on all actuator displays.
   disableSpeedCorridor (pCorr: Corridor)
   pre elems pCorr subset inds acts
   post forall a in set elems pCorr @ acts(a).disp = nationalSpeedLimit
         --a should not be in any other corridors!
    
    
  
   
   --Determines if Neighbour is needed based upon the starting location and distance of the
   -- speed corridor
   --**IMPORTANT** current model of location means this is only valid for Country B!
   isNeighbourNeeded (startLoc : int, distance : nat) b: bool
   post b = not (exists act in set elems acts @ act.loc >= (startLoc + distance)) -- Only valid for Country B!
   
   
   --Operation to state current speeds shown on actuators
   outputActs() displ: seq of nat
   post displ = [acts(a).disp | a in set inds acts] 
   
@Skip
end 


values
 
  maxDiff : nat = 20
  maxChange : nat = 40
  
  theAId : CountryId = mk_token("Country A")
  theBId :CountryId = mk_token("Country B")
  
  limitA : nat = 100
  
  aIdA1 : ActuatorId = mk_token("aIdA1")
  aIdA2 : ActuatorId = mk_token("aIdA2")
  aIdA3 : ActuatorId = mk_token("aIdA3")
  aIdA4 : ActuatorId = mk_token("aIdA4")
   
  actA1 : Actuator = mk_Actuator(aIdA1, 4, limitA)
  actA2 : Actuator = mk_Actuator(aIdA2, 3, limitA)
  actA3 : Actuator = mk_Actuator(aIdA3, 2, limitA)
  actA4 : Actuator = mk_Actuator(aIdA4, 1, limitA)
  
  actCorrA : seq of Actuator = [actA1, actA2, actA3, actA4]
  
  restrictionSpeed : nat = 20
  blockageSpeed : nat = 5
  
  maxInterval : nat = 40

~~~
{% endraw %}

