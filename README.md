[![NuGet](https://img.shields.io/nuget/v/Microsoft.FASTER.Core.svg)](https://www.nuget.org/packages/Microsoft.FASTER.Core/)
[![Build Status](https://dev.azure.com/ms/FASTER/_apis/build/status/Microsoft.FASTER)](https://dev.azure.com/ms/FASTER/_build/latest?definitionId=8)
[![Gitter](https://badges.gitter.im/Microsoft/FASTER.svg)](https://gitter.im/Microsoft/FASTER?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

# Introduction

Managing large application state easily, resiliently, and with high performance is one of the hardest
problems in the cloud today. The FASTER project offers two artifacts to help tackle this problem.

* **FASTER Log** is a high-performance concurrent persistent recoverable log, iterator, and random 
reader library in C#. It supports very frequent commit operations at low latency, and can quickly saturate 
disk bandwidth. It supports both sync and async interfaces, handles disk errors, and supports checksums. Learn 
more about the FASTER Log in C# here: [github](docs/cs/FasterLog.md) | [web](https://microsoft.github.io/FASTER/cs/fasterlog).

* **FASTER KV** is a concurrent key-value store + cache (available in C# and C++) that is designed for point 
lookups and heavy updates. FASTER supports data larger than memory, by leveraging fast external 
storage (local or cloud). It also supports consistent recovery using a new checkpointing technique that lets 
applications trade-off performance for commit latency. Learn 
more about the FASTER KV in C# here: [github](docs/cs/FasterKV.md) | [web](https://microsoft.github.io/FASTER/cs/fasterkv).
For FASTER C++, check here: [github](docs/cc/) | [web](https://microsoft.github.io/FASTER/cc).


Some key differentiating features of FASTER KV and FASTER Log include:

1. Latch-free cache-optimized index, in FASTER KV.
2. A fast persistent recoverable append-only log based on fine-grained epoch protection for concurrency, 
in FASTER Log.
3. Unique “hybrid record log” design in FASTER KV, that combines the above log with in-place updates, to 
shape the memory working set and retain performance.
4. Architecture as a component that can be embedded in multi-threaded cloud apps. 
5. Asynchronous recovery model based on group commit (called [CPR](#Recovery-in-FASTER)).
6. A rich extensible storage device abstraction called `IDevice`, with implementations for local
storage, cloud storage, tiered storage, and sharded storage.

For standard benchmarks where the working set fits in main memory, we found FASTER KV to achieve
significantly higher throughput than current systems, and match or exceed the performance of pure 
in-memory data structures while offering more functionality. See 
[the SIGMOD paper](https://www.microsoft.com/en-us/research/uploads/prod/2018/03/faster-sigmod18.pdf)
for more details. We also have a detailed analysis of C# FASTER KV performance in a wiki page 
[here](https://github.com/Microsoft/FASTER/wiki/Performance-of-FASTER-in-C%23). The performance of the 
C# and C++ versions of FASTER are very similar. FASTER Log is also extremely fast, capable of saturating modern
NVMe SSDs using less than a core of CPU, and scaling well in a multi-threaded setting.

:new: We now support variable-length keys and values in FasterKV C# via `Memory<byte>` and more generally `Memory<T> where T : unmanaged` as key/value/input types. We also added
a new type called `SpanByte` to represent variable-length keys and values. See the sample [here](https://github.com/Microsoft/FASTER/tree/master/cs/samples/StoreVarLenTypes) for details on these capabilities. This is in addition to the existing object-log support for class types.

:new: We support C# async in FASTER KV (and FASTER Log). See the detailed guide at [this link](docs/cs/README.md) for more information. Also, check out the 
samples [here](https://github.com/Microsoft/FASTER/tree/master/cs/samples).

# Getting Started

* Docs: [github](docs/) | [web](https://microsoft.github.io/FASTER)
* Research papers: [https://aka.ms/FASTER](https://aka.ms/FASTER)
* FASTER C# details: [github](docs/cs/) | [web](https://microsoft.github.io/FASTER/cs)
* FASTER C++ details: [github](docs/cc/) | [web](https://microsoft.github.io/FASTER/cc)
* Tuning FASTER: [github](docs/tuning/) | [web](https://microsoft.github.io/FASTER/tuning)
* Project roadmap: [github](docs/Roadmap.md) | [web](https://microsoft.github.io/FASTER/roadmap).
* NuGet binary feed:
  * [Microsoft.FASTER.Core](https://www.nuget.org/packages/Microsoft.FASTER.Core/)
  * [Microsoft.FASTER.Devices.AzureStorage](https://www.nuget.org/packages/Microsoft.FASTER.Devices.AzureStorage/)
* Samples: [github](https://github.com/Microsoft/FASTER/tree/master/cs/samples)

# Build and Test

For C#, click [here](cs/).

For C++, click [here](cc/).

# Recovery in FASTER KV

Both the C# and C++ version of FASTER KV support asynchronous checkpointing and recovery, based on a new
recovery model called Concurrent Prefix Recovery (CPR for short). You can read more about CPR in our research
paper [here](https://www.microsoft.com/en-us/research/uploads/prod/2019/01/cpr-sigmod19.pdf) (to appear in 
SIGMOD 2019). Briefly, CPR is based on (periodic) group commit. However, instead of using an expensive 
write-ahead log (WAL) which can kill FASTER's high performance, CPR: (1) provides a semantic description of committed
operations, of the form “all operations until offset Ti in session i”; and (2) uses asynchronous 
incremental checkpointing instead of a WAL to implement group commit in a scalable bottleneck-free manner.

CPR is available in the C# and C++ versions of FASTER. More documentation on recovery in the C# version is
[here](docs/cs/FasterKV.md#checkpointing-and-recovery). For C++, we only
have examples in code right now. The sum-store, located [here](cc/playground/sum_store-dir), is a good example of checkpointing and recovery.

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
