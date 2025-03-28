---
title: New go event sourcing library named incata
date: 2016-02-01T19:59:22+02:00
series: Pattern series
categories:
  - go
  - eventsourcing
  - eventstore
tags:
  - go
  - eventsourcing
  - eventstore
---

Event sourcing is capturing all changes of an application state as a sequence of events.
Since we only store events we only have to add events to a store, in contrast to updates for keeping the application state.
A much simpler model that scales very well. When needing the application state we just aggregate the events into

More on this can be read all over the internet but two excellent links are available below:

* [Martin Fowler Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html)
* [Event Sourcing Basics by Event Store](http://docs.geteventstore.com/introduction/event-sourcing-basics/)

The library has support for the following RDBMS's:

* Microsoft SQL Server
* PostgreSQL

[incata](https://github.com/mantzas/incata) is very easy to setup and to use like illustrated on the github page.
Any ideas or improvements are highly welcome. Enjoy!
