---
layout: default
title: InsielCUSSoS_FT
---

## InsielCUSSoS_FT
Author: 


This project needs a description.


| Properties | Values          |
| :------------ | :---------- |
|Language Version:| cml|


### CUSSoS_FT.cml

{% raw %}
~~~
types
     Unwanted = <NoRSConnection> | <NoMSConnection> 
     			| <LostRSConnection> | <LostMSConnection> 
     			| <WrongRSData> | <WrongMSData>
     
channels 

	init, finish
	
	CUStoCSChannel
	CStoCUSChannel
	CStoRSChannel
	CStoMSChannel
	RStoCSChannel
	MStoCSChannel

	unwanted: Unwanted
	
	reconnect, resend

chansets
    H = {| reconnect, resend, finish |}

    Alpha = {| init, finish, reconnect, resend, CUStoCSChannel, CStoCUSChannel, 
	   CStoRSChannel, CStoMSChannel, RStoCSChannel, MStoCSChannel |}

    Alpha_CUSSoS = Alpha

process Limit_CUSSoS = Limit

process Limit =
begin
     actions
          Init = Choose
          Choose = RS [] MS
          RS = NRSC [] LRSC [] WRSD
          MS = LMSC [] WMSD
          NRSC = unwanted.<NoRSConnection> -> Choose
          LRSC = unwanted.<LostRSConnection> -> Choose
          WRSD = unwanted.<WrongRSData> -> Choose
          LMSC = unwanted.<LostMSConnection> -> Choose
          WMSD = unwanted.<WrongMSData> -> Choose
     @ Init
end
 
process CS =
begin
    actions
    
    --STARTUP = RECEIVE_FROM_RS [] RECEIVE_FROM_MS [] RECEIVE_FROM_CUS -- to make this action MC-compliant
    STARTUP = RECEIVE_FROM_RS [] RECEIVE_FROM_CUS -- to make this action MC-compliant
    RECEIVE_FROM_RS = RStoCSChannel -> CStoCUSChannel -> STARTUP
    --RECEIVE_FROM_MS = MStoCSChannel -> CStoCUSChannel -> STARTUP -- to make this action MC-compliant
    RECEIVE_FROM_CUS = CUStoCSChannel -> RS_TRY
    
    RS_TRY = RS_SEND [] RS_LOST_CONNECTION [] RS_WRONG_DATA [] RS_NO_CONNECTION
    RS_SEND = CStoRSChannel -> STARTUP
    RS_LOST_CONNECTION = unwanted.<LostRSConnection> -> reconnect -> resend -> RS_SEND
    RS_WRONG_DATA = unwanted.<WrongRSData> -> resend -> RS_SEND
    RS_NO_CONNECTION = unwanted.<NoRSConnection> -> MS_TRY
    MS_TRY = MS_SEND [] MS_LOST_CONNECTION [] MS_WRONG_DATA [] MS_NO_CONNECTION
    --MS_SEND = CStoMSChannel -> STARTUP -- to make this action MC-compliant
    MS_SEND = CStoRSChannel -> STARTUP
    MS_LOST_CONNECTION = unwanted.<LostMSConnection> -> reconnect -> resend -> MS_SEND
    MS_WRONG_DATA = unwanted.<WrongMSData> -> resend -> MS_SEND
    MS_NO_CONNECTION = unwanted.<NoMSConnection> -> finish -> Skip -- channel finish to avoid deadlock
    
    @ (init -> STARTUP) 
end


-- This process conforms to the CUS contract
process ERU =
begin
    actions
        STARTUP = 
            (RStoCSChannel -> STARTUP) 
            --[] MStoCSChannel -> STARTUP -- to make this action MC-compliant
            [] (CStoRSChannel -> STARTUP) 
            --[] CStoMSChannel -> STARTUP -- to make this action MC-compliant
            [] (reconnect -> STARTUP)
            [] (finish -> Skip) -- channel finish to avoid deadlock
    @ init -> STARTUP
end

process CUS = 
begin
    actions
        STARTUP = 
            SEND 
            [] (CStoCUSChannel -> STARTUP) 
            [] (resend -> STARTUP) 
            [] (finish -> Skip) -- channel finish to avoid deadlock
        SEND = CUStoCSChannel -> STARTUP
    @ init -> STARTUP
end


process CUSSoS = 
    CS 
    [| Alpha |] 
    (ERU [| {|init,finish|} |] CUS) -- channel finish to avoid deadlock

-- If MC supports renaming, then the MC-compliance comments shall be replaced 
-- by following code:
--process CUSSoS_FT = 
--    CUSSoS [[ RStoCSChannel <- MStoCSChannel, CStoRSChannel <- CStoMSChannel ]]
~~~
{% endraw %}

