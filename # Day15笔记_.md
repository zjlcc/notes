# Day15笔记

# C# Task

看看C# 的Task, constructor 开宗明义告诉我们，请定义这个Task 要做什么。

| 1 | **public**Task**(**Action**action**)**;** |
| --- | ------------------------------------------- |

TaskStatus 告诉你目前Task 处于哪个状态，Created, Running, WaitingToRun, RanToCompletion 等等

| 1 | **public**TaskStatus**Status**{**get**;**}** |
| --- | ---------------------------------------------- |

Task.ContinueWith 告诉你，这个Task 完成后，接下来要做什么

| 1 | **public**Task**ContinueWith**(**Action**<**Task**>**continuationAction**)**;** |
| --- | --------------------------------------------------------------------------------- |

我们省略掉许多其他复杂的栏位和多载，但Task 的本质就是这么一回事。

如果我们还想表达Task完成后会携带结果，我们可以用Task 来表达

**constructor**Task<**TResult**>**(**Func**<**Object**,**TResult**>**,**Object**) **// property in Task**public**TResult**Result**{**get**;**} |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |

实际上，C# Task 是个多才多艺的类别，他不仅乘载了非同步的概念，另外还有许多关于执行的方法。例如Run(), Start(), Wait()等等。但我们暂且先不管 **谁去执行这个Task** 、**执行完Task后该做什么**等，我们先专注在，Task能够协助我们在C#中表达非同步的概念。

关于task，我们还需要多知道两个常用的方法与特性，一个是Task 的`Wait()`, 一个是Task 的`.Result`。这两个方法都是用在等待任务完成，Wait 不会有回传值，而Result 会有任务完成的回传结果。但不论是哪种方式，呼叫这两个，都会造成当前的thread 被block 住，直到任务完成，才能继续执行。

