---
layout: default
title: CML Examples
---

### AccountSys
This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model bank accounts and transactions made on these as a series of deposits and withdrawals. The CML model has been made by Peter Gorm Larsen.


| | |
|------|-------|
|Author:|Quentin Charatan and Aaron Kans|
|Version:|CML - cml|
|Details...|[model (zip)](AccountSys/AccountSys.zip)  / [show specification](AccountSys/index.html)|


### Airport
This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model the air traffic control of an airport at a very high level of abstraction. The CML model has been made by Jim Woodcock including adding the reactive behaviour.


| | |
|------|-------|
|Author:|Quentin Charatan and Aaron Kans|
|Version:|CML - cml|
|Details...|[model (zip)](Airport/Airport.zip)  / [show specification](Airport/index.html)|


### Alarm
This is the alarm example from the VDM-SL book, John Fitzgerald and Peter Gorm Larsen, Modelling Systems -- Practical Tools and Techniques in Software Development}, Cambridge University Press, 2nd edition 2009. The example is inspired by a subcomponent of a large alarm system developed by IFAD A/S. It is modelling the management of alarms for an industrial plant. The purpose of the model is to clarify the rules governing the duty roster and calling out of experts to deal with alarms. A comparable model of this example also exists in VDM++. This VDM-SL model has been translated to CML by Peter Gorm Larsen.


| | |
|------|-------|
|Author:|John Fitzgerald and Peter Gorm Larsen|
|Version:|CML - cml|
|Details...|[model (zip)](Alarm/Alarm.zip)  / [show specification](Alarm/index.html)|


### AVDeviceDiscovery
This is an example produced entirely inside the COMPASS project. The example illustrates the discovery protocol for new audio / video equipment when new devices are turned on/off.


| | |
|------|-------|
|Author:|Klaus Kristensen|
|Version:|CML - cml|
|Details...|[model (zip)](AVDeviceDiscovery/AVDeviceDiscovery.zip)  / [show specification](AVDeviceDiscovery/index.html)|


### AVDeviceDiscovery_MC
This is an example produced entirely inside the COMPASS project. The example illustrates the discovery protocol for new audio / video equipment when new devices are turned on/off. This version of the example can be processed using the COMPASS model checker. This cut-down version is produced by Adalberto Cajueiro.


| | |
|------|-------|
|Author:|Klaus Kristensen|
|Version:|CML - cml|
|Details...|[model (zip)](AVDeviceDiscovery_MC/AVDeviceDiscovery_MC.zip)  / [show specification](AVDeviceDiscovery_MC/index.html)|


### BEOCoSimulation_MC



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](BEOCoSimulation_MC/BEOCoSimulation_MC.zip)  / [show specification](BEOCoSimulation_MC/index.html)|


### BEOStreaming_MC



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](BEOStreaming_MC/BEOStreaming_MC.zip)  / [show specification](BEOStreaming_MC/index.html)|


### BitRegister
This simple bit register example has been provided in VDM by a student called Andreas Mueller from Austria. It has been translated to CML by Jim Woodcock and used for testing the Symphony tool.

This example defines the main operators you can conduct on a very simple calculator where you can have registers you can store and load values to and from, and then add and subtract. Note that it only handles bytes up to 255 so it is possible to see the effect of overflows and underflows.

Just like for the Stack example the RegisterProc process can be debugged and it is possible to load values into the register and try to add and subtract them from each other. Here the natural behaviour of the bit register (including underflow and overflow scenarios) can be exercised. Note that you also can set breakpoints and inspect the different variables when the simulation is stopped at such breakpoints.

In the current version there are a number of proof obligations generated that you will not be able to discharge. Please update the argument types for some of the operations so this can be improved.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Version:|CML - cml|
|Details...|[model (zip)](BitRegister/BitRegister.zip)  / [show specification](BitRegister/index.html)|


### BitRegister_MC



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](BitRegister_MC/BitRegister_MC.zip)  / [show specification](BitRegister_MC/index.html)|


### ChoiceSupportInDebugger



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](ChoiceSupportInDebugger/ChoiceSupportInDebugger.zip)  / [show specification](ChoiceSupportInDebugger/index.html)|


### ConwayOlympian
This example illustrate Conways game of life (see http://en.wikipedia.org/wiki/Conway's_Game_of_Life) which has has interesting emergent properties. A VDM-SL version of this example was produced by Nick Battle and Peter Gorm Larsen and Claus Ballegaard Nielsen produced a graphical user interface showing the evolution of life following the rules of the game of life. The universe of the Game of Life is an infinite two-dimensional orthogonal grid of square cells, each of which is in one of two possible states, alive or dead. Every cell interacts with its eight neighbours, which are the cells that are horizontally, vertically, or diagonally adjacent. At each step in time, the following transitions occur:
1.Any live cell with fewer than two live neighbours dies, as if caused by under-population.
2.Any live cell with two or three live neighbours lives on to the next generation.
3.Any live cell with more than three live neighbours dies, as if by overcrowding.
4.Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.
A CML version of this was produced by Peter Gorm Larsen but no graphical user interface is available at this stage. In addition, this CML model does not yet have any any reactive bahaviour so it cannot be animated in the Symphony tool at this stage.


| | |
|------|-------|
|Author:|Peter Gorm Larsen, Claus Ballegaard Nielsen and Nick Battle|
|Version:|CML - cml|
|Details...|[model (zip)](ConwayOlympian/ConwayOlympian.zip)  / [show specification](ConwayOlympian/index.html)|


### DataFlow



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](DataFlow/DataFlow.zip)  / [show specification](DataFlow/index.html)|


