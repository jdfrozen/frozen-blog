# Java线程（下）局部变量是线程安全的

java方法调用：
![](http://cdn.jdfrozen.cn/1576632794727-c02da512-37d1-45d6-ab8f-22697c870e69.png)
局部变量存储：
栈帧和方法是同生共死的，局部变量就是放到了调用栈里
![](http://cdn.jdfrozen.cn/1576632826359-39fca60f-a79a-44c7-9e8b-19620f61bee0.png)
调用栈与线程：
每个线程都有自己独立的调用栈
![](http://cdn.jdfrozen.cn/1576632916440-b78a536c-f08b-49dc-ac8e-8f427201dffe.png)
线程封闭：
仅在单线程内访问数据
