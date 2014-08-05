---
layout: default
title: RingBuffer
---

## RingBuffer
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### RingBuffer.cml

{% raw %}
~~~
     types
        Value = nat
        CellId = nat inv id == id > 0 and id <= maxring
        Direction =  <req> | <ack>

    values
        maxbuff = 4
        maxring = maxbuff - 1

    channels
        input, output : Value
        write, read: CellId * Direction * Value
        rrd, wrt: Direction * Value
        rd_i, wrt_i: CellId * Direction * Value

    process RingCell =
    begin
        state v:Value

        operations
            setV:Value ==> ()
            setV(x) ==
               v := x

            OrigsetV(x:Value)
                frame wr v
                post v = x

        actions
            Act = wrt.<req>?x -> wrt.<ack>.x -> setV(x); Act
                  []
                  rrd.<req>?dumb -> rrd.<ack>!v -> Act

        @ Act
    end

    process Controller =
    begin
        state cache:Value
              size:nat
              top:CellId
              bot:CellId

        operations
            Init: Value * nat * CellId * CellId ==> ()
            Init(c,s,t,b) ==
              (cache := c;
               size := s;
               top := t;
               bot := b)
               
             SetCache: Value ==> ()
             SetCache(x) ==
                cache := x

            SetSize: nat ==> ()
            SetSize(x) ==
               size := x

            SetTop: CellId ==> ()
            SetTop(x) ==
              top := x

            SetBot: CellId ==> ()
            SetBot(x) ==
              bot := x
             
             InitOrig(c:Value, s:nat, t:CellId, b:CellId)
                post cache=c and size=s and top=t and bot=b

            InitSetCache(x:Value)
                frame wr cache:Value
                post cache = x

            InitSetSize(x:nat)
                frame wr size:nat
                post size = x

            InitSetTop(x:CellId)
                frame wr top:CellId
                post top = x

            InitSetBot(x:CellId)
                frame wr bot:CellId
                post bot = x

        actions
            Input =
                [size < maxbuff] &
                    input?x ->
                        ( [size = 0] & SetCache(x); SetSize(1)
                          []
                          [size > 0] &
                                write.top.<req>!x -> write.top.<ack>?dumb ->
                                InitSetSize(size+1);
                                InitSetTop((top mod maxring)+1) )

            Output =
                [size > 0] &
                    output!cache ->
                        ( [size > 1] &
                            (|~| dumb:Value @
                                read.bot.<req>.dumb -> read.bot.<ack>?x -> Skip);
                            InitSetSize(size-1);
                            InitSetBot((bot mod maxring)+1)
                          []
                          [size = 1] &
                            InitSetSize(0))

        @ Init(0,0,1,1); mu X @ ((Input [] Output); X)
    end

~~~
{% endraw %}

