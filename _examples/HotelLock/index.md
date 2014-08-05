---
layout: default
title: HotelLock
---

## HotelLock
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### HotelLock.cml

{% raw %}
~~~
values

  debug = false -- true 

  maxkeys:   nat = 2
  maxrooms:  nat = 2
  maxguests: nat = 2

  KeySet      : set of nat = {1,...,maxkeys}
  RoomSet     : set of nat = {1,...,maxrooms}
  GuestSet    : set of nat = {1,...,maxguests}
  KeySetSet   : set of set of nat = power KeySet 
  RoomSetSet  : set of set of nat = power RoomSet 
  GuestSetSet : set of set of nat = power GuestSet 

types 
  Key = nat
    inv k == k in set KeySet
    
  Room = nat
    inv r == r in set RoomSet
  
  Guest = nat
    inv g == g in set GuestSet
  
  Card :: fst : Key
          snd : Key 
values
  CardSet = { mk_Card(f,s) | f,s in set KeySet @ f <> s }
  
  CardSetSet = power CardSet

types 
  Desk :: issued : set of Key
          prev   : map Room to Key
    inv d == rng d.prev subset d.issued 

channels
  choosekey   :   set of Key
  chooseguest : (set of Card) * Key
  checkin     : Guest * Room
  enter       : Room * Guest
  issuecard
  addroom     : Room * Key
  addguest    : Guest * set of Card

channels
  debugdesk     : Desk
  debuglocks    : map Room to Key
  debugguests   : map Guest to set of Card
  debuglastkey  : nat
  debuglastcard : nat

process Hotel =
begin
  state 
    desk      : Desk := mk_Desk({},{|->})
    locks     : map Room to Key := {|->}
    guests    : map Guest to set of Card := {|->}
    lastkey   : nat := 0
    lastcard  : nat := 0
    inv dom desk.prev subset dom locks
        and lastkey in set KeySet union {0}
        and dunion { {c.fst, c.snd} | c in set dunion rng guests } subset desk.issued
        and lastcard in set KeySet union {0}

  actions
    Debug = b: bool @
      if debug 
      then
        debugdesk!desk ->
          debuglocks!locks ->
            debugguests!guests ->
             debuglastkey!lastkey ->
               debuglastcard!lastcard -> 
                 Skip
      else
        Skip
        
    Debug1 = b: bool @ Skip -- Debug(b)

    Debug2 = b: bool @ Debug(b)

  functions
    preChooseKey: set of Key -> bool
    preChooseKey(s) == card s <> 0
 
    preChooseCard: set of Card -> bool
    preChooseCard(s) == card s <> 0

    preCheckIn: Room * Desk -> bool 
    preCheckIn(r,desk) == r in set dom desk.prev
  
    preEnter: Room * Guest * (map Room to Key) * (map Guest to set of Card) -> bool
    preEnter(r,g,locks,guests) ==
      r in set dom locks
      and g in set dom guests
      and exists c in set guests(g) @ c.fst=locks(r) or c.snd=locks(r)

    preAddRoom: Room * Key * Desk * (map Room to Key) -> bool
    preAddRoom(r,k,desk,locks) == k in set desk.issued and r not in set dom locks

    preAddGuest: Guest * (set of Card) * Desk -> bool
    preAddGuest(g,cs,desk) == 
      forall c in set cs @
        {c.fst, c.snd} subset desk.issued
        
    operations  
    -- ChooseKey(s: set of Key) r: Key
    --   pre  prechoiceKey(s)
    --   post lastkey = lastkey~+1 and r = lastkey

    -- ChooseGuest(s: set of Card, k: Key) c: Card
    --   pre  prechoiceCard(s)
    --   post lastcard = lastcard~+1
    --        and ( (c.fst = k and c.snd = lastcard)
    --              or (c.fst = lastcard and c.snd = k) )
                 
    -- CheckIn(g:Guest,r:Room)
    --   frame wr desk   : Desk
    --         wr guests : map Guest to set of Card
    --   pre r in set dom desk.prev
    --   post exists new_k:Key @
    --     new_k not in set desk~.issued
    --     and 
    --       let new_c = mk_Card(desk~.prev(r),new_k)
    --       in
    --         desk.issued = desk~.issued union {new_k} and 
    --         desk.prev = desk~.prev ++ {r |-> new_k}  and 
    --         if g in set dom guests~
    --         then 
    --           guests = guests~ ++ {g |-> guests~(g) union {new_c}}
    --         else 
    --           guests = guests~ munion {g |-> {new_c}}

    -- Enter(r:Room,g:Guest)
    --   frame wr locks  : map Room to Key
    --         rd guests : map Guest to set of Card
    --   pre r in set dom locks  and 
    --       g in set dom guests and
    --       exists c in set guests(g) @ c.fst=locks(r) or c.snd=locks(r) 
    --   post exists c in set guests(g) @ 
    --       c.fst = locks(r) and locks = locks~ ++ {r |-> c.snd} 
    --       or
    --       c.snd = locks(r) and locks = locks~

    ExplChooseKey: set of Key ==> Key
    ExplChooseKey(s) ==
      (lastkey := lastkey+1; return lastkey)
      pre true -- preChooseKey(s)
      
    ExplChooseGuest1: (set of Card) * Key ==> Card
    ExplChooseGuest1(s,k) ==
      ( lastcard := lastcard+1; return mk_Card(k,lastcard) )
      pre preChooseCard(s)

    ExplChooseGuest2: (set of Card) * Key ==> Card
    ExplChooseGuest2(s,k) ==
      ( lastcard := lastcard+1; return mk_Card(lastcard,k) )
      pre  preChooseCard(s)

    ExplCheckIn: Guest * Room ==> ()
    ExplCheckIn(g,r) ==
      let new_k = ExplChooseKey(desk.issued)
      in 
        let new_c = mk_Card(desk.prev(r),new_k)
        in
          ( desk.issued := desk.issued union {new_k}; 
            desk.prev   := desk.prev ++ {r |-> new_k}; 
            guests      := if g in set dom guests
                           then 
                             guests ++ {g |-> guests(g) union {new_c}}
                           else 
                             guests munion {g |-> {new_c}}
          )
      pre r in set dom desk.prev

    --
    -- Explicit versions of operations for execution on the interpreter
    --

    ExplEnter: Room * Guest ==> ()
    ExplEnter(r,g) ==
      let c = ExplChooseGuest1(guests(g),locks(r))
      in 
        if c.fst = locks(r)
        then locks := locks ++ {r |-> c.snd}
      pre r in set dom locks  and 
        g in set dom guests and
        exists c in set guests(g) @ c.fst=locks(r) or c.snd=locks(r)

    IssueCard: () ==> Key
    IssueCard() ==
      let k = ExplChooseKey(desk.issued)
      in
        ( desk.issued := desk.issued union {k};
          return k
        )

    AddRoom: Room * Key ==> ()
    AddRoom(r,k) ==
      ( desk.prev := desk.prev munion {r |-> k};
        locks := locks munion {r |-> k}
      )
      pre k in set desk.issued and r not in set dom locks

    AddGuest: Guest * set of Card ==> ()
    AddGuest(g,cs) ==
      guests := guests ++ {g |-> if g in set dom guests
                                 then guests(g) union cs
                                 else cs}
    pre preAddGuest(g,cs,desk)

