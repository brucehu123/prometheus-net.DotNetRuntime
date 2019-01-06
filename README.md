# prometheus-net.DotNetMetrics
A plugin for the [prometheus-net](https://github.com/prometheus-net/prometheus-net) package, exposing .NET core runtime metrics including:
- Garbage collection frequencies, pauses and timings
- Heap size by generation
- Bytes allocated
- JIT compilations
- Thread pool size, scheduling delays and reasons for growing/ shrinking
- Lock contention

These metrics are essential for understanding the peformance of any non-trivial application. Even if your application is well instrumented, you're only getting half the story- what the runtime is doing completes the picture.

## Installation
**Requires .NET core v2.2+**.

Add the packge using:
```
dotnet add package prometheus-net.DotNetRuntime --version 0.0.6-alpha
```

And then register the collector:
```
DefaultCollectorRegistry.Instance.RegisterOnDemandCollectors(DotNetRuntimeStatsBuilder.Default());
```

You can customize the types of .NET metrics collected via the `Customize` method:
```
var onDemandCollector = DotNetRuntimeStatsBuilder.Customize()
	.WithContentionStats()
	.WithJitStats()
	.WithThreadPoolSchedulingStats()
	.WithThreadPoolStats()
	.WithGcStats()
	.Create();
				
DefaultCollectorRegistry.Instance.RegisterOnDemandCollectors(onDemandCollector);
```

Once the collector is registered, you should see metrics prefixed with `dotnet_` visible in your metric output.

## Performance impact
The harder you work the .NET core runtime, the more events it generates. Event generation and processing costs can stack up, especially around these types of events:
- **JIT stats**: each method compiled by the JIT compiler emits two events. Most JIT compilation is performed at startup and depending on the size of your application, this could impact your startup performance.
- **GC stats**: every 100KB of allocations, an event is emitted. If you are consistently allocating memory at a rate > 1GB/sec, you might like to disable GC stats.
- **.NET thread pool scheduling stats**: For every work item scheduled on the thread pool, two events are emitted. If you are scheduling thousands of items per second on the thread pool, you might like to disable scheduling events.

## Further reading 
- The mechanism for listening to runtime events is outlined in the [.NET core 2.2 release notes](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-core-2-2#core).
- A partial list of core CLR events is available in the [ETW events documentation](https://docs.microsoft.com/en-us/dotnet/framework/performance/clr-etw-events).




