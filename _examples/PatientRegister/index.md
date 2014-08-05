---
layout: default
title: PatientRegister
---

## PatientRegister
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### PatientRegister.cml

{% raw %}
~~~
types
  Patient = token
  
values
  LIMIT : int = 200

channels
  init
  add, remove : Patient
  success, failure
  get : set of Patient
  howmany : int

process PatientRegister =
begin
  state
    reg : set of Patient
  inv card reg <= LIMIT

  functions
    moreroom: set of Patient -> bool
    moreroom(r) ==
      card r < LIMIT

    present: Patient * set of Patient -> bool
    present(p,r) ==
      p in set r

  operations
    Init : () ==> ()
    Init() ==
      reg := {}

    addPatient : Patient ==> ()
    addPatient(p) ==
      reg := reg union {p}
    pre moreroom(reg) and not present(p,reg)  

    removePatient : Patient ==> ()
    removePatient(p) ==
      reg := reg \ {p}
    pre present(p,reg)

  actions
    Cycle =
      ( [moreroom(reg)] & add?p -> 
          ( if not present(p,reg) then
              success -> addPatient(p)
            else
              failure -> Skip )
        []
        remove?p ->
          ( if present(p,reg) then
              success -> removePatient(p)
            else
              failure -> Skip )
        []
        get!reg -> Skip
        []
        howmany!(card reg) -> Skip ) ; Cycle
  @
    init -> Init(); Cycle
end
~~~
{% endraw %}

