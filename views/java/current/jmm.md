# Java内存模型

如何解决可见性、有序性、原子性问题：
按需禁用缓存以及编译优化


什么是Java内存模型：
volatile、synchronized 和 final 三个关键字
六项 Happens-Before 规则

Happens-Before 规则：
前面一个操作的结果对后续操作是可见的
1. 程序的顺序性规则
程序前面对某个变量的修改一定是对后续操作可见的。
2. volatile 变量规则
对一个 volatile 变量的写操作， Happens-Before 于后续对这个 volatile 变量的读操作。
3. 传递性
A Happens-Before B，且 B Happens-Before C，那么 A Happens-Before C。
4. 管程中锁的规则
一个锁的解锁 Happens-Before 于后续对这个锁的加锁。
5. 线程 start() 规则
主线程 A 启动子线程 B 后，子线程 B 能够看到主线程在启动子线程 B 前的操作。
6. 线程 join() 规则
主线程 A 等待子线程 B 完成（主线程 A 通过调用子线程 B 的 join() 方法实现），当子线程 B 完成后（主线程 A 中 join() 方法返回），主线程能够看到子线程的操作。
