# 绿色线程

绿色线程解决了编程中的常见问题。您不希望你的代码会堵塞 CPU，阻止 CPU 去执行有意义的工作。而这个问题，我们是通过使用多任务来解决的，它允许我们在恢复一段代码时暂停执行另一段代码，还会在“上下文”之间切换。

请不要与并行性混淆，虽然容易混淆，但它们是两个不同的东西。用下面的思考方式想一想，绿色线程会让我们更聪明，更高效地解决\(问题\)，从而更有效地利用我们的资源，而并行性就像在问题上投入更多资源。

（绿色线程）通常有两种方法：

* 抢先式多任务处理
* 非抢占式多任务处理（或协作式多任务处理）

### 抢先式多任务处理

> 译：它的总控制权在操作系统手中，操作系统会轮流询问每一个任务是否需要使用 CPU ，需要使用的话就让它用，不过在一定时间后，操作系统会剥夺当前任务的 CPU 使用权，把它排在询问队列的最后，再去询问下一个任务……。—— 百度百科

一些外部调度程序在切换回来之前，停止任务并运行另一个任务。任务没有什么'发言权'，决定是由“别的东西”给出的（通常是某种调度程序）。在操作系统中内核会使用这种处理。如：允许您在运行 CPU 时，使用 UI 在单线程系统上进行计算。我们现在不会谈论这种线程，但我想啊，当你理解\(下\)一种规范例子时，你就能很好地掌握两种（处理）。

### 非抢占式多任务处理

> 译：一个任务得到了 CPU 时间，除非它自己放弃使用 CPU ，否则将完全霸占 CPU ，所以任务之间需要协作——使用一段时间的 CPU ，放弃使用，其它的任务也如此，才能保证系统的正常运行。—— 百度百科

这就是我们今天要谈的内容。一个任务能自己决定处理，处理条件是若 CPU 会更好完成其他事情，而不是等待当前任务中发生的事情。通常这是通过将控制权`yielding`\(让给\)调度程序。一个正常的用例是在发生堵塞执行的事情时，让出控制权。能具体说明性的例子就是 IO 操作。当控制让出时，中央调度程序指示 CPU 继续处理另一个准备实际执行的任务，而不是堵塞的。

> 译：设想下，如果有一个任务死锁，则（非抢占式多任务处理）系统也同样死锁，而（抢先式多任务处理）系统仍能正常运行。—— 百度百科

