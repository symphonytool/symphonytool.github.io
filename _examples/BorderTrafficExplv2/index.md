---
layout: default
title: BorderTrafficExplv2
---

## BorderTrafficExplv2
Author: 


This example models the traffic flow of a border crossing between two countries in explicit style.

The Border Traffic model aims to describe a contract to which a country's Traffic Management System (TMS) must conform. The model takes a simplified view of a 1-way road which connects two countries A and B, with a border cutting the road in two. 


### BorderTrafficv2.cml

{% raw %}
~~~
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 

types

  CountryId = token
  ActuatorId = token
  
  Location = int
           
  Actuator :: aId : ActuatorId
              loc : Location
              disp : nat
              
  ActuatorList = seq of Actuator

  --natural number corresponding to sequence position
  Corridor = seq of nat
  inv Corr == len Corr = card elems Corr
  
functions 
   
   -- Function to calculate the distance required for the corridor based upon the 
   -- target speed and the current national speed limit of the country.
   calcDistance : nat * nat-> nat
   calcDistance(targetSpeed, speedLimit) ==
   (
      -- not sure if this is best way, ok for now
      if targetSpeed <= speedLimit
      then floor ((speedLimit-targetSpeed)/maxInterval) + 1 
      else 0
   )
   pre maxInterval > 0
   
   -- Function to calculate the speed target the neighbouring TMS must acheive. 
   -- This is based upon the starting location of the corridor, the required 
   -- distance of the corridor, and the ultimate target speed
   calcNeighbourTarget : int * nat * nat * nat -> nat   
   calcNeighbourTarget(startLoc, target, distance, speedLimit)  ==
   (
     let interval : nat = calcInterval(target, distance, speedLimit) in
     (
        target + (interval * abs(startLoc))
     )
   )
   pre speedLimit > target and distance > 0

  -- Function to determine the interval between actuators
  calcInterval: nat * nat * nat -> nat
  calcInterval(target, distance, speedLimit) == 
      abs(floor((speedLimit-target)/distance))
   pre speedLimit > target and distance > 0    
   
   
   --Function to get the first actuator from the speed corridor
   getFirstAct: Corridor -> nat
   getFirstAct(Corr) == 
     hd Corr
   pre Corr <> []
   
   
   --Function to get the set of all actuators across the boundary
   --need to flatten the two sequences - must be distinct
   actsToSet : ActuatorList * ActuatorList -> set of Actuator
   actsToSet(a1, a2) == 
      getSingleActuatorSet({elems a1, elems a2})
   pre elems a1 inter elems a2 = {}
   
   getSingleActuatorSet : set of set of Actuator -> set of Actuator
   getSingleActuatorSet (allacts) == dinter allacts
   pre allacts <> {}

channels

  --environment channels--
  inIncident : CountryId * int * nat
  incidentClear: CountryId * int

  --interface channels--
  neighbourRequest : CountryId * CountryId * nat
  neighbourOk : CountryId * CountryId
  
  --act mngr channel--
  actStatus : CountryId * seq of nat 
  
  --internal corridor channels--
  determineCorr : int * nat
  newCorr : Corridor
  createCorr : Corridor * nat
  disableCorr : Corridor
 

chansets
  -- the main interface between countryTMSs
  interface = {neighbourRequest, neighbourOk}
  
  -- interface of SoS and incident creator
  incident = {inIncident,incidentClear}

  --internal channels--
  internal = {determineCorr, newCorr, createCorr, disableCorr}


process CountryTMS = i:CountryId, n:CountryId, sl:nat,  
                     aCorr : ActuatorList @
