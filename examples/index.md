---
layout: default
title: CML Examples
---

### AccountSys
This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model bank accounts and transactions made on these as a series of deposits and withdrawals. The CML model has been made by Peter Gorm Larsen. This example illustrate the use of records with invariants and
a state with the bank accounts indexed over the account number. All the operations are
defined implicitly. However the reactive part is not defined so that is left as an exercise for the reader.


| | |
|------|-------|
|Author:|Quentin Charatan and Aaron Kans|
|Details...|[model (zip)](AccountSys/AccountSys.zip)  / [show specification](AccountSys/index.html)|


### Airport
This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model the air traffic control of an airport at a very high level of abstraction. The CML model has been made by Jim Woodcock including adding the reactive behaviour.


| | |
|------|-------|
|Author:|Quentin Charatan and Aaron Kans|
|Details...|[model (zip)](Airport/Airport.zip)  / [show specification](Airport/index.html)|


### Alarm
This is the alarm example from the VDM-SL book, John Fitzgerald and Peter Gorm Larsen, Modelling Systems -- Practical Tools and Techniques in Software Development}, Cambridge University Press, 2nd edition 2009. The example is inspired by a subcomponent of a large alarm system developed by IFAD A/S. It is modelling the management of alarms for an industrial plant. The purpose of the model is to clarify the rules governing the duty roster and calling out of experts to deal with alarms. A comparable model of this example also exists in VDM++. This VDM-SL model has been translated to CML by Peter Gorm Larsen.


| | |
|------|-------|
|Author:|John Fitzgerald and Peter Gorm Larsen|
|Details...|[model (zip)](Alarm/Alarm.zip)  / [show specification](Alarm/index.html)|


### AVDeviceDiscovery
This is an example produced entirely inside the COMPASS project. The example illustrates the discovery protocol for new audio / video equipment when new devices are turned on/off.


| | |
|------|-------|
|Author:|Klaus Kristensen|
|Details...|[model (zip)](AVDeviceDiscovery/AVDeviceDiscovery.zip)  / [show specification](AVDeviceDiscovery/index.html)|


### AVDeviceDiscovery_MC
This is an example produced entirely inside the COMPASS project. 
The example illustrates the discovery protocol for new audio / video equipment 
when new devices are turned on/off. This version of the example can be processed 
using the COMPASS model checker. This cut-down version is produced by Adalberto Cajueiro.


| | |
|------|-------|
|Author:|Klaus Kristensen|
|Details...|[model (zip)](AVDeviceDiscovery_MC/AVDeviceDiscovery_MC.zip)  / [show specification](AVDeviceDiscovery_MC/index.html)|


### BEOCoSimulation_MC
This is an example produced entirely inside the COMPASS project. 
The example illustrates the CoSimulation example from B & O. 
This cut-down version of the example was produced by Klaus Kristensen and adapted by 
Adalberto Cajueiro to be processed using the COMPASS model checker.


| | |
|------|-------|
|Author:|Klaus Kristensen|
|Details...|[model (zip)](BEOCoSimulation_MC/BEOCoSimulation_MC.zip)  / [show specification](BEOCoSimulation_MC/index.html)|


### BEOStreaming_MC
This is an example produced entirely inside the COMPASS project.
The example illustrates the Streaming model from B & O. 
This cut-down version of the example was produced by Klaus Kristensen and adapted by 
Adalberto Cajueiro to be processed using the COMPASS model checker.


| | |
|------|-------|
|Author:|Klaus Kristensen|
|Details...|[model (zip)](BEOStreaming_MC/BEOStreaming_MC.zip)  / [show specification](BEOStreaming_MC/index.html)|


### BitRegister
This simple bit register example has been provided in VDM by a student called Andreas Mueller from Austria. It has been translated to CML by Jim Woodcock and used for testing the Symphony tool.

This example defines the main operators you can conduct on a very simple calculator where you can have registers you can store and load values to and from, and then add and subtract. Note that it only handles bytes up to 255 so it is possible to see the effect of overflows and underflows.

Just like for the Stack example the RegisterProc process can be debugged and it is possible to load values into the register and try to add and subtract them from each other. Here the natural behaviour of the bit register (including underflow and overflow scenarios) can be exercised. Note that you also can set breakpoints and inspect the different variables when the simulation is stopped at such breakpoints.

In the current version there are a number of proof obligations generated that you will not be able to discharge. Please update the argument types for some of the operations so this can be improved.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Details...|[model (zip)](BitRegister/BitRegister.zip)  / [show specification](BitRegister/index.html)|