### Dphils_MC



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](Dphils_MC/Dphils_MC.zip)  / [show specification](Dphils_MC/index.html)|


### Dwarf
This is a CML Specification of the Dwarf Signal Controller. The CML
model made in this document is a combination of a VDM-SL model produced by Peter Gorm Larsen and a CSP model produced by Jim Woodcock which again are inspired by the Dwarf Signal control
system described by Marcus Montigel, Alcatel Austria AG. These models was presented at a FM Railway
workshop. The CML model of this has been produced jointly by Peter Gorm Larsen, Jim Woodcock and Alvaro Miyazawa.


| | |
|------|-------|
|Author:|Peter Gorm Larsen|
|Version:|CML - cml|
|Details...|[model (zip)](Dwarf/Dwarf.zip)  / [show specification](Dwarf/index.html)|


### FaultTolerance



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](FaultTolerance/FaultTolerance.zip)  / [show specification](FaultTolerance/index.html)|


### Frogs
This example is the frog jumping puzzle that was used in the 
introductionary CSP training of CSP at the first COMPASS meeting.
This example illustrates a lot of invariants but no concurrency.
The CML model is written by Jim Woodcock.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Version:|CML - cml|
|Details...|[model (zip)](Frogs/Frogs.zip)  / [show specification](Frogs/index.html)|


### HotelLock



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](HotelLock/HotelLock.zip)  / [show specification](HotelLock/index.html)|


### IncubatorController
This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model an incubator controller at a very abstract level. The CML model of this has been produced by Jim Woodcock. The basic incubation controller example is simply a device that continually monitors and controls a temperature that either can be increased with an Increment operation or decreased with a Decrement operation. This example illustrate the use of guards since there is a minimum and a maximum temperature. Whenever the inc and dec channels are used the temperature represented with the state component temp is incremented and decremented by one. Pre and post-conditions are also incorporated in this example.

The IncubatorController process can be debugged but no input is expected from the user at all (except for choosing to go up or down in temperature). Note that you also can set breakpoints and inspect the temp variable when the simulation is stopped at such breakpoints. Note also that it is easy to reach a deadlock by giving a value outside the state invariant. Try it and find out how to repair this.


| | |
|------|-------|
|Author:|Quentin Charatan and Aaron Kans|
|Version:|CML - cml|
|Details...|[model (zip)](IncubatorController/IncubatorController.zip)  / [show specification](IncubatorController/index.html)|


### IncubatorMonitor



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](IncubatorMonitor/IncubatorMonitor.zip)  / [show specification](IncubatorMonitor/index.html)|


### InfComm_MC



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](InfComm_MC/InfComm_MC.zip)  / [show specification](InfComm_MC/index.html)|


### InsielCUSSoS_FT



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](InsielCUSSoS_FT/InsielCUSSoS_FT.zip)  / [show specification](InsielCUSSoS_FT/index.html)|


### InsielCUSSoS_MC



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](InsielCUSSoS_MC/InsielCUSSoS_MC.zip)  / [show specification](InsielCUSSoS_MC/index.html)|


### LevelCrossing



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](LevelCrossing/LevelCrossing.zip)  / [show specification](LevelCrossing/index.html)|


### Library



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](Library/Library.zip)  / [show specification](Library/index.html)|


### MiniMondex



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](MiniMondex/MiniMondex.zip)  / [show specification](MiniMondex/index.html)|


### PatientRegister



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](PatientRegister/PatientRegister.zip)  / [show specification](PatientRegister/index.html)|


### ProcessManager



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](ProcessManager/ProcessManager.zip)  / [show specification](ProcessManager/index.html)|


### RingBuffer



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](RingBuffer/RingBuffer.zip)  / [show specification](RingBuffer/index.html)|


### Scheduler



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](Scheduler/Scheduler.zip)  / [show specification](Scheduler/index.html)|


### Sieve



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](Sieve/Sieve.zip)  / [show specification](Sieve/index.html)|


### Simple-Minimondex



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](Simple-Minimondex/Simple-Minimondex.zip)  / [show specification](Simple-Minimondex/index.html)|


### Stack
This is the standard stack example written in CML. If you wish to play with this you can add an invariant to put a limit to the stack size.


| | |
|------|-------|
|Author:|Peter Gorm Larsen|
|Version:|CML - cml|
|Details...|[model (zip)](Stack/Stack.zip)  / [show specification](Stack/index.html)|


### Tests



| | |
|------|-------|
|Author:||
|Version:|CML - classic|
|Details...|[model (zip)](Tests/Tests.zip)  / [show specification](Tests/index.html)|

