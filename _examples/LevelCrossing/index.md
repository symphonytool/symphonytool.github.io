---
layout: default
title: LevelCrossing
---

## LevelCrossing
Author: 


This project needs a description.


### LevelCrossing.cml

{% raw %}
~~~
-- Based on Bill Roscoe's CSP specification

channels tock

-- The following are the communications between the controller process and the gate process

types
  gatestates = <godown> | <goup> | <up> | <down>

-- where we can think of the first two as being commands to it, and the
-- last two as being confirmations from a sensor that they are up or down.

channels
  gate: gatestates

-- For reasons discussed below, we introduce a special error event:

channel error

-- To model the speed of trains, and also the separation of more than one train, we divide the track into
-- segments that the trains can enter or leave.

types
  TRAINS = <Thomas> | <Duncan>

values
  N = 5

types
-- Here, segment 0 represents the outside world, and 1-4 actual track segments including the crossing,
  TRACKS = nat
    inv t == t in set {0,...,N}

  TRACKSEGS = nat
    inv t == t in set {1,...,N}
  
-- which is at

values
  GateSeg = N-1

-- To make modular arithmetic convenient, we define

channels
  enter, leave: TRAINS * TRACKS

-- Trains are detected when they enter the first track segment by a sensor that drives the controller, 
-- and are also detected by a second sensor when they leave.

types
  sensorsig = <inseg> | <outseg>

channels
  sensor: sensorsig


-- Untimed trains and tracks

-- The following gives an untimed description of Train A on track segment j. A train not currently in 
-- the domain of interest is given index 0.

process Trains =
begin
  actions
    Train = A: TRAINS, j: TRACKS @
      enter.A.((j+1) mod (N+1)) -> leave.A.j -> Train(A,(j+1) mod (N+1))
@
  Train(<Thomas>,0) ||| Train(<Duncan>,0)
end

-- The real track segments can be occupied by one train at a time, and each time a train enters
-- segment 1 or leaves GateSeg the sensors fire.

process Track = j: TRACKS @
begin
  actions
    Tracknf = j: TRACKS @
      if j=1 then
        enter?A!j -> sensor.<inseg> -> Trackf(A,j) 
      else
        enter?A!j -> Trackf(A,j)
        
    Trackf = A: TRAINS, j: TRACKS @
      leave.A.j ->
        if j = GateSeg then
          sensor.<outseg> -> Tracknf(j)
        else
          Tracknf(j)
@
  Tracknf(j)
end

-- Like the trains, the untimed track segments do not communicate with each other

process Tracks = ||| i: TRACKSEGS @ Track(i)

-- And we can put together the untimed network, noting that since there is
-- no process modelling the outside world there is no need to synchronise
-- on the enter and leave events for this area.

process Network =
  Trains [|({|enter,leave|} \ {|enter.A.0 | A: TRAINS|}) \ {|leave.A.0 | A: TRAINS|}|] Tracks

-- Speed assumptions on trains

-- We make assumptions about the speed of trains by placing (uniform)
-- upper and lower "speed limits" on the track segments:

values
  MinTocksPerSeg = 3

  MaxTocksPerSeg = 6

-- The speed regulators express bounds on the times between successive
-- enter events.

--chansets
--  AlphaSR(j) = {tock,enter.A.j,enter.A.(j+1) mod (N+1) | A <- TRAINS}

process SpeedRegs = 
begin
  actions
    SpeedReg = j: TRACKS @
      enter?A!j -> SpeedReg'(j,0)
      []
      tock -> SpeedReg(j)

  SpeedReg' = j: TRACKS, n: int @
    [n < MaxTocksPerSeg] & tock -> SpeedReg'(j,n+1) 
    []
    [n >= MinTocksPerSeg] & enter?A!((j+1) mod (N+1)) -> SpeedReg(j)
@
  || j: TRACKSEGS @ 
    [ {} | {tock}
      union {enter.A.j | A: TRAINS}
      union {enter.A.((j+1) mod (N+1)) | A: TRAINS}
    ] SpeedReg(j)
end

-- The following pair of processes express the timing contraint that
-- the two sensor events occur within one time unit of a train entering
-- or leaving the domain.


-- The timing constraints of the trains and sensors are combined into the
-- network as follows, noting that no speed limits are used outside the domain:

