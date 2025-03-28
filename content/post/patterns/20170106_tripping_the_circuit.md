---
title: Tripping the circuit
date: 2017-01-06T13:11:25+02:00
tags: 
  - dotnet
  - go
series: Pattern series
categories: 
  - dotnet
  - go
---

This is probably one of the most useful "cloud" patterns out there and it is fairly easy to implement.  
There are great articles and implementations, like [Polly](https://github.com/App-vNext/Polly),
already on the internet about this pattern so why another one?

> Κρείττον οψιμαθή είναι ή αμαθή.  
> Socrates 469-399 BC., Philosopher

Which translates to:

> Better too have learned lately than never, as he tried to explain why he learned to play
> guitar in his old age.

I have learned better by reading, implementing and writing about something so stick with me.

## Problem

Almost every application communicate with other services or resources, and they fail...

The reasons:

- slow network connections
- timeouts
- the resources being over-committed or temporarily unavailable
- buggy release
- etc

When this happen our system becomes unstable, unreliable, brittle and failures cascade.

Let's go with an example of a failing remote service, let's say we have the following scenario

- the remote service times out after 60sec
- our service gets 30 req/s
- the usual response time is 1s  
- each request takes up 1MB of RAM

What happens in our application?  

- The current request takes up resources in order to make the call and blocks (or awaits) for 60sec
- All requests for the same service will suffer the same fate
- Almost 1800 requests will be waiting for response at the end of the first 60s
- Almost 1800MB of RAM is used up at the end of the first 60s
- All clients that call our service fail in the same way
- The failure cascades
- The response times go through the roof and will be 60s for each request due to the timeout
- The SLA, we might have, will be breached
- The perfect storm is about to form

The above is a simplified example but is not that far fetched.

## Solution

Using a circuit breaker can improve the stability and resilience of our application.
The circuit is actually a state machine with 3 states

- Closed, meaning the execution will proceed
- Open, meaning the execution will not proceed and throw a exception or return an error (implementation specific)
- Half-Open, meaning that some executions are allowed after some time in order to check the remote service
and open or close the circuit based on the response

For deeper knowledge on the pattern please read the following excellent articles

- [MSDN Circuit Breaker](https://msdn.microsoft.com/en-us/library/dn589784.aspx)
- [Martin Fowler Circuit Breaker](http://martinfowler.com/bliki/CircuitBreaker.html)

## Benefits

By using a circuit breaker we have the following benefits:

- Fail in a controlled manner
- Fail fast
- Save server resources
- Maintain response times (SLA)
- Handle failures differently when the circuit opens eg
  - Redirect to another resources
  - Save for later retry
  - etc

## Implementation

There are two implementation of the circuit breaker.
They share the same philosophy but are written in C# and Go.
Both implementations have a setting provider interface which can be implemented
in order to get the settings from anywhere. There is a in-memory settings implementation which
holds the settings in memory.
Both implementation are key based which means that for every key
the implementation provides a separate circuit which is actually the state.
The following setting exist for each key:

- Failure Threshold at which the circuit opens
- Retry Timeout defines after how much time after the circuit trips will the state be half-open
- Retry Success Threshold defines after how many successful retries will the circuit reset and Closed
- Max Retry Execution Threshold defines how many retries are allowed in the half-open state

The C# implementation can be found @ [clouddotnet](https://github.com/mantzas/clouddotnet).
The implementation is generic, asynchronous and thread safe and lock-free.  
The Go implementation can be found @ [gocloud](https://github.com/mantzas/gocloud).
The implementation is idiomatic and "goroutine" safe.

## Diffs

- Since Go does not have generics the action to be executed returns a interface and a error
so type casting is necessary

## Epilogue

My [blog](http://github.com/mantzas/blog) is hosted in github so for any change, improvement or fix
you can either open a issue or submit a pull request.  
The same goes for both implementations.
