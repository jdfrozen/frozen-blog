# 字节码—实例创建

```java
public class Test {
    public static void main(String[] args) {
        Point a = new Point(1, 1);
        Point b = new Point(5, 3);
        int c = a.area(b);
    }
}
class Point {
    int x, y;
    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public int area(Point b) {
        int length = Math.abs(b.y - this.y);
        int width = Math.abs(b.x - this.x);
        return length * width;
    }
}
```
```java
public static void main(java.lang.String[]);
  descriptor: ([Ljava/lang/String;)V
  flags: (0x0009) ACC_PUBLIC, ACC_STATIC
  Code:
    stack=4, locals=4, args_size=1
       0: new           #2       // class test/Point
       3: dup
       4: iconst_1
       5: iconst_1
       6: invokespecial #3       // Method test/Point."&lt;init&gt;":(II)V
       9: astore_1
      10: new           #2       // class test/Point
      13: dup
      14: iconst_5
      15: iconst_3
      16: invokespecial #3       // Method test/Point."&lt;init&gt;":(II)V
      19: astore_2
      20: aload_1
      21: aload_2
      22: invokevirtual #4       // Method test/Point.area:(Ltest/Point;)I
      25: istore_3
      26: return
```
#### 详解
new：创建对象后，保留一个引用，（main栈）结果为： point
dup：赋值引用后，保留两个引用，（main栈）结果为： point,point
iconst_1：压入常量1入（main栈）
iconst_1：压入常量1入（main栈）
invokespecial： 调用了构造方法用一个引用point和1，1，（main栈）结果为： point
astore_1：弹出point，放入本地变量a1
注意：第一个和第四个为（main栈）
![](http://cdn.jdfrozen.cn/1608804444979-878bec72-1733-4652-8e49-31670cd6b0f4.png)
![](http://cdn.jdfrozen.cn/1608804451796-5a2adf25-7f5b-41e2-a284-c50edf92dbfb.png)
b实例创建以此类推
![](http://cdn.jdfrozen.cn/1608804620005-a5bdeeb1-5ec6-49a0-917d-739fff84cb85.png)
![](http://cdn.jdfrozen.cn/1608804626590-36dff4e3-7b49-4602-9b05-b5a301f7da43.png)
#### 实例方法调用：
aload_1：将局部变量表位置1(a)压入操作数栈
aload_2：将局部变量表位置b(b)压入操作数栈
invokevirtual：调用`area`方法`invokevirtual`
istore_3：方法返回放入本地变量c3