process SensorTiming = 
begin
  actions
    InSensorTiming =
      tock -> InSensorTiming
      []
      enter?A!1 -> sensor.<inseg> -> InSensorTiming

    OutSensorTiming = 
      tock -> OutSensorTiming
      []
      leave?A!GateSeg -> sensor.<outseg> -> OutSensorTiming
@
  InSensorTiming [|{|tock|}|] OutSensorTiming
end

process NetworkTiming = SpeedRegs [|{tock} union {enter.A.1 | A: TRAINS}|] SensorTiming

process TimedNetwork =
  Network [|{|enter,sensor|} union {leave.A.GateSeg | A: TRAINS}|] NetworkTiming


-- Gate and controller

-- The last components of our system are a controller and the gate.
-- The duties of the controller
-- are to ensure that the gate is always down when there is a train on the
-- gate, and that it is up whenever prudent.


-- When the gate is up, the controller does nothing until the sensor
-- detects an approaching train.  
-- In this state, time is allowed to pass arbitrarily, except that the
-- signal for the gate to go down is sent immediately on the occurrence of
-- the sensor event.

process Controller =
begin
  actions
    ControllerUp =
      sensor.<inseg> -> gate!<godown> -> ControllerGoingDown(1)
      []
      sensor.<outseg> -> ERROR
      []
      tock -> ControllerUp

    ControllerGoingDown = n: TRACKSEGS @
      ( if GateSeg < n then ERROR else sensor.<inseg> -> ControllerGoingDown(n+1) )
      []  gate.<down> -> ControllerDown(n)
      []  tock -> ControllerGoingDown(n)
      []  sensor.<outseg> -> ERROR

    ControllerDown = n: TRACKSEGS @
      ( if GateSeg < n then ERROR else sensor.<inseg> -> ControllerDown(n+1))
      [] sensor.<outseg> -> 
           ( if n=1 then gate!<goup> -> ControllerGoingUp else ControllerDown(n-1) )
      []
      tock -> ControllerDown(n)

    ControllerGoingUp =
      gate!<up> -> ControllerUp
      [] tock -> ControllerGoingUp
      [] sensor.<inseg> -> gate!<godown> -> ControllerGoingDown(1)
      [] sensor.<outseg> -> ERROR

  ERROR = error -> Stop
@
  ControllerUp
end

-- The two states ControllerGoingDown and ControllerDown 
-- both keep a record of how many trains
-- have to pass before the gate may go up.  Each time the sensor event
-- occurs this count is increased.
-- The count should not get greater than the number of trains that
-- can legally be between the sensor and the gate (which equals the
-- number of track segments).
-- The ControllerGoingDown state comes to an end when the gate.down event occurs
              
-- When the gate is down, the occurrence of a train entering its sector
-- causes no alarm, and each time a train leaves the gate sector the
-- remaining count goes down, or the gate is signalled to go up, as appropriate.
-- Time is allowed to pass arbitrarily in this state, except that the
-- direction to the gate to go up is instantaneous when due.

-- When the gate is going up, the inward sensor may still fire, which means that
-- the gate must be signalled to go down again.
-- Otherwise the gate goes up after UpTime units.

-- Any process will be allowed to generate an error event, and since we will
-- be establishing that these do not occur, we can make the successor process
-- anything we please, in this case STOP.

-- The following are the times we assume here for the gate to go up
-- and go down.  They represent upper bounds in each case.

values
  DownTime = 5

  UpTime = 2

process Gate =
begin
  actions
    GateUp =
      gate.<goup> -> GateUp
      [] gate.<godown> -> GateGoingDown(0)
      [] tock -> GateUp

    GateGoingDown = n: nat @
      gate.<godown> -> GateGoingDown(n)
      [] 
      ( if n = DownTime then
          gate.<down> -> GateDown
        else
          ( gate.<down> -> GateDown
		    |~|
		    tock -> GateGoingDown(n+1) ) )

    GateDown =
      gate.<godown> -> GateDown
      []
      gate.<goup> -> GateGoingUp(0)
      [] 
      tock -> GateDown
      
    GateGoingUp = n: nat @
      gate.<goup> -> GateGoingUp(n)
      []
      gate.<godown> -> GateGoingDown(0)
      [] ( if n = UpTime then
             gate.<up> -> GateUp
	       else
	         ( gate.<up> -> GateUp
		       |~|
		       tock -> GateGoingUp(n+1) ) )
@
  GateUp
end

process GateAndController = Controller [|{|tock,gate|}|] Gate

process System = TimedNetwork [|{|sensor,tock|}|] GateAndController

~~~
{% endraw %}

