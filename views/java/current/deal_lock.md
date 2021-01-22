# 死锁了，怎么办

为什么会产生死锁：
细粒度锁。使用细粒度锁可以提高并行度，是性能优化的一个重要手段
![](http://cdn.jdfrozen.cn/1576337798571-d86fce35-1196-4d40-a7f8-708ecaa62180.png)
使用细粒度锁是有代价的，这个代价就是可能会导致死锁。
![](http://cdn.jdfrozen.cn/1576337852235-193cb526-e0ff-4209-8c71-59ccf3d8ccd4.png)![](https://cdn.nlark.com/yuque/0/2019/png/257847/1576337873836-2ecb8a51-5f6d-4de1-930f-6557ed554022.png#align=left&display=inline&height=278&originHeight=682&originWidth=1142&size=0&status=done&style=none&width=464)
产生死锁的条件：
1.互斥，共亨资源Ⅹ和Y只能被一个线程占用
2.占有且等待，线程T1已经取得共享资源Ⅹ，在等待共享资源Y的时候，不释放共享资源Ⅹ
3.不可抢占，其他线程不能强行抢占线程T1占有的资源；
4.循环等待，线程T1等待线程T2占有的资源，线程T2等待线程T1占有的资源，就是循环等待
如何避免死锁：
破坏死锁条件，只要我们破坏其中一个，就可以成功避免死锁的发生
1.对于“占用且等待”这个条件，我们可以一次性申请所有的资源，这样就不存在等待
2.对于“不可抢占”这个条件，占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源，这样不可抢占这个条件就破坏掉了
3.对于“循环等待”这个条件，可以靠按序申请资源来预防。所谓按序申请，是指资源是有线性顺序的，申请的时候可以先申请资源序号小的，再申请资源序号大的，这样线性化后自然就不存在循环了。
一次性申请资源：
![](http://cdn.jdfrozen.cn/1576338187427-ec3ea583-b018-4f0e-8b40-314657060b8d.png)

```shell

class Allocator {
  private List<Object> als =
    new ArrayList<>();
  // 一次性申请所有资源
  synchronized boolean apply(
    Object from, Object to){
    if(als.contains(from) ||
         als.contains(to)){
      return false;  
    } else {
      als.add(from);
      als.add(to);  
    }
    return true;
  }
  // 归还资源
  synchronized void free(
    Object from, Object to){
    als.remove(from);
    als.remove(to);
  }
}

class Account {
  // actr应该为单例
  private Allocator actr;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    // 一次性申请转出账户和转入账户，直到成功
    while(!actr.apply(this, target))
      ；
    try{
      // 锁定转出账户
      synchronized(this){              
        // 锁定转入账户
        synchronized(target){           
          if (this.balance > amt){
            this.balance -= amt;
            target.balance += amt;
          }
        }
      }
    } finally {
      actr.free(this, target)
    }
  } 
}
```
破坏不可抢占条件：
java.util.concurrent 这个包下面提供的 Lock 是可以轻松解决这个问题

破坏循环等待条件：
按照序号从小到大的顺序锁定账户
```shell

class Account {
  private int id;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    Account left = this        ①
    Account right = target;    ②
    if (this.id > target.id) { ③
      left = target;           ④
      right = this;            ⑤
    }                          ⑥
    // 锁定序号小的账户
    synchronized(left){
      // 锁定序号大的账户
      synchronized(right){ 
        if (this.balance > amt){
          this.balance -= amt;
          target.balance += amt;
        }
      }
    }
  } 
}
```
总结：
利用现实世界的模型来构思解决方案！
用细粒度锁来锁定多个资源时，要注意死锁的问题