actions 
  Interface =
    ( [] ks in set KeySetSet @ 
        [preChooseKey(ks)] &
          choosekey.ks -> 
            Debug1(debug);
            ( dcl r: Key @ r := ExplChooseKey(ks) );
            Debug2(debug) )
    []
    ( [] cs in set CardSetSet, k in set KeySet @ 
        [preChooseCard(cs)] & 
          chooseguest.cs.k -> 
            Debug1(debug);
            ( dcl c: Card @ c := ExplChooseGuest1(cs,k) );
            Debug2(debug) )
    []
    ( [] g in set GuestSet, r in set RoomSet @
        [preCheckIn(r,desk)] & 
          checkin.g.r ->
            Debug1(debug);
            ExplCheckIn(g,r);
            Debug2(debug) )
    []
    ( [] r in set RoomSet, g in set GuestSet @
        [preEnter(r,g,locks,guests)] &
          enter.r.g ->
            Debug1(debug);
            ExplEnter(r,g);
            Debug2(debug) )
    []
    ( issuecard ->
        Debug1(debug);
        IssueCard();
        Debug2(debug) )
    []
    ( [] r in set RoomSet, k in set KeySet @
        [preAddRoom(r,k,desk,locks)] &
          addroom.r.k ->
            Debug1(debug);
            AddRoom(r,k);
            Debug2(debug) )
    []
    ( [] g in set GuestSet, s in set CardSetSet @
        [preAddGuest(g,s,desk)] &
          addguest.g.s ->
            Debug1(debug);
            AddGuest(g,s);
            Debug2(debug) )
@
  while true do Interface
end
~~~
{% endraw %}

