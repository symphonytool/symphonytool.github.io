---
layout: default
title: Tests
---

## Tests
Author: 




| Properties | Values          |
| :------------ | :---------- |
|Language Version:| classic|


### Tests.cml

{% raw %}
~~~
channels a, b, c, d

-- Test1
-- The following process has the following failures:
--   (<>,{}), (<>,{a}), (<>,{b}), (<>,{a,b})
--   (<a>,{}), (<a>,{a}), (<a>,{b}), (<a>,{a,b}) 
--   (<b>,{}), (<b>,{a}), (<b>,{b}), (<b>,{a,b}) 

process Test1 =
begin
@
  (a -> Skip |~| b -> Skip)
  [|{a,b}|]
  (a -> Skip |~| b -> Skip)
end

-- Symphony offers only the trace a.

-- Test5
-- The following process is equivalent to
-- (b -> Skip) |~| ((b -> Skip) [] (c -> Skip))
-- = (b -> Skip) |~| ((b -> Skip) [] (c -> Skip))
-- = ((b -> Skip) [] Stop) |~| ((b -> Skip) [] (c -> Skip))
-- = (b -> Skip) [] (Stop |~| (c -> Skip))

process Test5 =
begin
@
  ((a -> b -> Skip) [] (c -> Skip)) \\ {a}
end

-- Symphony offers just b

-- Test6
-- The following process can avoid deadlock if Test5 chooses to offer c

process P = begin @ c -> Skip end

process Test6 = Test5 [|{c}|] P

-- This process deadlocks


~~~
{% endraw %}

