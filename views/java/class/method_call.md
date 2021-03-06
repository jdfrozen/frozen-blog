# 字节码—方法调用

```java
public static void main(String[] args) {
    int a = 1;
    int b = 2;
    int c = calc(a, b);
}
static int calc(int a, int b) {
    return (int) Math.sqrt(Math.pow(a, 2) + Math.pow(b, 2));
}
```
```java
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
       6: invokestatic  #2         // Method calc:(II)I
       9: istore_3
      10: return
static int calc(int, int);
  descriptor: (II)I
  flags: (0x0008) ACC_STATIC
  Code:
    stack=6, locals=2, args_size=2
       0: iload_0
       1: i2d
       2: ldc2_w        #3         // double 2.0d
       5: invokestatic  #5         // Method java/lang/Math.pow:(DD)D
       8: iload_1
       9: i2d
      10: ldc2_w        #3         // double 2.0d
      13: invokestatic  #5         // Method java/lang/Math.pow:(DD)D
      16: dadd
      17: invokestatic  #6         // Method java/lang/Math.sqrt:(D)D
      20: d2i
      21: ireturn
```
#### main方法调用：
**参数： **所有参数以正确的顺序推入操作数堆栈中，从而准备这些参数。
**`invokestatic`（或类似的调用指令，如后所述），弹出这些参数，并为被调用的方法创建一个新框架，其中将参数放置在其局部变量数组中。**
**`invokestatic详解：`**
invokestatic通过查看地址（从6跳到9），该指令占用了3个字节。这是因为，与到目前为止所看到的所有指令不同，该指令invokestatic包括两个额外的字节来构造对要调用的方法的引用（此外到操作码）。javap将引用显示为#2，这是对该calc方法的符号引用，可以从前面描述的常量池中解析该方法。
#### calc方法调用：
iload_0：将第一个整数参数加载到操作数堆栈。
i2d：应用加宽转换将其转换为双精度，所得的double替换操作数堆栈的顶部。
ldc2_w：指令将双精度常量`2.0d`  （从常量池中取出）压入操作数堆栈。
invokestatic  #5：使用到目前为止已准备好的两个操作数值（to的第一个参数`calc` 和constant `2.0d`）调用static方法。当`Math.pow`方法返回时，其结果将被存储在其调用的操作数堆栈上。
下一条指令 `dadd`弹出顶部的两个中间结果，将它们相加，然后将总和推回顶部。最后invokestatic`Math.sqrt`对结果总和进行调用，并使用缩小转换（`d2i`）将结果从double转换为int 。生成的int返回到main方法，该方法将其存储回`c`（`istore_3`）。
#### 图解流程：
![](http://cdn.jdfrozen.cn/1608801991774-95632816-2fa8-4632-93dc-e98a5eb95a41.png)![](http://cdn.jdfrozen.cn/1608801997831-fda0fe1e-2831-4393-80f9-4e34560a77ad.png)
