---
title: Various .NET Benchmarks
date: 2017-07-11T07:36:33+03:00
series: Dotnet series
tags:
  - dotnet
  - benchmark
categories:
  - dotnet
  - benchmark
---

A lot of times i was wondering what is the best performing code.
In order to determine that i had to benchmark my code, but benchmarks are hard to write.
Luckily there is a open source project that does this work perfectly good and very easy.

The name of the library is BenchmarkDotNet and the documentation can be found [here](http://benchmarkdotnet.org/).
The only thing you have to do is:

- Create a console application project (.NET/.NET Core)
- Install the nuget BenchmarkDotNet and it's dependencies
- Always run in release mode!

The following code is needed in the Program Main

```csharp
public static void Main(string[] args)
{
    BenchmarkSwitcher.FromAssembly(typeof(Program)
                    .GetTypeInfo().Assembly).Run(args);
}
```

which will scan the assembly for benchmarks and asks you which one to run.

A typical benchmark is just a class with methods that are annotated with the `[Benchmark]` attribute like the following one:

```csharp
[MemoryDiagnoser]
public class DateTimeToStringBenchmark
{
    private static readonly DateTime _dateTime = 
        new DateTime(2016, 12, 31, 23, 59, 59, 999);
    
    [Benchmark(Baseline = true, Description = "DateTime ToString")]
    public string DateTimeToString()
    {
        return _dateTime.ToString();
    }

    [Benchmark(Description = "DateTime ToString with format")]
    public string DateTimeToStringFormat()
    {
        return _dateTime.ToString("g");
    }
}
```

Pretty simple isn't it?

After the run the result is the following:

| Method                        | Mean     | Error     | StdDev    | Scaled | ScaledSD | Gen 0   | Gen 1   | Allocated |
|-------------------------------|----------|-----------|-----------|--------|----------|---------|---------|-----------|
| 'DateTime ToString'           | 848.8 ns | 2.8059 ns | 2.4874 ns | 1.00   | 0.00     | 0.0410  | -       | 132 B     |
| 'DateTime ToString with format' | 790.0 ns | 18.0956 ns | 16.0413 ns | 0.93   | 0.02     | 0.0391  | 0.0104  | 124 B     |

I get the timings of the method and by using the `[MemoryDiagnoser]` attribute i get event the GC Stats.

## Conducted Benchmarks

Given the easiness to create such benchmark i have started a github repository named [DotNetBenchmarks](https://github.com/mantzas/DotNetBenchmarks).

There you can find some of the performance questions that i have about some components like logging frameworks, XML Serialization, String concatenation etc.
Every time i have to check the performance of some component i will add a new benchmark to this repository. If anyone likes to contribute, even better!
Make a PR!

Please note that:

- These benchmark are not conducted in order to make certain libraries look good or bad
- Please create a issue or better make a PR if you think that the benchmark methodology is wrong or the setup is wrong
- Do not take the results as granted and conduct your own benchmarks to see if in the context of your application the results differ

Thanks and enjoy!
