# Day24笔记

## Rabbit MQ

### 简单队列：

RabbitMQ 是一个消息代理：它接收并转发消息。您可以将其想象成一个邮局：当您将要邮寄的邮件放入邮箱时，您可以确信邮递员最终会将邮件递送给收件人。在这个比喻中，RabbitMQ 是一个邮箱、一个邮局和一个邮递员。

RabbitMQ 与邮局之间的主要区别在于它不处理纸张，而是接受、存储和转发二进制数据块（ 消息）。

RabbitMQ 以及一般的消息传递功能使用了一些术语。

* 生产无非就是发送。发送消息的程序就是生产者：
* 队列是 RabbitMQ 中邮箱的名称。尽管消息通过 RabbitMQ 和您的应用程序流动，但它们只能存储在队列中  。队列仅受主机的内存和磁盘限制，它本质上是一个大型消息缓冲区。
  多个生产者可以发送消息到一个队列，多个 消费者可以尝试从一个队列接收数据。
  这就是我们表示队列的方式：
* 消费与接收含义类似。消费者是一个大部分时间等待接收消息的程序：

### 工作队列：

工作队列（又称 *任务队列* ）背后的主要思想是避免立即执行资源密集型任务并等待其完成。相反，我们将任务安排在稍后完成。我们将任务封装 *为*消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行该作业。当您运行许多工作进程时，任务将在它们之间共享。

这个概念在 Web 应用程序中特别有用，因为在短暂的 HTTP 请求窗口内不可能处理复杂的任务。

### 发布订阅：

工作队列背后的假设是，每个任务只发送给一个工作线程。在本部分中，我们将做一些完全不同的事情——我们将向多个消费者发送一条消息。这种模式称为“发布/订阅”。为了说明该模式，我们将构建一个简单的日志系统。它将由两个程序组成 - 第一个程序将发出日志消息，第二个程序将接收并打印它们。
在我们的日志系统中，每个正在运行的接收器程序副本都将收到消息。这样，我们就可以运行一个接收器并将日志直接发送到磁盘；同时，我们还可以运行另一个接收器并在屏幕上查看日志。本质上，已发布的日志消息将被广播给所有接收者。

### 路由：

* **描述** ：在工作模式中，使用 Direct Exchange 发送任务消息到一个或多个工作队列。每个队列会将消息分发到连接到它的消费者，确保消息的负载均衡和任务分配。
* **路由** ：Direct Exchange 将消息路由到具有匹配路由键的队列。

主题：
在 Topic Exchange 中，消息的路由键可以是一个主题字符串（由点 `.` 分隔）。例如，`"logs.info"` 和 `"logs.error"` 是两个不同的主题。

* **路由键和模式** ：
  * **路由键** ：是发送消息时指定的字符串。
  * **绑定键（Binding Key）** ：是绑定队列到 Topic Exchange 时指定的字符串，通常使用通配符 `*` 和 `#` 进行模式匹配。
* **通配符** ：
  * `*`（星号）：匹配一个词。例如，`"logs.*"` 将匹配 `"logs.info"` 和 `"logs.error"`，但不会匹配 `"logs.info.system"`。
  * `#`（井号）：匹配零个或多个词。例如，`"logs.#"` 将匹配 `"logs.info"`, `"logs.error"`, `"logs.info.system"` 等。