### BitRegister_MC
This simple bit register example has been provided in VDM by a student called Andreas Mueller from Austria. 
It has been translated to CML by Jim Woodcock and used for testing the Symphony tool.

This example defines the main operators you can conduct on a very simple calculator where you can have 
registers you can store and load values to and from, and then add and subtract. The original version handles 
bytes up to 255. This cut-down version was produced by Adalberto Cajueiro to handle bytes up to 4 so it is 
possible to see the effect of overflows and underflows in the model checker.

The deadlock found by the model checker produced the same trace in the animator (except for the tau-events).
A good exercise it to run these two plugins to observe the results. 

The original version uses an input as the value to be incremented/decremented. This version uses a value 
that can be changed in each execution/analysis.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Details...|[model (zip)](BitRegister_MC/BitRegister_MC.zip)  / [show specification](BitRegister_MC/index.html)|


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
|Details...|[model (zip)](ConwayOlympian/ConwayOlympian.zip)  / [show specification](ConwayOlympian/index.html)|


### DataFlow
The data flow machine transforms its sequence of values in its input channel into a sequence of computations in its output channel. It shows how the use of parallel processes can speed up the computation. An important idiom is introduced: the use of ghost variables and a ghost process to demonstrate the correctness of the computation. A second important idiom is the use of data independence in model checking: the behaviour of the data flow machine does not depend on decisions taken on the basis of its inputs, it merely transforms the data. This means that checks for deadlock and livelock freedom of the network of processes in the machine can be carried out with synchronisations rather than communications. In this case, the infinite state data flow machine can be easily model checked with only 24 states.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Details...|[model (zip)](DataFlow/DataFlow.zip)  / [show specification](DataFlow/index.html)|


### Dphils_MC
This simple dining philosophers models the classical example considering two philosofers.

The deadlock found by the model checker produced the same trace in the animator (except for the tau-events).
A good exercise it to run these two plugins to observe the results.


| | |
|------|-------|
|Author:|Adalberto Cajueiro|
|Details...|[model (zip)](Dphils_MC/Dphils_MC.zip)  / [show specification](Dphils_MC/index.html)|


### Dwarf
This is a CML Specification of the Dwarf Signal Controller. The CML
model made in this document is a combination of a VDM-SL model produced by Peter Gorm Larsen and a CSP model produced by Jim Woodcock which again are inspired by the Dwarf Signal control
system described by Marcus Montigel, Alcatel Austria AG. These models was presented at a FM Railway
workshop. The CML model of this has been produced jointly by Peter Gorm Larsen, Jim Woodcock and Alvaro Miyazawa.


| | |
|------|-------|
|Author:|Peter Gorm Larsen|
|Details...|[model (zip)](Dwarf/Dwarf.zip)  / [show specification](Dwarf/index.html)|


### FaultTolerance
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](FaultTolerance/FaultTolerance.zip)  / [show specification](FaultTolerance/index.html)|


### Frogs
This example is the frog jumping puzzle that was used in the 
introductionary CSP training of CSP at the first COMPASS meeting.
This example illustrates a lot of invariants but no concurrency.
The CML model is written by Jim Woodcock.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Details...|[model (zip)](Frogs/Frogs.zip)  / [show specification](Frogs/index.html)|


### HotelLock
The final example is rather larger than the other examples: it models the card-key system found in most hotels, where there is no network connecting locks to the front desk, and yet a security property emerges. This demonstrates an idiom for emergence in CML, with an abstract, Olympian view of the specification satisfying a global invariant that is then completely distributed to components that interact only indirectly. Modelling idioms presented include restricting communicable values on channels and debugging techniques for tracing behaviour through parallel processes.


| | |
|------|-------|
|Author:|Jim Woodcock and Peter Gorm Larsen|
|Details...|[model (zip)](HotelLock/HotelLock.zip)  / [show specification](HotelLock/index.html)|


### IncubatorController
This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model an incubator controller at a very abstract level. The CML model of this has been produced by Jim Woodcock. The basic incubation controller example is simply a device that continually monitors and controls a temperature that either can be increased with an Increment operation or decreased with a Decrement operation. This example illustrate the use of guards since there is a minimum and a maximum temperature. Whenever the inc and dec channels are used the temperature represented with the state component temp is incremented and decremented by one. Pre and post-conditions are also incorporated in this example.

The IncubatorController process can be debugged but no input is expected from the user at all (except for choosing to go up or down in temperature). Note that you also can set breakpoints and inspect the temp variable when the simulation is stopped at such breakpoints. Note also that it is easy to reach a deadlock by giving a value outside the state invariant. Try it and find out how to repair this.


