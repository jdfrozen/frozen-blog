# 互斥锁（上）解决原子性问题

原子性问题的源头是线程切换：
“同一时刻只有一个线程执行”这个条件非常重要，我们称之为**互斥**
![](http://cdn.jdfrozen.cn/1576333504411-42bcfe53-c6ba-49fe-87b3-149fa636179d.png)
synchronized：

```shell
class X {
  // 修饰非静态方法
  synchronized void foo() {
    // 临界区
  }
  // 修饰静态方法
  synchronized static void bar() {
    // 临界区
  }
  // 修饰代码块
  Object obj = new Object()；
  void baz() {
    synchronized(obj) {
      // 临界区
    }
  }
}  
```
![](http://cdn.jdfrozen.cn/1576333664872-cbc2f1df-c762-4ac5-82ef-3038f2855b92.png)
锁和受保护资源的关系：
锁和受保护资源之间的关联关系是 1:N 的关系
错误示范：

![](http://cdn.jdfrozen.cn/1576333876305-fa34efbc-92a6-485d-8680-30dd00b4bf77.png)
