---
layout: default
title: AVDeviceDiscovery
---

## AVDeviceDiscovery
Author: Klaus Kristensen


This is an example produced entirely inside the COMPASS project. The example illustrates the discovery protocol for new audio / video equipment when new devices are turned on/off. 



| Properties | Values          |
| :------------ | :---------- |
|Language Version:| cml|


### BeoAVDeviceDiscovery.cml

{% raw %}
~~~
  types
    Device_Record = nat -- uniq device ID
    ServiceCallData = nat  -- abstation for the service call data structure being sent over the network
    DeviceEvent = (<INTERUPT>|<POWER_ON>|<POWER_OFF>) -- user device event
    TCP_PortEvent = (<CONNECT>|<DISCONNECT>) -- IP TCP events using for adding IP connect/disconnect logic to channels

  values
    timeout : nat = 1 -- generic IP timeout
    sourceBroadcastTime : nat = 2 -- timeouts for the source product
    targetBroadcastTime : nat = 2 -- timout for the targte product  these values can be used for playing with the wait time for the processes
    ServiceCallDataERROR:ServiceCallData = 0  -- error return value to the application layer
  
  functions
   IsDeviceInSet:Device_Record * set of Device_Record ->bool -- helper function 
   IsDeviceInSet(d, s)== d not in set s   
  
  channels 
--- channels for the DD and SD part of the SoS
    IPMulticastChannel:Device_Record
    SourceProductPowerChannel:DeviceEvent
    TargetProductPowerChannel:DeviceEvent

--- channels for the service part of the SoS
   SourceProductEventChannel:DeviceEvent
   TargetProductEventChannel:DeviceEvent
   BrowsingInterfaceCallChannel:ServiceCallData
   BrowsingInterfaceReplyChannel:ServiceCallData
   IPTCPChannel:ServiceCallData
   IPTCPPORTChannel:TCP_PortEvent
   LocalInterfaceChannel:ServiceCallData
  
  --chansets --CHANSET DO NOT WORK
  -- DD_SR_chanset = {|IPMulticastChannel,SourceProductPowerChannel,TargetProductPowerChannel|}
   --SR_chanset = {|SourceProductEventChannel,TargetProductEventChannel,BrowsingInterfaceCallChannel,BrowsingInterfaceReplyChannel,IPTCPChannel |}

