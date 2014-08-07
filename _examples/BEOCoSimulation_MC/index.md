---
layout: default
title: BEOCoSimulation_MC
---

## BEOCoSimulation_MC
Author: Klaus Kristensen


This is an example produced entirely inside the COMPASS project. 
The example illustrates the CoSimulation example from B & O. 
This cut-down version of the example was produced by Klaus Kristensen and adapted by 
Adalberto Cajueiro to be processed using the COMPASS model checker. 


### BEOCoSimulation_MC.cml

{% raw %}
~~~
types

TCP_Event = <CONNECT>|<DISCONNECT> 
CoSimulationProtocol = <RegisterSubSystemMessage>|<InspectMessage>|<InspectReplyMessage>|<ExecuteMessage>|<ExecuteCompletedMessage>|<FinishedRequestMessage>|<FinishedReplyMessage>|<DisconnectMessage> 

channels

ch_tcp:TCP_Event |CoSimulationProtocol
ch_init



process CoSimulationServer =
begin
 state
   isExecuteable:bool := false

 operations
   setExeState:bool==>()
   setExeState(b)== isExecuteable := b

actions

   act_serviceLoop = ch_tcp.<CONNECT>->(act_connected /_\ (ch_tcp.<DISCONNECT>->Skip[]ch_tcp.<DisconnectMessage>->Skip))
   
   act_connected = ch_tcp.<RegisterSubSystemMessage>->act_clientProtocolLoop
   
   act_clientProtocolLoop = ( act_inspecting []act_finished [] [isExecuteable]& act_executing); act_clientProtocolLoop
   
   act_inspecting = ch_tcp.<InspectMessage>->ch_tcp.<InspectReplyMessage>->(setExeState(true));Skip 
   
   act_executing = ch_tcp.<ExecuteMessage>->ch_tcp.<ExecuteCompletedMessage>->(setExeState(false));Skip
   
   act_finished = ch_tcp.<FinishedRequestMessage>->ch_tcp.<FinishedReplyMessage>->Skip 

   @ ch_init-> (act_serviceLoop /_\ch_init->Skip)

end

process CoSimulationClient =
begin

state
   isExecuteable:bool := false


operations

   setExeState:bool==>()
   setExeState(b)== isExecuteable := b


actions

   act_serviceLoop = ch_tcp.<CONNECT>->(act_connected /_\ (ch_tcp.<DISCONNECT>->Skip[]ch_tcp.<DisconnectMessage>->Skip))
   
   act_connected = ch_tcp.<RegisterSubSystemMessage>->act_clientProtocolLoop
   
   act_clientProtocolLoop = ( act_inspecting []act_finished [] [isExecuteable]& act_executing); act_clientProtocolLoop
   
   act_inspecting = ch_tcp.<InspectMessage>->ch_tcp.<InspectReplyMessage>->(setExeState(true));Skip 
   
   act_executing = ch_tcp.<ExecuteMessage>->ch_tcp.<ExecuteCompletedMessage>->(setExeState(false));Skip
   
   act_finished = ch_tcp.<FinishedRequestMessage>->ch_tcp.<FinishedReplyMessage>->Skip 

   @ ch_init->(act_serviceLoop /_\ch_init->Skip)

end

process CoSimlulationClientAndServer =   CoSimulationServer  [|{|ch_init,ch_tcp |}|]CoSimulationClient
~~~
{% endraw %}