| | |
|------|-------|
|Author:|Quentin Charatan and Aaron Kans|
|Details...|[model (zip)](IncubatorController/IncubatorController.zip)  / [show specification](IncubatorController/index.html)|


### IncubatorMonitor
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](IncubatorMonitor/IncubatorMonitor.zip)  / [show specification](IncubatorMonitor/index.html)|


### InfComm_MC
This example shows processes that communicate data from infinite domains. 
It is useful to show how the CML model checker is able to provide results whereas traditional
tools (like FDR and PAT) are not. 

For the process P, it is required that the user modify the Model Checker Setup preferences to 2, so 
the model checker instantiates two integers that are sufficient to produce a deadlock.

For the recursive process PRec, only one instance of natural numbers communicated in each channel 
(possibly different) is enough to produce a deadlock. Thus, it is required that the user 
modify the Model Checker Setup preferences to 1.


| | |
|------|-------|
|Author:|Adalberto Cajueiro|
|Details...|[model (zip)](InfComm_MC/InfComm_MC.zip)  / [show specification](InfComm_MC/index.html)|


### InsielCUSSoS_FT
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](InsielCUSSoS_FT/InsielCUSSoS_FT.zip)  / [show specification](InsielCUSSoS_FT/index.html)|


### InsielCUSSoS_MC
This example was provided by Paolo and modified by Adalberto Cajueiro. 
It represents the CUSSos model from Insiel.


| | |
|------|-------|
|Author:|Paolo|
|Details...|[model (zip)](InsielCUSSoS_MC/InsielCUSSoS_MC.zip)  / [show specification](InsielCUSSoS_MC/index.html)|


### LevelCrossing
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](LevelCrossing/LevelCrossing.zip)  / [show specification](LevelCrossing/index.html)|


### Library
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](Library/Library.zip)  / [show specification](Library/index.html)|


### PatientRegister
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](PatientRegister/PatientRegister.zip)  / [show specification](PatientRegister/index.html)|


### ProcessManager
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](ProcessManager/ProcessManager.zip)  / [show specification](ProcessManager/index.html)|


### RingBuffer
This project needs a description.


| | |
|------|-------|
|Author:||
|Details...|[model (zip)](RingBuffer/RingBuffer.zip)  / [show specification](RingBuffer/index.html)|


### Scheduler
This system schedules a collection of processes to ensure that only one is in a critical region for using some resource. The system shows how to build a network of processes and run the execution engine to test the result. A refinement is to require that the processes are permitted to enter the critical region in the order in which they became ready to do so. The idiom used here is to strengthen the behaviour of the system by adding a process in parallel that constrains behaviour as required. The result is a formal refinement. This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Details...|[model (zip)](Scheduler/Scheduler.zip)  / [show specification](Scheduler/index.html)|


### Sieve
The prime number generator is based on the classic algorithm of the Sieve of Eratosthenes, but implemented in a dynamic systolic array. The example shows how to use concurrency, recursion, and hiding together in quite a subtle way to implement an infinite computation in a lazy, reactive behaviour. It shows how to encode the use of dynamic channel and process creation with tokens, important idioms in CML for this pattern for reconfigurable systems.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Details...|[model (zip)](Sieve/Sieve.zip)  / [show specification](Sieve/index.html)|


### Simple-Minimondex
This CML model has been developed by Jim Woodcock inspired by the original work done with the Mondex secure card system (see [Stepney&00,Woodcock&08a]. In 1994, National Westminster Bank developed an electronic
purse (or smart card) system, called Mondex. An
electronic purse is a card-sized device intended to replace "real"
coins with electronic cash. In contrast to a credit or debit card,
an electronic purse stores its balance in itself, thus does not 
necessarily require any network access to update a remote database
during a transaction. So, electronic purses can be used in small
stores or shops, such as bakeries, where small amounts of money
are involved. In the CML model here however, the different potential faults that could happen in these transactions are not taken into account.

When debugging the MiniMondex example one will get an option to select between ten different cards that wish to transfer an amount of money to another card. Here you can experiment with transferring money between cards and see how it can communicate either over the accept or the reject channel. Experiment with sending money from a card to itself and with transferring more money than the amount left on a particular Card.


| | |
|------|-------|
|Author:|Jim Woodcock|
|Details...|[model (zip)](Simple-Minimondex/Simple-Minimondex.zip)  / [show specification](Simple-Minimondex/index.html)|


### Stack
This is the standard stack example written in CML. If you wish to play with this you can add an invariant to put a limit to the stack size.


| | |
|------|-------|
|Author:|Peter Gorm Larsen|
|Details...|[model (zip)](Stack/Stack.zip)  / [show specification](Stack/index.html)|

