# 字节码-方法变量

#### 代码：
```c
public static void main(String[] args) {
    int a = 1;
    int b = 2;
    int c = a + b;
}
```
javap -v Test.class
方法签名main，一个描述符，该描述符指示该方法采用Strings数组（[Ljava/lang/String;），并具有void返回类型（V）。随后的一组标记将方法描述为public（ACC_PUBLIC）和static（ACC_STATIC）
最重要的部分是`Code`属性，该属性包含该方法的说明以及诸如操作数堆栈的最大深度（在这种情况下为2）以及为该方法在框架中分配的局部变量数之类的信息。这个案例）。以上指令中引用了所有局部变量，但第一个变量（索引为0）除外，该`args`变量保存对参数的引用。其他3个局部变量与变量相对应`a`，`b`并`c`在源代码中。
```c
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: (0x0009) ACC_PUBLIC, ACC_STATIC
Code:
stack=2, locals=4, args_size=1
0: iconst_1
1: istore_1
2: iconst_2
3: istore_2
4: iload_1
5: iload_2
6: iadd
7: istore_3
8: return
...
```
地址0到8的指令将执行以下操作：
`iconst_1`：将整数常量1压入操作数堆栈。
![](http://cdn.jdfrozen.cn/1608797938689-3753d7c8-a5a5-40e9-b07c-2733f8db3376.png)
`istore_1`：弹出顶部操作数（一个int值），并将其存储在与变量对应的索引1的局部变量中`a`。
![](http://cdn.jdfrozen.cn/1608797938670-a8705132-7680-402e-bc0e-4fc5553fae27.png)
`iconst_2`：将整数常量2压入操作数堆栈。
![](http://cdn.jdfrozen.cn/1608797938658-04f74085-0308-449f-8958-579f3d5521ac.png)
`istore_2`：弹出顶部操作数int值，并将其存储在索引为2的局部变量中，该变量对应于variable `b`。
![](http://cdn.jdfrozen.cn/1608797938665-0fe2f6ac-9844-42a0-86cb-cca3bc892c6f.png)
`iload_1`：从局部变量的索引1处加载int值，并将其压入操作数堆栈。
![](http://cdn.jdfrozen.cn/1608797938659-818ef826-8bf7-4bd7-9717-58deac21728d.png)
`iload_2`：从局部变量的索引1处加载int值，并将其压入操作数堆栈。
![](http://cdn.jdfrozen.cn/1608797938577-867c6080-16d8-4661-b0cf-68dbde2e5469.png)
`iadd`：从操作数堆栈中弹出前两个int值，将它们相加，然后将结果推回到操作数堆栈中。
![](http://cdn.jdfrozen.cn/1608797938627-c9d284f6-d4ed-453f-8eec-62913f37458c.png)
`istore_3`：弹出顶部操作数int值，并将其存储在索引为3的局部变量中，该变量对应于variable `c`。
![](http://cdn.jdfrozen.cn/1608797938644-553a5e41-4e50-4986-82a1-5098c1630f78.png)
`return`：从void方法返回。
上面的每条指令仅由一个操作码组成，该操作码精确指示了JVM要执行的操作。
