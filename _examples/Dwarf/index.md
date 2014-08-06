---
layout: default
title: Dwarf
---

## Dwarf
Author: Peter Gorm Larsen


This is a CML Specification of the Dwarf Signal Controller. The CML
model made in this document is a combination of a VDM-SL model produced by Peter Gorm Larsen and a CSP model produced by Jim Woodcock which again are inspired by the Dwarf Signal control
system described by Marcus Montigel, Alcatel Austria AG. These models was presented at a FM Railway
workshop. The CML model of this has been produced jointly by Peter Gorm Larsen, Jim Woodcock and Alvaro Miyazawa.



### Dwarf.cml

{% raw %}
~~~
types
  LampId      = <L1> | <L2> | <L3>
  Signal      = set of LampId
  ProperState = Signal
  inv ps == ps in set {dark, stop, warning, drive}

  DwarfType :: lastproperstate    : ProperState
               turnoff            : set of LampId 
               turnon             : set of LampId
               laststate          : Signal
               currentstate       : Signal
               desiredproperstate : ProperState
  inv d == 
    (((d.currentstate \ d.turnoff) union d.turnon) = d.desiredproperstate) and
    (d.turnoff inter d.turnon = {}) and
    -- NeverShow All
    d.currentstate <> {<L1>,<L2>,<L3>} and
    -- MaxOneLampChange
    card ((d.currentstate \ d.laststate) union (d.laststate \ d.currentstate)) <= 1 and
    -- ForbidStopToDrive
    ((d.lastproperstate = stop) => d.desiredproperstate <> drive) and
    -- DarkOnlyToStop
    ((d.lastproperstate = dark) => d.desiredproperstate in set {dark,stop}) and
    -- DarkOnlyFromStop
    ((d.desiredproperstate = dark) => d.lastproperstate in set {dark,stop})
    
  DwarfSignal = DwarfType
  inv d == NeverShowAll(d) and 
  		    MaxOneLampChange(d) and 
  		    ForbidStopToDrive(d) and 
  		    DarkOnlyToStop(d) and 
  		    DarkOnlyFromStop(d)

values
  dark: Signal    = {}
  stop: Signal    = {<L1>, <L2>}
  warning: Signal = {<L1>, <L3>}
  drive: Signal   = {<L2>, <L3>}

functions

  NeverShowAll: DwarfType -> bool
  NeverShowAll(d) == d.currentstate <> {<L1>,<L2>,<L3>}
  
  MaxOneLampChange: DwarfType -> bool 
  MaxOneLampChange(d) == card ((d.currentstate \ d.laststate) union (d.laststate \ d.currentstate)) <= 1
  
  ForbidStopToDrive : DwarfType -> bool
  ForbidStopToDrive(d) == ((d.lastproperstate = stop) => d.desiredproperstate <> drive)
  
  DarkOnlyToStop : DwarfType -> bool
  DarkOnlyToStop(d) == ((d.lastproperstate = dark) => d.desiredproperstate in set {dark,stop}) 
  
  DarkOnlyFromStop: DwarfType -> bool
  DarkOnlyFromStop(d) == ((d.desiredproperstate = dark) => d.lastproperstate in set {dark,stop})

channels
  init
  light: LampId
  extinguish: LampId
  setPS: ProperState
  shine: Signal
  thunk

process Dwarf = begin 

state 
  dw : DwarfType

operations
  Init : () ==> ()
  Init() ==
    dw := mk_DwarfType(stop, {}, {}, stop, stop, stop) 
    post dw.lastproperstate = stop and 
         dw.turnoff = {} and 
         dw.turnon = {} and 
         dw.laststate = stop and 
         dw.currentstate = stop and 
         dw.desiredproperstate = stop

  SetNewProperState: (ProperState) ==> ()
  SetNewProperState(st) ==
    dw := mk_DwarfType( dw.currentstate
    	                , dw.currentstate \ st
    	                , st \ dw.currentstate
    	                , dw.laststate
    	                , dw.currentstate
    	                , st) 
  
  	pre dw.currentstate = dw.desiredproperstate and
  	    st <> dw.currentstate
  	     
  	post dw.lastproperstate = dw~.currentstate and
  	     dw.turnoff = dw~.currentstate \ st and
  	     dw.turnon  = st \ dw~.currentstate and
  	     dw.laststate = dw~.laststate and
  	     dw.currentstate = dw~.currentstate and
  	     dw.desiredproperstate = st

  TurnOn: (LampId) ==> ()
  TurnOn(l) == 
    dw := mk_DwarfType( dw.lastproperstate
                      , dw.turnoff \ {l}
                      , dw.turnon \ {l}
                      , dw.currentstate
                      , dw.currentstate union {l}
                      , dw.desiredproperstate)
                                          
    pre l in set dw.turnon
    post dw.lastproperstate = dw~.lastproperstate and
         dw.turnoff = dw~.turnoff \ {l} and
         dw.turnon  = dw~.turnon  \ {l} and 
         dw.laststate = dw~.currentstate and
         dw.currentstate = dw~.currentstate union {l} and
         dw.desiredproperstate = dw~.desiredproperstate


  TurnOff : (LampId) ==> ()
  TurnOff(l) == 
    dw := mk_DwarfType( dw.lastproperstate
                      , dw.turnoff \ {l}
                      , dw.turnon \ {l}
                      , dw.currentstate
                      , dw.currentstate \ {l}
                      , dw.desiredproperstate)
  
    pre l in set dw.turnoff
    post dw.lastproperstate = dw~.lastproperstate and
         dw.turnoff = dw~.turnoff \ {l} and
         dw.turnon  = dw~.turnon  \ {l} and 
         dw.laststate = dw~.currentstate and
         dw.currentstate = dw~.currentstate \ {l} and
         dw.desiredproperstate = dw~.desiredproperstate

  actions
    DWARF =
      (  (light?l -> TurnOn(l); DWARF) 
      [] (extinguish?l -> TurnOff(l); DWARF)
      [] (setPS?l -> SetNewProperState(l); DWARF)
      [] (shine!(dw.currentstate) -> DWARF))
      
    -- Working test
    TEST1 = setPS!warning -> extinguish!<L2> -> light!<L3> 
          -> setPS!drive -> extinguish!<L1> -> light!<L2> -> Stop

    -- Tries to turn on 3 lights simulaneously
    TEST2 = setPS!warning -> light!<L3> -> extinguish!<L2> 
          -> setPS!drive -> extinguish!<L1> -> light!<L2> -> Stop

    -- Tries to go from dark to warning
    TEST3 = setPS!dark -> extinguish!<L1> -> extinguish!<L2> -> setPS!warning -> 
            light!<L1> -> light!<L2> -> Stop 
          
    -- Tries to go from stop to drive
    TEST4 = setPS!drive -> extinguish!<L1> -> light!<L3> -> Stop

    DWARF_TEST1 = DWARF [|{setPS,light,extinguish}|] TEST1
    DWARF_TEST2 = DWARF [|{setPS,light,extinguish}|] TEST2      
    DWARF_TEST3 = DWARF [|{setPS,light,extinguish}|] TEST3
    DWARF_TEST4 = DWARF [|{setPS,light,extinguish}|] TEST4
    
@ 

init -> Init() ; DWARF_TEST3

end
~~~
{% endraw %}

