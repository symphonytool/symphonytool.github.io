---
layout: default
title: InsielCUSSoS_MC
---

## InsielCUSSoS_MC
Author: Paolo


This example was provided by Paolo and modified by Adalberto Cajueiro. 
It represents the CUSSos model from Insiel.  


### CUSSoS_MC.cml

{% raw %}
~~~
channels 

init

CUStoCSChannel
CStoCUSChannel
CStoRSChannel
CStoMSChannel
RStoCSChannel
MStoCSChannel

noconnection, lostconnection, wrongdata
reconnect, resend

chansets
E = {| noconnection, lostconnection, wrongdata |}
F = {| lostconnection, wrongdata |}
H = {| reconnect, resend |}

Alpha = {|reconnect, resend, CUStoCSChannel, CStoCUSChannel, 
CStoRSChannel, CStoMSChannel,RStoCSChannel, MStoCSChannel |}
Alpha_CUS = Alpha
Alpha_ERU = Alpha
Alpha_CS = Alpha


process CS =
begin
	actions
	
	STARTUP = RECEIVE_FROM_RS [] RECEIVE_FROM_MS [] RECEIVE_FROM_CUS
	RECEIVE_FROM_RS = RStoCSChannel -> CStoCUSChannel -> STARTUP
	RECEIVE_FROM_MS = MStoCSChannel -> CStoCUSChannel -> STARTUP
	RECEIVE_FROM_CUS = CUStoCSChannel -> (RS_SEND [] RS_LOST_CONNECTION [] RS_WRONG_DATA [] RS_NO_CONNECTION)
	
	RS_SEND = CStoRSChannel -> STARTUP
	RS_LOST_CONNECTION = lostconnection -> reconnect -> resend -> RECEIVE_FROM_CUS
	RS_WRONG_DATA = wrongdata -> resend -> RECEIVE_FROM_CUS
	RS_NO_CONNECTION = noconnection -> ((CStoMSChannel -> STARTUP)
										[] (lostconnection -> reconnect -> resend -> RECEIVE_FROM_CUS)
										[] (wrongdata -> resend -> RECEIVE_FROM_CUS)
										[] (noconnection -> Skip))
	
	@init -> STARTUP
end


-- This process conforms to the CUS contract
process ERU =
begin
   actions
    	STARTUP = RStoCSChannel -> STARTUP 
    				[] MStoCSChannel -> STARTUP
					[] CStoRSChannel-> STARTUP 
					[] CStoMSChannel -> STARTUP
					[] reconnect -> STARTUP
 @init -> STARTUP
end

process CUS = 
begin
	actions
		STARTUP = SEND [] (CStoCUSChannel ->STARTUP) [] (resend -> SEND)
    	SEND = CUStoCSChannel -> STARTUP
  @init -> STARTUP
end
	
	
process CUSSoS = CS [|Alpha|] (ERU ||| CUS)
~~~
{% endraw %}

