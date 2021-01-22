# 管程：并发编程的万能钥匙

什么是管程：
管程，指的是管理共享变量以及对共享变量的操作过程，让他们支持并发
java中===synchronized 关键字及 wait()、notify()、notifyAll() ！
管程的三种模型：Hasen 模型、Hoare 模型和 MESA 模型
MESA 模型：
有两大核心问题：
一个是互斥
另一个是同步
管程解决互斥问题的思路很简单，就是将共享变量及其对共享变量的操作统一封装起来：
每个条件变量都对应有一个等待队列
![](http://cdn.jdfrozen.cn/1576407634827-a7071953-d930-4210-ba40-b38becce9419.png)![](https://cdn.nlark.com/yuque/0/2019/png/257847/1576407663841-f2da6b0d-f376-4daa-b5d4-036c74fa3eb4.png#align=left&display=inline&height=316&originHeight=765&originWidth=1142&size=0&status=done&style=none&width=471)
MESA 模型编程范式：

```
while(条件不满足) {
  wait();
}
```
三个模型比较：
Hasen 模型：
要求 notify() 放在代码的最后，这样 T2 通知完 T1 后，T2 就结束了，然后 T1 再执行。
Hoare 模型：
T2 通知完 T1 后，T2 阻塞，T1 马上执行；等 T1 执行完，再唤醒 T2，也能保证同一时刻只有一个线程执行。
MESA 管程：
T2 通知完 T1 后，T2 还是会接着执行，T1 并不立即执行，仅仅是从条件变量的等待队列进到入口等待队列里面。就是当 T1 再次执行的时候，可能曾经满足的条件，现在已经不满足了，所以需要以循环方式检验判断条件（判断条件绑定到添加变量上）
Java管程：
语言级别管程（synchronized，wait()、notify()、notifyAll() ，条件变量能是一个对象），wait()绑定条件变量的等待队列即阻塞队列
![](http://cdn.jdfrozen.cn/1576417214324-dbf634a9-1fb6-44b7-94b5-f02670ed3ced.png)
JDK并发包里的 Lock 和 Condition（可以多个）
```shell
public class BlockedQueue<T>{
  final Lock lock =
    new ReentrantLock();
  // 条件变量：队列不满  
  final Condition notFull =
    lock.newCondition();
  // 条件变量：队列不空  
  final Condition notEmpty =
    lock.newCondition();
  // 入队
  void enq(T x) {
    lock.lock();
    try {
      while (队列已满){
        // 等待队列不满 
        notFull.await();
      }  
      // 省略入队操作...
      //入队后,通知可出队
      notEmpty.signal();
    }finally {
      lock.unlock();
    }
  }
  // 出队
  void deq(){
    lock.lock();
    try {
      while (队列已空){
        // 等待队列不空
        notEmpty.await();
      }
      // 省略出队操作...
      //出队后，通知可入队
      notFull.signal();
    }finally {
      lock.unlock();
    }  
  }
}
```
notify() 何时可以使用：
除非经过深思熟虑，否则尽量使用 notifyAll()
什么时候可以使用 notify()：
1、所有等待线程拥有相同的等待条件。
2、所有等待线程被唤醒后，执行相同的操作。
3、只需要唤醒一个线程