-- CML process modelling the B&O DD protocol for the source product in the network
  process SourceProduct_DD_SD_InterfaceProtocolView =
  begin
	  state
         product_DR:Device_Record := 2 -- local device id              
      operations      
         createDDSDObject :()==>() --update the data with service status info, needs to be done before each broadcast
         createDDSDObject()== product_DR := 2 -- just simulate updating the device record
	  actions	  
		  Off_State = SourceProductPowerChannel.<POWER_ON>->On_State -- product can be powered on by a user event
		  On_State = DD_SD_Announcement_State /_\(SourceProductPowerChannel.<POWER_OFF>->Off_State)  -- the Announcement loop can be interupted by a user event 
		  DD_SD_Announcement_State = (MarshallData_State; Wait(sourceBroadcastTime);  DD_SD_Announcement_State) -- Announce the device in the network using multicast
		  MarshallData_State = ( createDDSDObject();IPMulticastChannel!product_DR->Skip)[_ timeout _> (Skip) --- simulate IP multicast logic using timeouts  					   
      @ Off_State
  end
-- CML process modelling the B&O DD protocol for the target product in the network 
  process TargetProduct_DD_SD_InterfaceProtocolView =
  begin
	  state
         product_DRs:set of Device_Record := {} -- set of Ddiscoveed products
	
	  operations
	     resetProductSet:()==>()-- reset the device set then the product is power on
	     resetProductSet() == product_DRs := {}
	     
	     updateProductSet:Device_Record==>()-- add device to the set, if the pre condition holds
	     updateProductSet(dr)== product_DRs := product_DRs union {dr}  
	     pre dr not in set product_DRs
	     post dr in set product_DRs 
	      
	  
	  actions
		  Off_State = resetProductSet(); TargetProductPowerChannel.<POWER_ON>->On_State -- the product if off== not visable in the network
		  On_State = Start_Discovery_State  /_\(TargetProductPowerChannel.<POWER_OFF>->Off_State) -- product on = try finding other B&O product. in this state the product can be power of by a user event
		  Start_Discovery_State = (DD_SD_Discovery_State ; Wait(targetBroadcastTime) ; Start_Discovery_State)--run the Discovery logic 
		  DD_SD_Discovery_State =  (IPMulticastChannel?pr->if not IsDeviceInSet(pr,product_DRs) then updateProductSet(pr); Skip )[_ timeout _> (Skip) 		                     
    @ Off_State                     -- simulate IP multicast logic using timeouts
  end	
   
-- SoS leveel process, contaning one source- and one target product
  process Beo_DD_SD_InterfaceProtocolViews =  SourceProduct_DD_SD_InterfaceProtocolView [|{IPMulticastChannel}|]TargetProduct_DD_SD_InterfaceProtocolView
  
  --- test process for playing the SoS level process
  process Test_TurnOnProduct = 
  begin
   actions
    SoS_On = SourceProductPowerChannel.<POWER_ON>->TargetProductPowerChannel.<POWER_ON>->Wait 5;SoS_Off
    SoS_Off = SourceProductPowerChannel.<POWER_OFF>->TargetProductPowerChannel.<POWER_OFF>->Wait 1;Skip
    
  @ SoS_On
  end
  -- test process combining the test process and the SoS process
  process Test_1 = Test_TurnOnProduct [|{SourceProductPowerChannel,TargetProductPowerChannel}|]Beo_DD_SD_InterfaceProtocolViews
  
  
  ----------------------------------------------------------------------------------------------------------------------------------------------------------
 -- section contain the processes for calling AV services over the network, they are currently not described in the case doc
 
  process TargetProduct_SR_InterfaceProtocolView =
  begin
	  actions
		  Off_State = (TargetProductEventChannel.<POWER_ON>->On_State) 
		  On_State = WaitingOnFunctionCall_State /_\ TargetProductEventChannel.<POWER_OFF>->Off_State 
		  WaitingOnFunctionCall_State = BrowsingInterfaceCallChannel?x->ConnectToDevice_State(x)
		  ConnectToDevice_State= x:ServiceCallData @  (IPTCPPORTChannel.<CONNECT>->SendingBrowsingRequest_State(x)) 
		  						 [_ timeout _>BrowsingInterfaceReplyChannel!ServiceCallDataERROR-> WaitingOnFunctionCall_State 
		  SendingBrowsingRequest_State = x:ServiceCallData @ SendCallData_State(x) 
		  						/_\ (IPTCPPORTChannel.<DISCONNECT>->BrowsingInterfaceReplyChannel!ServiceCallDataERROR->WaitingOnFunctionCall_State 
		  					    [] TargetProductEventChannel.<INTERUPT>->WaitingOnFunctionCall_State)   
		  SendCallData_State = x:ServiceCallData @ (IPTCPChannel!x->IPTCPChannel?y->ReplayTofunctionCall_State(y))
		  						[_ timeout _> BrowsingInterfaceReplyChannel!ServiceCallDataERROR->WaitingOnFunctionCall_State
		  ReplayTofunctionCall_State = x:ServiceCallData @ BrowsingInterfaceReplyChannel!x->WaitingOnFunctionCall_State  
      @ Off_State
  end 
  
  process SourceProduct_SR_InterfaceProtocolView =
  begin   
	  actions
		  Off_State = SourceProductEventChannel.<POWER_ON>->On_State
		  On_State = WaitingOnServiceCall_State /_\ SourceProductEventChannel.<POWER_OFF>->Off_State
		  WaitingOnServiceCall_State = IPTCPPORTChannel.<CONNECT>->ReadCallData_State
		  ReadCallData_State = ReadCallData /_\ (SourceProductEventChannel.<INTERUPT>->WaitingOnServiceCall_State [] IPTCPPORTChannel.<DISCONNECT> ->WaitingOnServiceCall_State)   
		  ReadCallData = (IPTCPChannel?x->LocalInterfaceChannel!x->LocalInterfaceChannel?y->IPTCPChannel!y->WaitingOnServiceCall_State) [_ timeout _>WaitingOnServiceCall_State    
      @ Off_State
  end
  
  process Beo_RS_InterfaceProtocolViews =  TargetProduct_SR_InterfaceProtocolView [|{IPTCPChannel,IPTCPPORTChannel}|]SourceProduct_SR_InterfaceProtocolView
  
  
~~~
{% endraw %}

