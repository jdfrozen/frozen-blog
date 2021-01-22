# 面向对象思想写好并发程序

一、封装共享变量
将共享变量作为对象属性封装在内部，对所有公共方法制定并发访问策略。
```

public class Counter {
  private long value;
  synchronized long get(){
    return value;
  }
  synchronized long addOne(){
    return ++value;
  }
}
```

二、 final 关键字来修饰不可变变量

三、制定并发访问策略
**避免共享**：避免共享的技术主要是利于线程本地存储以及为每个任务分配独立的线程。
**不变模式**：这个在 Java 领域应用的很少，但在其他领域却有着广泛的应用，例如 Actor 模式、CSP 模式以及函数式编程的基础都是不变模式。
**管程及其他同步工具**：Java 领域万能的解决方案是管程，但是对于很多特定场景，使用 Java 并发包提供的读写锁、并发容器等同步工具会更好。
