# Android高性能编程

我的 [安卓性能优化专栏](https://blog.csdn.net/csqingchen/category_10025139.html) 
[Android 高性能编程](https://blog.csdn.net/CSqingchen/article/details/43733337)
[Android 性能优化 -- APP启动、UI优化](https://blog.csdn.net/CSqingchen/article/details/106186210)
[Android 性能优化 -- 内存优化、电量优化](https://blog.csdn.net/CSqingchen/article/details/106186286)
[Android 性能优化 -- 网络优化](https://blog.csdn.net/CSqingchen/article/details/106671184)
[Android 性能优化 -- 进程保活(11种方案总结)](https://blog.csdn.net/CSqingchen/article/details/105362755)
[Android 性能优化 -- Protobuf使用及原理](https://blog.csdn.net/CSqingchen/article/details/50669725)


---
## Android高性能编程基本原则

>本章节转自 <https://www.jianshu.com/p/4516598e5d1e>

1. 尽量少的声明全局变量

2. 声明全局静态变量，一定要加final声明

3. 声明非静态的全局变量，最好不要初始化任何值，在使用到的地方，在进行初始化

4. 函数中若干次使用全局变量，应该将全局变量赋值给本地变量，然后直接使用本地变量

5. 能用Int，不要使用浮点数

6. 能用乘法不用除法

7. 尽量避免使用geter和setter方法

8. 在Activity的onCreate函数中，尽量做少的事。

9. 在Activity中声明的静态数组或者静态代码块，重构到单独的一个类里。

10. 布局文件要尽可能的优化，减少布局的解析时间 。 尽量减少布局的嵌套层次

11. Activity启动后开始进行异步线程的加载，最好delay一下。再开启线程

12. 对于存在于集合中的Bean对象，尽可能少的声明变量。能用int 就不要用long. 声明的string等复杂变量，最好不要进行初始化。

13. 使用线程，一定要给它传一个名字，然后需要定义线程的优先级

14. 在使用集合的时候，优先选择SparseArray。

15. 尽量避免使用枚举

16. 工具方法尽量写成是静态方法

17. 线程间同步尽量使用开销小的同步锁

18. 在使用集合类的时候，如果已知数据的规模，在初始化的时候，就设定好默认大小。

19. 私有内部类访问外部类的私有变量，要将变量修改为包继承权限

20. 对于开销大的算法，且不止是执行一次的，要使用缓存策略

21. 避免在绘制或者解析布局的时候，分配对象。例如onDraw方法

22. 不要给布局写无用的参数，例如RelativeLayout，写layout_weight属性

23. 尽量减少布局的嵌套层数。例如包含一个ImageView和TextView的线性布局，可以用CompoundDrawable的TextView来代替

24. 尽量用Android提供的SparseArray来代替HashMap

25. 如果LinearLayout用于嵌套的layout空间计算，它的android:baselineAligned设置为false,可以加速layout计算

26. 用FloatMath代替Math

27. 尽量避免嵌套的使用layout_weight,那样会影响执行效率

28. 如果为rootView设置了背景，那么会先用Theme指定的背景绘制一遍，然后才用指定的背景绘制，这叫做"overdraw",可以通过theme的background为null来避免

29. 不要有无用的任何资源或者文件

---

[Android最佳性能实践(一)——合理管理内存  - CSDN博客 ](https://blog.csdn.net/sinyu890807/article/details/42238627)
- 节制地使用Service
- 当界面不可见时释放内存
- 当内存紧张时释放内存
- 避免在Bitmap上浪费内存
- 使用优化过的数据集合
- 知晓内存的开支情况
- 谨慎使用抽象编程
- 尽量避免使用依赖注入框架
- 使用ProGuard简化代码
- 使用多个进程

[Android最佳性能实践(二)——分析内存的使用情况  - CSDN博客 ](https://blog.csdn.net/sinyu890807/article/details/42238633)


[Android最佳性能实践(三)——高性能编码优化  - CSDN博客 ](https://blog.csdn.net/sinyu890807/article/details/42318689)
- 避免创建不必要的对象
- 静态优于抽象
- 对常量使用static final修饰符
- 使用增强型for循环语法
- 多使用系统封装好的API
- 避免在内部调用Getters/Setters方法

[Android最佳性能实践(四)——布局优化技巧  - CSDN博客 ](https://blog.csdn.net/sinyu890807/article/details/43376527)
- 重用布局文件，使用<include>和<merge>这两个非常有用的标签
- 仅在需要时才加载布局


[Android高性能编程(1)--基础篇 - 不精通则死_CSDN博客 ](https://blog.csdn.net/litton_van/article/details/21702299)


[Android高性能编程(2)--延迟初始化  - CSDN博客 ](https://blog.csdn.net/litton_van/article/details/21872677)




