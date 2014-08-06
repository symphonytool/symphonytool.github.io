---
layout: default
title: Airport
---

## Airport
Author: Quentin Charatan and Aaron Kans


This example comes from the book "Formal Software Development: From VDM to Java" written by Quentin Charatan and Aaron Kans. This example illustrate how to model the air traffic control of an airport at a very high level of abstraction. The CML model has been made by Jim Woodcock including adding the reactive behaviour. 



| Properties | Values          |
| :------------ | :---------- |
|Language Version:| cml|


### Airport.cml

{% raw %}
~~~
types
  Aircraft = token

channels
  init, success, failure
  givePermission, recordLanding, recordTakeOff : Aircraft
  getLanded, getPermission : set of Aircraft
  numberWaiting : nat

process Airport =
begin
  state
    permission : set of Aircraft := {}
    landed     : set of Aircraft := {}
    inv landed subset permission
  
 functions  
    permitted: Aircraft * set of Aircraft -> bool
    permitted(a,perm) ==
      a in set perm
      
    down: Aircraft * set of Aircraft -> bool
    down(a,land) ==
      a in set land
      
    PreRecordLanding: Aircraft * set of Aircraft * set of Aircraft -> bool
    PreRecordLanding(a,p,l) ==
      permitted(a,p) and not down(a,l)

  operations

    Init : () ==> ()
    Init() ==
      permission := {} ; landed := {}

    GivePermission : Aircraft ==> ()
    GivePermission(a) ==
      permission := permission union {a}
    pre not permitted(a,permission)

    RecordLanding : Aircraft ==> ()
    RecordLanding(a) ==
      landed := landed union {a}
    pre PreRecordLanding(a,landed,permission)

    RecordTakeOff : Aircraft ==> ()
    RecordTakeOff(a) ==
      atomic (
      landed := landed \ {a};
      permission := permission \ {a} )
    pre down(a,landed)

    NumberWaiting : () ==> nat
    NumberWaiting() ==
      return card (permission \ landed)

  actions
    Cycle =
      ( givePermission?a -> 
          ( [not permitted(a,permission)] & success -> GivePermission(a)
            []
            [permitted(a,permission)] & failure -> Skip )
        []
        recordLanding?a ->
          ( [PreRecordLanding(a,permission,landed)] & success -> RecordLanding(a)
            []
            [not PreRecordLanding(a,permission,landed)] & failure -> Skip )
        []
        recordTakeOff?a ->
          ( [down(a,landed)] & success -> RecordTakeOff(a)
            []
            [not down(a,landed)] & failure -> Skip )
        []
        getLanded!landed -> Skip
        []
        getPermission!permission -> Skip
        []
        (dcl w : nat @ w := NumberWaiting(); numberWaiting!w -> Skip )) ; Cycle
  @
    init -> Init(); Cycle
end
~~~
{% endraw %}

