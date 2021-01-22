# Java线程（上）生命周期

Java 语言里的线程本质上就是操作系统的线程，它们是一一对应的
通用的线程生命周期：
![](http://cdn.jdfrozen.cn/1576418890601-80a7e1b0-8ee2-4fea-950f-095e1fb502e4.png)
Java 中线程的生命周期：
1、NEW（初始化状态）
2、RUNNABLE（可运行 / 运行状态）
3、BLOCKED（阻塞状态）
4、WAITING（无时限等待）
5、TIMED_WAITING（有时限等待）
6、TERMINATED（终止状态）
BLOCKED、WAITING、TIMED_WAITING **只要 Java 线程处于这三种状态之一，那么这个线程就永远没有 CPU 的使用权**
**    ![](http://cdn.jdfrozen.cn/1576419041449-8e2849ea-08c0-443c-9a5b-8273e2a9194f.png)
1. RUNNABLE 与 BLOCKED 的状态转换
线程等待 synchronized 的隐式锁
2. RUNNABLE 与 WAITING 的状态转换
第一种场景，获得 synchronized 隐式锁的线程，调用无参数的 Object.wait() 方法。
第二种场景，调用无参数的 Thread.join() 方法。
第三种场景，调用 LockSupport.park() 方法。
3. RUNNABLE 与 TIMED_WAITING 的状态转换
调用带超时参数的 Thread.sleep(long millis) 方法；
获得 synchronized 隐式锁的线程，调用带超时参数的 Object.wait(long timeout) 方法；
调用带超时参数的 Thread.join(long millis) 方法；
调用带超时参数的 LockSupport.parkNanos(Object blocker, long deadline) 方法；
调用带超时参数的 LockSupport.parkUntil(long deadline) 方法。
**对比**WAITING**触发条件多了超时参数**
4. 从 NEW 到 RUNNABLE 状态
Java 刚创建出来的 Thread 对象就是 NEW 状态
5. 从 RUNNABLE 到 TERMINATED 状态
线程执行完 run() 方法后，会自动转换到 TERMINATED 状态
调用 interrupt() 方法（stop方法已经标记为废弃）
interrupt()接收通知
一种是异常：
当线程 A 处于 WAITING、TIMED_WAITING 状态时，如果其他线程调用线程 A 的 interrupt() 方法，会使线程 A 返回到 RUNNABLE 状态，同时线程 A 的代码会触发 InterruptedException 异常。
当线程 A 处于 RUNNABLE 状态时，并且阻塞在 java.nio.channels.InterruptibleChannel 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 会触发 java.nio.channels.ClosedByInterruptException 这个异常；而阻塞在 java.nio.channels.Selector 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 的 java.nio.channels.Selector 会立即返回。
主动检测：
通过 isInterrupted() 主动检测
线程栈信息导出：
jstack 命令或者Java VisualVM这个可视化工具将 JVM 所有的线程栈信息导出来
