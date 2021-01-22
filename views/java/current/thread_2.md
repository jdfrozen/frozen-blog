# Java线程（中）创建多少个线程合适

多线程的目的：
就是提升 I/O 的利用率和 CPU 的利用率。
计算推理：
CPU 的利用率和 I/O 设备的利用率都是 50%
一个线程：
![](http://cdn.jdfrozen.cn/1576420339150-0a23183a-02ae-4f7e-89f5-b63531714418.png)
两个线程：
这样 CPU 的利用率和 I/O 设备的利用率就都达到了 100%
![](http://cdn.jdfrozen.cn/1576420391274-28f35b4a-2b1c-4c14-a879-a77b8d38f505.png)
CPU 密集型计算大部分场景下都是纯 CPU 计算：
理论上“线程的数量 =CPU 核数”
线程的数量一般会设置为“CPU 核数 +1”
![](http://cdn.jdfrozen.cn/1576420442208-90426d61-d07b-40e1-8eb3-c8aaef24adfd.png)
 CPU 计算和 I/O 操作的耗时是 1:2：
最佳线程数 =CPU 核数 * [ 1 +（I/O 耗时 / CPU 耗时）]
![](http://cdn.jdfrozen.cn/1576420531568-8768c9fd-8762-4ada-9611-9c5294fe00f9.png)
