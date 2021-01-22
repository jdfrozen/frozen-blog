# “等待-通知”机制优化循环等待

细粒度锁产生死锁的一种解决方案是循环等待。
但是循环等待太消cpu了，使用线程阻塞的方式就能避免循环等待消耗 CPU 的问题。
医院就医流程：
1、患者先去挂号，然后到就诊门口分诊，等待叫号；
2、当叫到自己的号时，患者就可以找大夫就诊了；
3、就诊过程中，大夫可能会让患者去做检查，同时叫下一位患者；
4、当患者做完检查后，拿检测报告重新分诊，等待叫号；
5、当大夫再次叫到自己的号时，患者再去找大夫就诊
**这个等待队列和互斥锁是一对一的关系，每个互斥锁都有自己独立的等待队列。**
**![](http://cdn.jdfrozen.cn/1576405592340-578bc3cd-d7e4-4f0c-a900-4ecbc714c078.png)
ait()、notify()、notifyAll()（**尽量用它**） 方法操作的等待队列是互斥锁的等待队列；
synchronized 锁定的是 this，那么对应的一定是 this.wait()、this.notify()、this.notifyAll()；
synchronized 锁定的是 target，那么对应的一定是 target.wait()、target.notify()、target.notifyAll() 


如何解决曾经满足条件的问题：
```

  while(条件不满足) {
    wait();
  }
```
等待通知经典代码样例：
```

class Allocator {
  private List<Object> als;
  // 一次性申请所有资源
  synchronized void apply(Object from, Object to){
    // 经典写法
    while(als.contains(from) ||als.contains(to)){
      try{
        wait();
      }catch(Exception e){
      }   
    } 
    als.add(from);
    als.add(to);  
  }
  // 归还资源
  synchronized void free(Object from, Object to){
    als.remove(from);
    als.remove(to);
    notifyAll();
  }
}
```


