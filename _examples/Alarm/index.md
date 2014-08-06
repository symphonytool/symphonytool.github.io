---
layout: default
title: Alarm
---

## Alarm
Author: John Fitzgerald and Peter Gorm Larsen


This is the alarm example from the VDM-SL book, John Fitzgerald and Peter Gorm Larsen, Modelling Systems -- Practical Tools and Techniques in Software Development}, Cambridge University Press, 2nd edition 2009. The example is inspired by a subcomponent of a large alarm system developed by IFAD A/S. It is modelling the management of alarms for an industrial plant. The purpose of the model is to clarify the rules governing the duty roster and calling out of experts to deal with alarms. A comparable model of this example also exists in VDM++. This VDM-SL model has been translated to CML by Peter Gorm Larsen.



### Alarm.cml

{% raw %}
~~~
types

  Plant :: schedule : Schedule
           alarms   : set of Alarm
  inv mk_Plant(schedule,alarms) ==
        forall a in set alarms @
	   forall peri in set dom schedule @
	     QualificationOK(schedule(peri),a.quali)
	     
  Schedule = map Period to set of Expert
inv sch ==
   forall exs in set rng sch @
          exs <> {} and
          forall ex1, ex2 in set exs @
                 ex1 <> ex2 => ex1.expertid <> ex2.expertid

  Period = token

  Expert :: expertid : ExpertId
            quali    : set of Qualification
  inv ex == ex.quali <> {}

  ExpertId = token

  Qualification = <Elec> | <Mech> | <Bio> | <Chem>
	   
  Alarm :: alarmtext : seq of char
           quali     : Qualification

functions

  NumberOfExperts: Period * Plant -> nat
  NumberOfExperts(peri,plant) ==
    card plant.schedule(peri)
  pre peri in set dom plant.schedule

  ExpertIsOnDuty: Expert * Plant -> set of Period
  ExpertIsOnDuty(ex,mk_Plant(sch,-)) ==
    {peri| peri in set dom sch @ ex in set sch(peri)}

  ExpertToPage(a:Alarm,peri:Period,plant:Plant) r: Expert
  pre peri in set dom plant.schedule and
      a in set plant.alarms
  post r in set plant.schedule(peri) and
       a.quali in set r.quali

  QualificationOK: set of Expert * Qualification -> bool
  QualificationOK(exs,reqquali) ==
    exists ex in set exs @ reqquali in set ex.quali

process Main =
begin

@ Skip

end
~~~
{% endraw %}

