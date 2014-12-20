---
layout: post
title:  "Test Data Generation with Prolog"
date:   2014-12-20 12:00:00
categories: testing
---

Creating test data is time consuming for QA.  One of the
industry standards is to have a room full of people think really hard
about the potential use cases and put these cases in an excel sheet.
Buisness models can be very complex so it is easy to make a mistake.
Even if you don't make any mistakes it's unlikely you have the time
or money to fully test the application.

Most often this type of testing is applied to integration testing.
In integration testing QA is testing whether or not some collection
of webapps, services, databases, and enterprise service buses are playing nice with
each other.  For the sake of generating test data the problem can
be reduced to constraints placed on a set of data.

The following example is in [SWI-Prolog](http://www.swi-prolog.org/download/devel) 7.1.
Version 7.1 adds sane double quoted [strings](http://www.swi-prolog.org/pldoc/man?section=strings)
and [dicts](http://www.swi-prolog.org/pldoc/man?section=dicts) with named arguments.

The example will produce data to test a coffee vender.

```prolog
% coffee.pl

:- use_module(library(http/json)).
:- use_module(library(http/json_convert)).

:- json_object
       coffee_order{size:size,flavor:flavor}.

size(small).
size(medium).
size(large).

flavor(mocha).
flavor(vanilla).

dump_coffee_orders :-
    findall(coffee_order{size:X,flavor:Y},(flavor(Y),size(X)),Z),
    json_write(current_output,Z).

run :-
    dump_coffee_orders,
    halt.
```
Execute the following to run.

```swipl -f coffee.pl -t run```

Which will produce:

```json
[
  {"flavor":"mocha", "size":"small"},
  {"flavor":"mocha", "size":"medium"},
  {"flavor":"mocha", "size":"large"},
  {"flavor":"vanilla", "size":"small"},
  {"flavor":"vanilla", "size":"medium"},
  {"flavor":"vanilla", "size":"large"}
]
```

This example uses unification to create orders for us.
The real power comes when you start to model nested
buisness rules.

Limitations
-----------
It may not be cost effective to model the entire application.  In these
cases a simplified model can take care of the bulk of the cases.

While Prolog is a great language for solving constraint satisfation problems
not many programmers are familiar with it.  In particular unification and
recursion are foreign concepts to programmers who program in imperative
languages.