begin

 state
  -- identifiers for the TMS and its neighbour
  myId : CountryId := i
  nId : CountryId := n
  nationalSpeedLimit : nat := sl
  -- an ordering of the actuators in the country - ordered by location
  acts : ActuatorList := aCorr

  -- all invariants for TMS process:
  -- i) ensures the neighbour id is not its own.
  -- ii) establishes a maximum difference between contiguous actuators
  -- iii) no actuator is displaying a speed too high
  inv myId <> nId and 
  forall a1, a2 in set inds acts @
         (a1 + 1 = a2 => acts(a1).disp - acts(a2).disp < maxDiff) and
  forall a in set elems acts @ a.disp <= nationalSpeedLimit

 operations
 
   -- Operation to determine which actuators should be present in the speed 
   -- corridor. Takes the stating location (i.e. where the target speed must be 
   -- reached) and the required distance of the corridor and returns a 'corridor'
   -- of actuators which will have their displays changed.
   determineSpeedCorridor : int * nat ==> Corridor
   determineSpeedCorridor (startLoc, distance) ==
   (
   	 --The, intially empty, corridor
     dcl corr : Corridor := [] @
     (
       -- for each element in the sequence of actuators
       for index = 1 to len acts by 1 do
       (
         -- if  actuator is within the starting and end point of the required distance
         if (acts(index).loc >= startLoc and acts(index).loc <= startLoc+distance)
         -- add it to the corridor
         then corr := corr ^ [index]
       );
       return corr
     )	
   )
   pre len acts > 1 -- the TMS must have actuators
   post elems RESULT subset inds acts and 
        forall a in set elems RESULT @ 
          (acts(a).loc >= startLoc and acts(a).loc <= distance+startLoc)   
   
   
   -- Given the collection of actuators provided, enact the speed corridor up to 
   -- a target speed as given in the operation parameters.
   createSpeedCorridor : Corridor * nat ==> () 
   createSpeedCorridor (pCorr, target) ==
   (
     if target < nationalSpeedLimit then
     (
      dcl newDisp : nat := target, interval : nat, corrDistance : nat @
      (
       -- determine the speed difference between actuators
       -- needs to be +1 to enable smoother transition
       corrDistance := abs(acts(pCorr(1)).loc - acts(pCorr(len pCorr)).loc) + 1;
       interval := floor((nationalSpeedLimit - target)/corrDistance);
      
       -- for each actuator in the sequence, change the display value
       for index = (len pCorr) to 1  by -1 do
       (
         acts(pCorr(index)) := mk_Actuator(acts(pCorr(index)).aId, acts(pCorr(index)).loc, newDisp);
         newDisp := newDisp + interval
       )
      )
     )
   ) 
   pre elems pCorr subset inds acts and
       (len pCorr > 0 => target < nationalSpeedLimit)
   post elems pCorr subset inds acts and 
        forall a1, a2 in set elems pCorr @ (a1 <> a2 => 
         (acts(a1).loc > acts(a2).loc => acts(a1).disp >= acts(a2).disp))
   
   
   
   -- Given the speed corridor provided, remove the speed limits and re-enact the
   -- country's national speed limit on all actuator displays.
   disableSpeedCorridor : Corridor ==> ()
   disableSpeedCorridor(pCorr) ==
   (
       -- for each actuator in the sequence, change the display value
       for index = 1 to (len pCorr) by 1 do
       (
         acts(pCorr(index)) := mk_Actuator(acts(pCorr(index)).aId, acts(pCorr(index)).loc, nationalSpeedLimit)
       )
   )
   pre elems pCorr subset inds acts
   post elems pCorr subset inds acts and
        forall a in set elems pCorr @ acts(a).disp = nationalSpeedLimit
         --a should not be in any other corridors!
    
  
   
   -- Determines if Neighbour is needed based upon the starting location and 
   -- distance of the speed corridor. **IMPORTANT** current model of location 
   -- means this is only valid for Country B!
   isNeighbourNeeded : int * nat ==> bool
   isNeighbourNeeded(startLoc, distance) ==
   (
     return  not (exists ha in set elems acts @ ha.loc > (startLoc + distance))
   )
   -- post RESULT = not (exists act in set elems acts @ act.loc >= (startLoc + distance))
   
   
   --Operation to output the current speeds as shown on actuators. 
   outputActs: () ==> seq of nat
   outputActs() ==
   (
     --The, intially empty, set of values
     dcl v : seq of nat := [] @
     (
       -- for each element in the sequence of actuators
       for index = 1 to len acts by 1 do
       (
         v := v ^ [acts(index).disp]
       );
       return v
     )	
   )
   post RESULT = [acts(a).disp | a in set inds acts]
   
   
 actions
   
   BEHAVIOUR = NEW_INCIDENT [] NEIGHBOUR_REQ
   	                     
   NEW_INCIDENT = inIncident.myId?l?t -> 
                  (dcl d : nat := calcDistance(t, nationalSpeedLimit) @ 
                   NEW_CORRIDOR(l, t, d))
   				 
   NEW_CORRIDOR = l : int, t: nat, d:nat @ ACT_STATUS; 
               (dcl c: Corridor := determineSpeedCorridor(l, d) @ 
                 (newCorr.c -> createSpeedCorridor(c, t); ACT_STATUS;
     			  (dcl neighbourNeeded : bool := isNeighbourNeeded(l, d) @
                   ([neighbourNeeded]     & (dcl ntarg:nat := calcNeighbourTarget(l, t, d, nationalSpeedLimit) @
                                              neighbourRequest!myId!nId!ntarg -> CLEAR_INCIDENT(l, c,true))
                    []
                    [not neighbourNeeded] & CLEAR_INCIDENT(l, c, false)))))
              
   CLEAR_INCIDENT = l:int, c:Corridor, neigh:bool @ 
  			   (incidentClear.myId.l ->  CLEAR_CORRIDOR(c, neigh))
   			  
   CLEAR_CORRIDOR = c:Corridor,neigh:bool @ 
                     [neigh]& disableCorr.c -> disableSpeedCorridor(c); neighbourOk!myId!nId -> ACT_STATUS; NEW_INCIDENT
                     []
                     [not neigh] & disableCorr.c -> disableSpeedCorridor(c); ACT_STATUS;  BEHAVIOUR 
   
   NEIGHBOUR_REQ = neighbourRequest.nId.myId?t -> 
                        (dcl d : nat := calcDistance(t, nationalSpeedLimit) @ 
                           (dcl c: Corridor := determineSpeedCorridor(0, d) @ 
                               ACT_STATUS; newCorr.c -> createSpeedCorridor(c, t); ACT_STATUS;  NEIGHBOUR_CLEAR(c)))
                           
   NEIGHBOUR_CLEAR = c:Corridor @ 
                            (neighbourOk.nId.myId -> ACT_STATUS; disableCorr.c -> disableSpeedCorridor(c); ACT_STATUS; BEHAVIOUR)
                            
   ACT_STATUS = (dcl s : seq of nat := outputActs() @  actStatus.myId!s -> Skip)                   
                   
  @ BEHAVIOUR			
