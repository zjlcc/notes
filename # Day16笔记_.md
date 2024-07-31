# Day16笔记

# 异步

**同步等待异步任务**

使用 `.Wait()` 或 `.Result` 等同步阻塞操作来等待异步任务完成，可能会导致死锁，特别是在 UI 线程或同步上下文中。
public async Task&lt;int&gt; CalculateAsync()
{
await Task.Delay(1000);
return 42;
}

public void CallCalculate()
{
int result = CalculateAsync().Result; // Potential deadlock
}

在这个示例中，`CalculateAsync` 是一个异步方法，而 `CallCalculate` 使用 `.Result` 来同步等待异步操作的结果。如果 `CalculateAsync` 需要同步上下文（如 UI 线程）来继续执行，而 `CallCalculate` 正在同步等待结果，则可能会导致死锁。

**如何避免死锁**

* 使用 `await` 替代 `.Wait()` 和 `.Result`：

使用 `await` 可以避免阻塞线程，确保异步操作能够正确完成。
public async Task&lt;int&gt; CalculateAsync()
{
	await Task.Delay(1000);
	return 42;
}

public async Task CallCalculateAsync()
{
	int result = await CalculateAsync(); // No deadlock
}

在这个示例中，`CallCalculateAsync` 使用 `await` 等待 `CalculateAsync` 完成，而不是使用 `.Result`，避免了死锁问题。


* 使用 `ConfigureAwait(false)`：

在库代码中使用 `ConfigureAwait(false)` 可以避免捕获同步上下文，减少死锁的可能性。请注意，这种方法在 UI 应用程序中不适用，因为你可能需要在原来的上下文中恢复执行。
public async Task&lt;int&gt; CalculateAsync()
{
await Task.Delay(1000).ConfigureAwait(false);
return 42;
}

在这个示例中，`ConfigureAwait(false)` 可以避免在恢复执行时捕获同步上下文，从而减少死锁的风险。

## 总而言之

综合以上几点，我们可以总结两个原则

* 写应用端的程式的时候，你很清楚自己目前使用的SynchronizationContext 是什么，尽量总是全程使用await，不要写.Result，因为可以减少thread 被block，效能会比较好。同时也减少deadlock 的可能。另外.Result 所抛出的例外会是AggregationException，await 抛出的例外会是内层的例外，比较好读。
* 写Library 的时候，你并不清楚Library 的呼叫者是在什么样的synchronization context 下。此时如果在Library里面写await 把Context 捕获住，使用Library 的人如果写了.Result，就会产生dead lock 。因此你必须要在Library 内遇到await 之处，都要写上.ConfigureAwait(false)，避免捕获当前的Context。

