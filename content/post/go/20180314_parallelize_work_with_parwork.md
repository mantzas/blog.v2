---
title: Parallelize work using parwork
date: 2018-03-14T19:00:10+02:00
series: Go series
tags:
  - parallel
  - go
  - fork-join
categories:
  - parallel
  - go
  - fork-join
---

In order to process a lot of work we have to parallelize work across all cores, and especially if it's CPU bound.
Go has goroutines, which can be used to parallelize the work, but there is the cost of context switching for a lot of goroutines.
Minimizing this context switching can be achieved by using a fork-join model when processing work.

[Parwork](https://github.com/mantzas/parwork) solves this problem by using goroutines, channels and waitgroups. It creates workers (goroutines) that pull
work of a queue (channel), process the work and report the work back to a queue (channel).
This is done in a abstracted way so the user has to provide implementation for:

- a Work interface
- a WorkGenerator function
- a WorkCollector function

### Work interface

```go
type Work interface {
    Do()
    GetError() error
    Result() interface{}
}
```

The work interface defines a method `Do()` which contains all the processing logic of the work item. The `GetError() error` method can be used to flag the work item as failed and return a error. The `Result() interface{}` defines a method which returns the result of the work. Due to the lack of generics the data return has to be cast from `interface{}` to the actual result type in order to be usable in the WorkCollector.

The following example work implementation shows a work item that calculates a MD5 hash of a string:

```go
// md5Work defines a structure that holds the value to be hashed and the result of the hashing
type md5Work struct {
    data   []byte
    hashed []byte
}

// Do calculates the hash of the given value
func (gw *md5Work) Do() {
    gw.data = md5.New().Sum(gw.hashed)
}

// GetError returns nil since the work does not fail
func (gw *md5Work) GetError() error {
    return nil
}

// Result returns the hashed result
func (gw *md5Work) Result() interface{} {
    return gw.data
}
```

Check out the examples folder of the [Github](https://github.com/mantzas/parwork) repo for a complete example.

### WorkGenerator function

```go
type WorkGenerator func() Work
```

The WorkGenerator function allows the user to provide a implementation that returns on each call a work item to be processed. If the generator returns nil the generation of work has finished.

Check out the examples folder of the [Github](https://github.com/mantzas/parwork) repo for a implementation of the generator.

### WorkCollector function

```go
type WorkCollector func(Work)
```

The WorkCollector function takes as a argument a completed Work item. It can check for a failure by calling the GetError or the Result method of the Work item and handle it appropriately.

Check out the examples folder of the [Github](https://github.com/mantzas/parwork) repo for a implementation of the collector.

### Check out

Head over to the [Github](https://github.com/mantzas/parwork) repo to see the code, with a working example, try it and if you find something, like a bug or a improvement, don't hesitate do open a issue or better yet create a PR.

Thanks and enjoy!