end 
  
process CountryA = CountryTMS(theAId, theBId, limitA, actCorrA)
process CountryB = CountryTMS(theBId, theAId, limitB, actCorrB)

process BorderTrafficSoS = CountryA [|interface|] CountryB --\\ internal


process Scenario = 
begin 
  @ inIncident!theBId!(-4)!20 -> incidentClear.theBId.(-4)-> inIncident.theBId.(-1).20 -> incidentClear.theBId.(-1) -> Skip 
end

process Test = Scenario [|incident|] BorderTrafficSoS


-- Values are used in the scenarios
values
 
  maxDiff : nat = 20
  maxChange : nat = 40
  
  theAId : CountryId = mk_token("Country A")
  theBId :CountryId = mk_token("Country B")
  
  limitA : nat = 100
  limitB : nat = 90
  
  aIdA1 : ActuatorId = mk_token("aIdA1")
  aIdA2 : ActuatorId = mk_token("aIdA2")
  aIdA3 : ActuatorId = mk_token("aIdA3")
  aIdA4 : ActuatorId = mk_token("aIdA4")
  
  aIdB1 : ActuatorId = mk_token("aIdB1")
  aIdB2 : ActuatorId = mk_token("aIdB2")
  aIdB3 : ActuatorId = mk_token("aIdB3")
  aIdB4 : ActuatorId = mk_token("aIdB4")
  
  actA1 : Actuator = mk_Actuator(aIdA1, 4, limitA)
  actA2 : Actuator = mk_Actuator(aIdA2, 3, limitA)
  actA3 : Actuator = mk_Actuator(aIdA3, 2, limitA)
  actA4 : Actuator = mk_Actuator(aIdA4, 1, limitA)
  
  actB1 : Actuator = mk_Actuator(aIdB1, -1, limitB)
  actB2 : Actuator = mk_Actuator(aIdB2, -2, limitB)
  actB3 : Actuator = mk_Actuator(aIdB3, -3, limitB)
  actB4 : Actuator = mk_Actuator(aIdB4, -4, limitB)
  
  actCorrA : ActuatorList = [actA1, actA2, actA3, actA4]
  actCorrB : ActuatorList = [actB1, actB2, actB3, actB4]
  
  restrictionSpeed : nat = 20
  blockageSpeed : nat = 5
  
  maxInterval : nat = 40
           
~~~
{% endraw %}

