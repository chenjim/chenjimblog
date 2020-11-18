
[TOC]

>本文收发地址 <https://blog.csdn.net/CSqingchen/article/details/135324313>
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>


## 前言

最近编码时看到 Intellij 建议使用 String 替换 StringBuilder ，不是应该推荐 StringBuilder 嘛？  
![](https://pic.chenjim.com/202312261108341.png-blog)  
在 jetbrains 上也有对此问题的讨论，链接如下  
<https://intellij-support.jetbrains.com/hc/en-us/community/posts/115000093250>  

---

## String 和 StringBuilder 性能对比
准备一段测试代码如下 
```java
public static void main(String[] args) {
    String input = "ABCDEF";
    String append = ",AP,";

    Instant start = Instant.now();
    String out = "oo";
    System.out.println(start);

    start = Instant.now();
    for (int i = 0; i < 10000; i++) {
        out = input + append;
    }
    System.out.println(out + Duration.between(start, Instant.now()).toNanos());

    start = Instant.now();
    for (int i = 0; i < 10000; i++) {
        StringBuilder sb = new StringBuilder();
        out = sb.append(input).append(append).toString();
    }
    System.out.println(out + Duration.between(start, Instant.now()).toNanos());
}
```
提供运行，结果如下  
```
ABCDEF,AP,2999000
ABCDEF,AP,1999600
```
发现效率并没有相差太大

## String 和 StringBuilder 使用的字节码对比
我们使用 `javap -c abc.class` 看一下如上代码执行的字节码
```
    30: sipush        10000
    33: if_icmpge     51
    36: aload_1
    37: aload_2
    38: invokedynamic #31,  0             // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
    43: astore        4
    45: iinc          5, 1
    48: goto          28
    51: getstatic     #19                 // Field java/lang/System.out:Ljava/io/PrintStream;
    ....
    83: sipush        10000
    86: if_icmpge     119
    89: new           #51                 // class java/lang/StringBuilder
    92: dup
    93: invokespecial #53                 // Method java/lang/StringBuilder."<init>":()V
    96: astore        6
    98: aload         6
    100: aload_1
    101: invokevirtual #54                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;     
    104: aload_2
    105: invokevirtual #54                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;     
    108: invokevirtual #58                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
    111: astore        4
    113: iinc          5, 1
    116: goto          81
    119: getstatic     #19                 // Field java/lang/System.out:Ljava/io/PrintStream;
```
通过对比可以看到，使用 StringBuilder 执行更多指令  
String 只需要 makeConcatWithConstants 拼接即可  
StringBuilder 需要多构建一个对象，并最终 toSring()，如果忽略这两个指令，效率理论是差不多的。

## 总结 
通过上面分析，两者拼接字符并没有太大性能差异，  
使用 `+` 有更好的可读性，所以 Intellij 建议也比较合理  
如果是涉及字符的大量操作，比如如下场景，还是建议使用 StringBuilder ，可以减少许多新对象的生成。   
```java
    start = Instant.now();
    for (int i = 0; i < 10000; i++) {
        out = out + i;
    }
    System.out.println(out.length() + "," + Duration.between(start, Instant.now()).toNanos());

    start = Instant.now();
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 10000; i++) {
        sb.append(input).append(append);
    }
    out = sb.toString();
    System.out.println(out.length() + "," + Duration.between(start, Instant.now()).toNanos());
```
![](https://pic.chenjim.com/202312281738867.png-blog)  
不建议在循环中使用 `+` 拼接字符  

---

本文到此结束，如果有相关问题，欢迎留言讨论。    
如果你觉得本文还不错，可以点赞+收藏。

