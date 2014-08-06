---
layout: default
title: BEOStreaming_MC
---

## BEOStreaming_MC
Author: 


This project needs a description.


### BEO_StreamingSoS_MC.cml

{% raw %}
~~~
types

-- inteface events for the steaming layer
SOURCE_NODE = <FILE_SOURCE_NODE>| <NETRADIO_SOURCE_NODE>|<MUSIC_SERVICE_SOURCE_NODE>|<AUX_SOURCE_NODE>
StreamingLayerControlEvent = <CONNECT>|<DISCONNECT>|<SETUPSTREAM>|<PLAY>|<STOP>|<INTERUPT>--- state transition events
StreamingLayerReplyEvent = <CONNECTED>|<DISCONNECTED>|<STREAM_READY>|<PLAYING>|<STOPPED>|<INIT> --async events
StreamingLayerErrorEvent = <STATE_ERROR> --async events

-- control layer user events
controlLayerUserEvent = <CONNECT>|<DISCONNECT>|<SETUPSTREAM>|<PLAY>|<STOP>|<INTERUPT>-- state transition events

channels

ch_init -- channel for starting and shurting down the processes

-- channels for the streaming layer interface
ch_streamingControl:StreamingLayerControlEvent
ch_streamingReply:StreamingLayerReplyEvent
ch_streamingLayerError:StreamingLayerErrorEvent

-- local channels for the control layer interface
ch_controlLayerUserEvent:StreamingLayerControlEvent--controlLayerUserEvent -- for now we just use the streaming layer events
ch_controlLayerInterupt
ch_genericEvent:StreamingLayerReplyEvent


//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// this process are supose to be very VDM heavy, and the intened use is for verification
// using the COMPASS PO and TO tools

process StreamingPlayerCSProcess =
begin

state
  currentState:StreamingLayerReplyEvent := <INIT>
  preState:StreamingLayerReplyEvent := <INIT>
    
  operations
    changeState:StreamingLayerReplyEvent==>()
    changeState(s) == 
    (
     preState := currentState;
     currentState := s
    )
      
actions
   -- action fro starting the main product statemachine, can be interupted by a sync on the ch_init channel
   act_mainEntryPointState = act_processLoop /_\ ch_init->Skip  
   
   -- main product state machine action. offes some events based on the current state of the product
   act_processLoop =   
                   (
                       [currentState <> <DISCONNECTED> ]& act_disconnect --  most always offer to disconnect after the product is connected
                       []
                       [currentState = <CONNECTED> or currentState = <STOPPED>]& act_connected -- if the product is not playing or is connected it offes prepare a song
                       []
                       [currentState = <DISCONNECTED> ]& act_waitOnConnect -- if the product is disconnected offer to connect
                       []
                       [currentState = <STREAM_READY> ]& act_readyToPlayState  -- if we have prepared a song offer to play it
                       []
                       [currentState = <PLAYING> ]& act_playingState -- if the product is playing a song offer to stop it
                    );
                    act_processLoop -- do not use a mu x. not supported by the model checker
                       
   --action for the connect state of the product
   act_waitOnConnect = ch_streamingControl.<CONNECT>->
   					   (-- the product could transit to the connect state. sends a ok reply back to the control layer CS
   					    ch_streamingReply.<CONNECTED>->changeState(<CONNECTED>) ;Skip 
   					    []-- the product could not trantis to the conncetd state during to some local errors, send a NACK back to the control layer CS
   					    ch_streamingLayerError.<STATE_ERROR>->Skip
   					   )
   					   
   -- action for preparing a song, meaning trying to connect to the contends provider of the stream like 
   --  a musinc service or netradio stations or just a local file					    
   act_connected =    ch_streamingControl.<SETUPSTREAM>->
    					( -- the product can play the stream. sends a OK back to the control layer CS and change state to ready to play
    					ch_streamingReply.<STREAM_READY>->(changeState(<STREAM_READY>);Skip) 
    					[]-- could not connect to the provider or cannot decode the stream. send a NACK back to the control layer CS
    					ch_streamingLayerError.<STATE_ERROR>->(changeState(<CONNECTED>));Skip //change the state back to connected, no stream are ready
    					                                                                       //even if we had a stream ready before
    					) 
 
   act_readyToPlayState = ch_streamingControl.<PLAY>->
                          (
                            ch_streamingReply.<PLAYING>->changeState(<PLAYING>);Skip 
                           [] 
                           ch_streamingLayerError.<STATE_ERROR>->Skip
                           )
                           []
                            act_connected
   
   act_playingState = ch_streamingControl.<STOP>->
                         (
                           ch_streamingReply.<STOPPED>->changeState(<CONNECTED>);Skip 
                           []
                           ch_streamingLayerError.<STATE_ERROR>->Skip
                          )
   
   act_disconnect = (ch_streamingControl.<DISCONNECT>->ch_streamingReply.<DISCONNECTED>->changeState(<DISCONNECTED>);Skip) 
   
   @ch_init ->act_mainEntryPointState
end



/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//test sections
// CML processes are model tranlations of the Req ARCE framework vaidation view points
//  Sunshine process, testing a CS sourcegraph refineement
process TestTraces1 = -- test the sunshine seq of the interface impl. 
begin

actions
    act_connect = ch_streamingControl.(<CONNECT>)->ch_streamingReply.<CONNECTED>->act_setstream
    act_setstream = ch_streamingControl.(<SETUPSTREAM>)->ch_streamingReply.(<STREAM_READY>)->act_play 
    act_play = ch_streamingControl.(<PLAY>)->ch_streamingReply.(<PLAYING>)->act_stop 
    act_stop = ch_streamingControl.(<STOP>)->ch_streamingReply.(<STOPPED>)->act_disconnect
    act_disconnect = ch_streamingControl.(<DISCONNECT>)->ch_streamingReply.(<DISCONNECTED>)->ch_init->Skip
@ ch_init->act_connect ///_\ ch_init->Skip reomve this for auto testing, the tool will chosse this event at stop the test right away. not that we want.....

end



//Test SoS process
process TestTPCS1 = TestTraces1 [|{|ch_init,ch_genericEvent,ch_streamingReply|}|]StreamingPlayerCSProcess -- do not syn aopn the error channel, want to deadlock i any sync happens on the error reply channel

////////////////////////////////////////////////////////////////////////////////////////////
//  test process, testing a CS sourcegraph refineement of the interface rqa regarding must alway reply (error or OK)
// in this case we ask the sourcegraph to do a no-legal transition from connected to connected 
process TestTraces2 =
begin

actions
    act_connect = ch_streamingControl.(<CONNECT>)->ch_streamingReply.<CONNECTED>->act_error_connect
    act_error_connect = ch_streamingControl.(<CONNECT>)->ch_streamingLayerError.<STATE_ERROR>->act_setstream  -- deadloack if we dont get a error
    act_setstream = ch_streamingControl.(<SETUPSTREAM>)->ch_streamingReply.(<STREAM_READY>)->act_play 
    act_play = ch_streamingControl.(<PLAY>)->ch_streamingReply.(<PLAYING>)->act_stop 
    act_stop = ch_streamingControl.(<STOP>)->ch_streamingReply.(<STOPPED>)->act_disconnect
    act_disconnect = ch_streamingControl.(<DISCONNECT>)->ch_streamingReply.(<DISCONNECTED>)->ch_init->Skip
@ ch_init-> act_connect

end

//Test SoS process
process TestTPCS2 = TestTraces2 [|{|ch_init,ch_genericEvent,ch_streamingReply,ch_streamingLayerError|}|]StreamingPlayerCSProcess



~~~
{% endraw %}

