# 基于堆栈的架构

**PC寄存器**：对于Java程序中运行的每个线程，PC寄存器都存储当前指令的地址。
**JVM栈**：为每个线程分配一个[栈](https://en.wikipedia.org/wiki/Stack_(abstract_data_type))，(框架)其中存储了本地变量，方法参数和返回值。这是显示3个线程的栈的插图。
![](http://cdn.jdfrozen.cn/1608795278328-4764e185-05a9-43ec-83cf-3b76c38b2463.png)                      

每个_框架_在调用方法时被压入栈，并在方法完成时（通过正常返回或引发异常）从栈中弹出。每个框架还包括：
**_局部变量_****数组（Local variables）**，索引从0到其长度减去1。长度由编译器计算。局部变量可以保存任何类型的值，除了`long`和`double`值，它们占用两个局部变量。
**一个操作数堆栈(Operand stack)**，用于存储用作指令的操作数或将参数推入方法调用的中间值。
![](http://cdn.jdfrozen.cn/1608795627121-8e96fe0a-79b0-4ace-b323-32fbdd0f6247.png)
**堆：**由所有线程共享并存储对象（类实例和数组）的内存。对象的重新分配由垃圾收集器管理。
                        ![](http://cdn.jdfrozen.cn/1608795313134-364d02f7-d94f-4f5c-81e8-3410a6f9c70c.png)
**方法区域**：对于每个加载的类，它存储方法的代码以及符号表（例如，对字段或方法的引用）和称为常量池的常量。
                            ![](http://cdn.jdfrozen.cn/1608795333331-61b6d155-486a-4073-8842-cac8de479121.png)