> 原文地址 <https://juejin.cn/post/7267737437953720359>



## 前言

对于Android的学习，很多⼈可能学了之后，不知道⾃⼰处于哪个阶段，也不到究竟要学到哪个程度，验证⾃⼰学得如何最好的⾯试，就是尝试去⾯试，⽽⾯试⽆⾮就是问你⼀些⾯试题。

而我之前也有整理过一些面试题，但不是很完整。

所小编重新整理了这些 Android ⾯试题，从 Java 基础，并发，虚拟机到Android Framework，开源框架，性能优化，并且附带了详细的答案，⽆论是想⾯试还是想看看⾃⼰学得如何，那么这份⾯试题，都值得你去学习。

希望能帮助到你面试前的复习并且找到一个好的工作，也节省你在网上搜索资料的时间来学习

> 整理不易，点赞+收藏+关注是对博主最大的支持

## 第1-10期 Java核心基础面试题

[面试官：Java中提供了抽象类还有接口，开发中如何去选择呢？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488726%26idx%3D1%26sn%3D9501f5e7ad24191a4894e4c102f3b14c%26chksm%3Dcf111630f8669f2651ef7256e70a864ac6c7bcf7c7911dbb747a9c54c33aa170a07174cf9977%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：重载和重写是什么意思，区别是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488726%26idx%3D2%26sn%3D3acc03e1c97978f674ca7b5b20e932a9%26chksm%3Dcf111630f8669f26b35e1933f959db7e32efec4d37bfcdad5c2940760900079fad8beb3f89ad%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：静态内部类是什么？和非静态内部类的区别是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488726%26idx%3D3%26sn%3De8a93c702a67c799160945c5204f2b72%26chksm%3Dcf111630f8669f264858291487ba36a7ceb49c2b2f954708d4657fcd4f55d9e3f2fc4ff514d3%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：Java中在传参数时是将值进行传递，还是传递引用？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488726%26idx%3D4%26sn%3Df127d7b22cbd59cc8de3e137bdd5b10d%26chksm%3Dcf111630f8669f2609cc4d4a2ad084b67c7d992ea2a2da39afe62ec30cfa62b6bb26b2e56707%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：使用equals和==进行比较的区别](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488742%26idx%3D1%26sn%3D44fd4ca0e6565caf3076d5cc4737d53d%26chksm%3Dcf111600f8669f168237bcb8aa65415b1aad29a74abdb9518c8d820f219cc332c47cec5d545d%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：String s = new String("xxx");创建了几个String对象?](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488742%26idx%3D2%26sn%3D8cb474744271ba84cc82a4c5ea6f9da3%26chksm%3Dcf111600f8669f16d322070c2c80a19631ea60427998a5a886e7d19cbf20eaf9c1551d56ebbb%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：finally中的代码一定会执行吗？try里有return，finally还执行么](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488742%26idx%3D3%26sn%3D99ad15b8054ef4c06d6f387f101be781%26chksm%3Dcf111600f8669f1662a0b4a83572fff62249624ff3b628279c5c47976580d647d3e3d65aaa7a%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：Java异常机制中，异常Exception与错误Error区别](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488742%26idx%3D4%26sn%3D4a55cd4c65f575c34115ecb57f60f166%26chksm%3Dcf111600f8669f163f1ace89634a3c5e111898467d94657f32fb9de3bb7cd11ab08ee44031c0%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：序列Parcelable,Serializable的区别？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488758%26idx%3D1%26sn%3Da088268a42bb52fb85f7e7d1237383da%26chksm%3Dcf111610f8669f065012b5acdb26bb14a389444f37396a02aba149529719769febe13ce12cfa%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[面试官：为什么Intent传递对象为什么需要序列化？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488758%26idx%3D2%26sn%3D0c5d6081a1ecc90d62a701925fd78a3f%26chksm%3Dcf111610f8669f06409703269f36f899bc15c0161c04d8c44f0ad294a83577f3ace25ce73e24%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第11-14期 Java深入泛型与注解面试题

面试题：[泛型是什么，泛型擦除呢？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488758%26idx%3D3%26sn%3Db39618727677195ceec19c307a3c51b6%26chksm%3Dcf111610f8669f06f6bdc754d4127e1fbf39b47d20ab9ef4fe53a2721bed17d9ce5f9a86a8fe%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[List能否转为List](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488758%26idx%3D4%26sn%3D529f9b5fe86d48232453084e846fe6d5%26chksm%3Dcf111610f8669f0626c6a2a1cef9bcb869a98c99c5dee894b2f0e1a872845eccc5aeabd16913%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Java的泛型中super 和 extends 有什么区别？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488789%26idx%3D1%26sn%3D467aa11912e51efe55d371a1dfd5c543%26chksm%3Dcf1117f3f8669ee5f731779ef78199ecb651f117316b9757094a5dc3307cf5ea9f33e58d4e6c%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[注解是什么？有哪些使用场景？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488789%26idx%3D2%26sn%3Db00c286b81ce7a1669c33b9fe89b5d6b%26chksm%3Dcf1117f3f8669ee5393c2b6adeae4efb01724cfbee4bd97e267f0659c98ec74da964d12d1637%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第15-24期 Java并发编程面试题

面试题：[假如只有一个cpu，单核，多线程还有用吗](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488789%26idx%3D3%26sn%3D633b94ceccbcbbba4c49d177d7f36eb6%26chksm%3Dcf1117f3f8669ee5b8a90dac180b9049e62cc898393413585edee2e9b24330b74e527b7e457a%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[sychronied修饰普通方法和静态方法的区别？什么是可见性?](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488789%26idx%3D4%26sn%3D18c25f86b03168753d840c768a6e4ab1%26chksm%3Dcf1117f3f8669ee59608b8b1cdef7d0e0bd7859874623fe8ee842f93a2c9084841f9bb57fca7%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[Synchronized在JDK1.6之后做了哪些优化](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488887%26idx%3D1%26sn%3D3bb0256c861d90554035e4b45e6169f2%26chksm%3Dcf111791f8669e874cf80b985b3a405b20c386659def88697e52a1c1aa7b30f05a5fa6375f71%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[CAS无锁编程的原理](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488887%26idx%3D2%26sn%3Da5050ab83a40bd1f6fccf96910fabddb%26chksm%3Dcf111791f8669e8780baec631bb3aa5b3efbb0f602ec2efdfc0635c957bcae3b09912098f074%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[AQS原理](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488887%26idx%3D3%26sn%3Dc1c65be66e2e8c4d920a8998040eb121%26chksm%3Dcf111791f8669e876760f6a943fd62d990f08754403ea16b692225dff462b9e054d119673141%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[ReentrantLock的实现原理](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488887%26idx%3D4%26sn%3Dcc73431bb047343d5ee27253312fbf34%26chksm%3Dcf111791f8669e8715d37602ba66c8f87990c0a687c74caa2c6d278a815b309ddcfaaa3bd0a7%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[Synchronized的原理以及与ReentrantLock的区别。](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488926%26idx%3D1%26sn%3D793711b03a0ff01cacabee3e91c7f020%26chksm%3Dcf111778f8669e6eba43c8967ecf95ea80ffd0414395da6a97ee411c317225a61411e1b94ef5%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[volatile关键字干了什么？（什么叫指令重排）](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488926%26idx%3D2%26sn%3D8d120590230c37e122be84671db25096%26chksm%3Dcf111778f8669e6e8ce2dee2950b2e5592fdb85a5b229729e400706b6b2696e6df9b44bb62d7%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[volatile 能否保证线程安全？在DCL上的作用是什么？](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488926%26idx%3D3%26sn%3Dc1d459dbd60de0b6704e80d64fa15e2f%26chksm%3Dcf111778f8669e6ecae47326d706929c4ee85c29c6eda5396f88c1b0d4b44b4d1efa8c099d75%26token%3D1305848414%26lang%3Dzh_CN%23rd)

面试题：[volatile和synchronize有什么区别？](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247488926%26idx%3D4%26sn%3D09f6dd23962ba6cb488083c56a5f4b81%26chksm%3Dcf111778f8669e6e081c144873c80d1e93ec9b59541de9f371be765a3108421ffe1afacf2080%26token%3D1305848414%26lang%3Dzh_CN%23rd)

## 第25-34期 Java虚拟机原理面试题

面试题：[描述JVM类加载过程](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489634%26idx%3D1%26sn%3D06a129062461432626aceefa11161c34%26chksm%3Dcf111a84f86693928d845c397d28586593058ca3f0b259627cd656f93d0e8b606af224dc2511%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[请描述new一个对象的流程](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489634%26idx%3D2%26sn%3Df94bf1c07ac6a93007d5e0c9272c6d59%26chksm%3Dcf111a84f8669392614f61271a5223246768d7baadda330751dfad3c47323f5e672bc7cd20f2%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Java对象会不会分配到栈中？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489634%26idx%3D3%26sn%3D8e20b79659ba9a0bbabd781f628fdab9%26chksm%3Dcf111a84f86693922d2b9b0c01dc935db0a409662a680f11a985de317c9c8e832c27cd5a7650%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[GC的流程是怎么样的？介绍下GC回收机制与分代回收策略](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489634%26idx%3D4%26sn%3D4ec00f9c2de5d97e6893695bf28238d8%26chksm%3Dcf111a84f8669392128fc63b504926fe0489523afb16ad544ab12d411503a426cc47d439e332%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Java中对象如何晋升到老年代？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489634%26idx%3D5%26sn%3Dfb4a84b63d5ae9123762b0d098af39a3%26chksm%3Dcf111a84f8669392d9f672d6e6739d8c9753ee807bcaa40a907af0857afa638d47d30a0ebff1%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[判断对象是否被回收，有哪些GC算法，虚拟机使用最多的是什么算法？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489634%26idx%3D6%26sn%3Da3d9fabed7874fb6b0bbb02223aff9c3%26chksm%3Dcf111a84f8669392602dd981a88270463a44ea216965e86e96866adc203a0321fb2b2254a6e7%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Class会不会回收？用不到的Class怎么回收？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489663%26idx%3D1%26sn%3D360c77ce7afb7b1b4a9a2a81e4ad2d80%26chksm%3Dcf111a99f866938fc1cc9bd3b8032b674726053dfe7c3e0b0de6cbf36d7367012b506f0ef86d%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Java中有几种引用关系，它们的区别是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489663%26idx%3D2%26sn%3D7519d1fd803a0a45fe51bd48382c36d2%26chksm%3Dcf111a99f866938f731e6c8e3444a2644586ab405306bf872e789a48f638a927f9006f7bf580%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[描述JVM内存模型](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489663%26idx%3D3%26sn%3Dcc08a4c9d03865ea78616c69d495fa4a%26chksm%3Dcf111a99f866938f2e7232ca44547fb378d1da808421feb588535da364a22689fe4d5592a541%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[StackOverFlow与OOM的区别？分别发生在什么时候，JVM栈中存储的是什么，堆存储的是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489663%26idx%3D4%26sn%3D28a41eeec5c296107fff251552f7cc25%26chksm%3Dcf111a99f866938f44228b81307aba74f8c6a87695955a1b68bcf8487e9ebb3bec2c6a5b149c%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第35-44期 Java反射类加载与动态代理面试题

面试题：[PathClassLoader与DexClassLoader的区别是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489732%26idx%3D1%26sn%3Dfe41164ceb53ce7ff911ebb8821eefab%26chksm%3Dcf111a22f8669334242e1d2c3d7dab8c661f2c889d2ee869d88f6f23a42d0552743d771e0d0b%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[什么是双亲委托机制，为什么需要双亲委托机制？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489732%26idx%3D2%26sn%3D223100bfb3e402d85f53fe43bb910828%26chksm%3Dcf111a22f86693349da991d982ebdcfa911754ee9baaa068ccadaddd46dcc45384e2cab24d90%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Android中加载类的方法有哪些？有什么区别？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489732%26idx%3D3%26sn%3D3a996cba8c22c63d6aa7d5087d4ca0f2%26chksm%3Dcf111a22f86693346d4ee3932d71230dd5d4c11d6c01f292e8f776b256456d19cda7dc51da43%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[ClassNotFound的有可能的原因是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489732%26idx%3D4%26sn%3Da2be8e99ff7481b13e25d011a4913893%26chksm%3Dcf111a22f866933448fe217a1cddb953e9db516dabbca838b3cfdd25541206ca94470b69f93d%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[odex了解吗？解释型和编译型有什么区别？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489820%26idx%3D1%26sn%3D9e5bbbdf68370bfd091bfaa3651b919f%26chksm%3Dcf111bfaf86692ecf6b3b90a4c333ef8941a579cb1f3eb46574606b01ee2fb69e4e658ee7f76%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[说说反射的应用场景，哪些框架？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489820%26idx%3D2%26sn%3D91a602b16bf459d0586716c8e0cc56ac%26chksm%3Dcf111bfaf86692ec5dd3e6bd16b934f06479d2ed7d96607c21366f25e789e1eb4c8c059b9edd%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[反射为什么慢？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489820%26idx%3D3%26sn%3D10a9499e5916f7a703a4c50490efdf71%26chksm%3Dcf111bfaf86692ecc85d54ea37a922a5c0fa326600d3acf80073ef3aa356d6a3f1a6182a5b06%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[动态代理是什么？如何实现？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489820%26idx%3D4%26sn%3Dfe3a51bbd8f193ecda6c6daf4ad00799%26chksm%3Dcf111bfaf86692ec005f2cd1fdb4dccc898b4f362d7050fe48661c1df8090eef6b8fb59970a3%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[动态代理的方法怎么初始化的？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489820%26idx%3D4%26sn%3Dfe3a51bbd8f193ecda6c6daf4ad00799%26chksm%3Dcf111bfaf86692ec005f2cd1fdb4dccc898b4f362d7050fe48661c1df8090eef6b8fb59970a3%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[CGLIB动态代理](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247489820%26idx%3D5%26sn%3D200aef14ffb8b584e4e68b9003e3f472%26chksm%3Dcf111bfaf86692ec91ee39b192b2c1ed3f484486affe121a20a0411b57f51feed11ce5e3bba3%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第45-54期 网络编程面试题

面试题：[请你描述TCP三次握手与四次挥手的过程与意义](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490060%26idx%3D6%26sn%3Db0ae2bfd71249fdc4a3cf8578b502062%26chksm%3Dcf1118eaf86691fc88dfc7055d9efc6ea056f36d11018e2ecb48f7390b86f4bc0f1f9d93f456%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[谈谈你对TCP与UDP的区别是什么的理解](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490060%26idx%3D7%26sn%3D9215d7da98084a05d84bbfe445edbf19%26chksm%3Dcf1118eaf86691fc4c38eabc2ad2fc3ee7842dc13397fec213aa902b74f260885cdad2bf2181%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[谈谈你对TCP 流量控制与拥塞控制的理解](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490082%26idx%3D1%26sn%3D2022d2c8247520b28020a5427fe6c3b2%26chksm%3Dcf1118c4f86691d2f6ff5d455661aa0275732e191a221b28f656f503ef36e6fa3f349525ae79%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[谈谈你对Http与Https的关系理解](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490082%26idx%3D2%26sn%3D6084c4330b40aa32a7366661cb5fbc43%26chksm%3Dcf1118c4f86691d237db7d5480d179319432cca4cc70536958eedcbba9fef57ead425058e250%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[SSL握手的过程都经历过什么](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490082%26idx%3D3%26sn%3De76ec8330022937eae9f4baabafc85c8%26chksm%3Dcf1118c4f86691d2907f8f47aa6543134fd1bdc21c14e61277d6ccc6cff8541a79bf778c9d50%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[谈谈你对Http的post与get请求区别的理解](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490082%26idx%3D4%26sn%3D68f502c30585eab75b6eb16427c7018b%26chksm%3Dcf1118c4f86691d23928439b96ee9a52d27ecd5d677c5dc8ca69bb8cc75a78025a44657f3a8e%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[输入一串URL到浏览器都经历过什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490082%26idx%3D5%26sn%3D66ea6ab17fcece2ad3e96e616a97cde1%26chksm%3Dcf1118c4f86691d204fb09ecc79fd33c84c4375320f837e85e73eec9b75d610a03bc08eb7c11%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[断点续传原理](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490102%26idx%3D1%26sn%3Dbe1a5c9616fd9ab02e72bf2fa64b887c%26chksm%3Dcf1118d0f86691c639bc29603b22eb19fd996e3482d249b511bd8bdaec9a48452a128d8b7798%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[如何保证下载文件的完整性](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490102%26idx%3D2%26sn%3D46e1c2aacdd6867f3595a42ef599c1c5%26chksm%3Dcf1118d0f86691c6c56cb682344637aac890969085f778b90219ca0040571983d4312b54554e%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第55-58期 Kotlin面试题

面试题：[Kotlin内置标准函数let的原理是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490102%26idx%3D3%26sn%3Da2cab87841cb5e5a746e6317c61df363%26chksm%3Dcf1118d0f86691c6dc74df187f61c55753198bf0f96107d16e51ca6f0f482bcdeb8d63d09156%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Kotlin语言的run高阶函数的原理是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490102%26idx%3D4%26sn%3Da8bd3e6bee0f6a62d5f627a893de36f0%26chksm%3Dcf1118d0f86691c68173a033c458cf777338d12e880833c6409da4dcb0a24952c63b0d8d89b7%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Kotlin语言泛型的形变是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490102%26idx%3D5%26sn%3D84baabc5ff6f874cfbb7f387de3614b4%26chksm%3Dcf1118d0f86691c6c8a543c2f9c382dec5c5b6a1f6be3ef0ab96f5beea14e5d1c268dbb68f3c%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Kotlin协程在工作中有用过吗？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490102%26idx%3D6%26sn%3D501c2975d1595ab021b750bd39f409bc%26chksm%3Dcf1118d0f86691c64ea7e86071d4fd02748a7efbad4513a1709aff141ca3bbc3ce45fc6f20de%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第59-68期 Android 高级UI面试题

面试题：[View的绘制原理](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490161%26idx%3D1%26sn%3Df444e20b0adad7d66ba6234782d3b7c0%26chksm%3Dcf111897f8669181c5c5cb5f5831000d53b228e14f3ca9176adf53762f3c5bc6c952a0418f7e%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[View绘制流程与自定义View注意点](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490161%26idx%3D2%26sn%3Db53624e65ca3782566c00090e267a42a%26chksm%3Dcf111897f86691818c6622b7231b7caa12657e5674fe178a5e83f6a524d7ca030ee95843d54c%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[自定义view与viewgroup的区别](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490161%26idx%3D3%26sn%3Df6d7da7f2b169a51197517d3cf473027%26chksm%3Dcf111897f8669181fb5f75cb491aaed4a5ff4b8d0f3ec673cf5f463f3b83ea69de15584664c4%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[View的绘制流程是从Activity的哪个生命周期方法开始执行的](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490161%26idx%3D4%26sn%3D55c9c13f412006e9ac846eecc112c991%26chksm%3Dcf111897f8669181c645c4494b1193a4f2681e682e5f4027e4644dde8a00a285b5a2a8701647%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Activity,Window,View三者的联系和区别](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490161%26idx%3D5%26sn%3D059191caad0e430cae543853629cac94%26chksm%3Dcf111897f866918160bb8e8d709c3fd0da911fdebd796ff18eeacad718052d2549ff178a1634%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[在onResume中是否可以测量宽高](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490161%26idx%3D6%26sn%3D8de2949bd48bfb1cd3fbfc3b9a7490ad%26chksm%3Dcf111897f8669181472fbb52dd01366786449f6caa25307aa07899c942ecc629b33702e64681%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[如何更新UI，为什么子线程不能更新UI？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490311%26idx%3D1%26sn%3Da8bcfb8f1eb0521951b8978a395b7deb%26chksm%3Dcf1119e1f86690f77191044f8d19579abc8bc05e7558fccc48ae2b3b7749892292e8c134421d%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[DecorView, ViewRootImpl,View之间的关系](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490311%26idx%3D2%26sn%3D8e227b53088b8d548cdded8c7d5e1356%26chksm%3Dcf1119e1f86690f70e8fe3c710ff6938dcb3ed44e9618112752f74d6ef3347ad90a59f27b8d6%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[自定义View执行invalidate()方法,为什么有时候不会回调onDraw()](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490311%26idx%3D3%26sn%3D2c15eaac3cf7c1c8a46a26af0bcc1619%26chksm%3Dcf1119e1f86690f768120f33db1d6937cdc7e506dd5cd7e8013b9e581952da9c45e073d83c20%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[invalidate() 和 postInvalicate() 区别](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490311%26idx%3D4%26sn%3Dd12e2cd3e6dd7b0f379f37b9ed4c47d3%26chksm%3Dcf1119e1f86690f738c8e608c132a849ac3033718f18ac9c9c34fcf6744bd7f92e3b46afc96b%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第69-78期 Android Framework面试题

面试题：[Android中多进程通信的方式有哪些？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490311%26idx%3D5%26sn%3Dd7f61f20bb7f21cfb863a9b22f807a6a%26chksm%3Dcf1119e1f86690f77105a774f0cb8e85f18b585f39581abfa896eab1c51442d2b679c9867afd%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[描述下Binder机制原理？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490311%26idx%3D6%26sn%3Db6a2a724fefad4a377594f3324371cf4%26chksm%3Dcf1119e1f86690f781b804c9855d8dfa7b9ffc3fd4166b1e9db5a1c8a2a2e1419c64122f9776%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[为什么 Android 要采用 Binder 作为 IPC 机制？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490333%26idx%3D1%26sn%3D71bf591e9d304655485ce9872a039c97%26chksm%3Dcf1119fbf86690ed7235edc16577e04bc99cfd9dd840b1d201db54a7657df41660d3192fc859%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Binder线程池的工作过程是什么样？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490333%26idx%3D2%26sn%3D50bd430ac11bac5da23cb7209ce8fcb2%26chksm%3Dcf1119fbf86690ed8e7b80ec95d1d59c7d457e7491cb0a287c8ca9192fe7481dbfa717b93389%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[AIDL 的全称是什么？如何工作？能处理哪些类型的数据？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490333%26idx%3D3%26sn%3D206fdeb745182bd7ad017cb069e513db%26chksm%3Dcf1119fbf86690edbda05addb937d8340afc920e5bdc6210e491e04689bf98696a3a07fe7e3e%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Android中Pid&Uid的区别和联系](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490333%26idx%3D4%26sn%3D8d3e4474a26ac37de801eaec76ce7706%26chksm%3Dcf1119fbf86690edede7c0fdd36e29b62702398ff1544421c4cf65b47e95239eb5e3aa0df013%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Handler怎么进行线程通信，原理是什么？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490363%26idx%3D2%26sn%3Dc4d0dab07a36bedf8b3608b66df65b9c%26chksm%3Dcf1119ddf86690cbca7a37d605c9381c85377d4a2f0c707affaaf86118a1bf4448d8bdf4503c%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[ThreadLocal的原理，以及在Looper是如何应用的？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490333%26idx%3D5%26sn%3Db07440228a14eba2793a4bb91f0a042b%26chksm%3Dcf1119fbf86690ed46e03e65448522bee6c7e5f9a045e4989b70262a1c2a535c194f61a39098%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Handler如果没有消息处理是阻塞的还是非阻塞的？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490333%26idx%3D6%26sn%3Dda84710c0f1740c40a8e3e039680571d%26chksm%3Dcf1119fbf86690ed8bb383d8a16c289eff448d7e29d050897355893bfcc39cbc0889d5545732%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[handler.post(Runnable) runnable是如何执行的？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490363%26idx%3D1%26sn%3De133267232068aa5b4a8e15ce8c2bd96%26chksm%3Dcf1119ddf86690cb2d789fb98f1b1f388f4cfe9bd51d84d581b6b4de3c8c506fb846fcb0b973%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第79-88期 Android组件内核面试题

面试题：[Acitvity的生命周期，如何摧毁一个Activity?](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490363%26idx%3D3%26sn%3D97854c3d5e8197b1929b1cff8a629445%26chksm%3Dcf1119ddf86690cb4c82184dc5d0d3e2c1a2b2864c744d36ae3a1feabeb5125e1639403b3215%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Activity的4大启动模式，与开发中需要注意的问题，如onNewIntent() 的调用](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490363%26idx%3D4%26sn%3Df1e1ff1be0ab83851d1a347cd7914290%26chksm%3Dcf1119ddf86690cbda1559c59213939ead34ff852290b5eced46d9a3170c3da295b64c7e766e%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Intent显示跳转与隐式跳转，如何使用？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490363%26idx%3D5%26sn%3D4caec5676d2d89bf71ca6beb498acaea%26chksm%3Dcf1119ddf86690cb90332929903d1a7edc088fb87719f5a8219d83aa55f4eb84a8298fd288fd%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Activity A跳转B，B跳转C，A不能直接跳转到C，A如何传递消息给C？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490363%26idx%3D6%26sn%3Dfcd2175fbbaf495eef64a541ac3efde1%26chksm%3Dcf1119ddf86690cbbb40f2fb02dab2b69bd10ace5885f1022cba6b146bce6fa0a3935cedec82%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Activity如何保存状态的？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490442%26idx%3D1%26sn%3D8ea4007dee12f99f5c811f9060e3366e%26chksm%3Dcf11196cf866907af8aee0856e3fcc6bda925c73101fa1eed76c88d396f9f50fe35d89f4a648%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[请描诉Activity的启动流程，从点击图标开始。](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490442%26idx%3D2%26sn%3D11940716b4d59a6855180aaf636c0e8d%26chksm%3Dcf11196cf866907af0b40c895b4e3d04e5ed4d3e97ef7da5efb49348ed83fa4d7753204edbaf%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Service的生命周期是什么样的？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490442%26idx%3D3%26sn%3D56cafc2c26a3a575ecb287ca1ea3d133%26chksm%3Dcf11196cf866907a276ce9dbaeb28adc385403ff4cb866c9fe6f84888592acb8e594a1d8359d%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[你会在什么情况下使用Service？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490442%26idx%3D4%26sn%3D42d68f8374c08b41b1c0458d10f2cb02%26chksm%3Dcf11196cf866907ac0d4dce2c687131804b8800b16bf1039f06d3aa4c9642ca434b73fd5fd95%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Service和Thread的区别？](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F644767271)

面试题：[IntentService与Service的区别？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490442%26idx%3D5%26sn%3D808c20af082115b743c7e566d2f0f2de%26chksm%3Dcf11196cf866907a997b76264b2beeec480bfe896668ce1c275a3d1c2cc0a06689b9f3ee7c35%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第89-98期 Android性能优化面试题

面试题：[一张图片100x100在内存中的大小？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490442%26idx%3D6%26sn%3D2304588196810ef6375c1abdbebf0587%26chksm%3Dcf11196cf866907af9da9f7e8916cbf9a5440929bbc37d136786015ef1d0a52fe573ae03e749%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[内存优化，内存抖动和内存泄漏。](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490472%26idx%3D1%26sn%3De2149f0533ae7cf902bf82d38fe7a5c0%26chksm%3Dcf11194ef86690588fe77dec39211a0fe867a28c3c621e5f5fbe7721fed3525525c78600e1ad%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[什么时候会发生内存泄漏？举几个例子](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490472%26idx%3D2%26sn%3De0f58b4a0102931d9586099616667be5%26chksm%3Dcf11194ef8669058dcacab4513ca9706c8a2e8de01a522925361faf8843e0debc3867a14f0f5%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Bitmap压缩，质量100%与90%的区别？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490472%26idx%3D3%26sn%3D9d6803f89c7b3854534b74ba89efa20f%26chksm%3Dcf11194ef866905894c71ebee4d322442e57ca70551e8ee77da827c1993b9f1eb77b1f3837bd%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[TraceView的使用，查找CPU占用](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490472%26idx%3D4%26sn%3Da39d2b8c51a2f11a8dcd351e8e3b436d%26chksm%3Dcf11194ef86690585248ec76634a3d8f9bca899f75913935b50c723afd08e3363c0b3fd53a7a%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[内存泄漏查找](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490472%26idx%3D5%26sn%3D37276aad0a3d1ceb7e8e1d807aab4b4c%26chksm%3Dcf11194ef866905858a599ddf667e4143cadf72601fffe93b515921b4f5903b50f605410204a%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[Android四大组件(以及Application)的onCreate/onReceiver方法中Thread.sleep()，会产生几个ANR?](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490472%26idx%3D6%26sn%3D60f104c3257725d54ed6bc4ce93fb476%26chksm%3Dcf11194ef8669058e436778aae8e5c8f127ea27dcf3c792fb8cf1c4102437716a61071514610%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[当前项目中是如何进行性能优化分析的](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490496%26idx%3D2%26sn%3Da4d3ab87736bad9e2fe70593341fc6c0%26chksm%3Dcf111926f866903066e01e9883a992e29a9a469937f612b597cfa2ee8c2a2f416821e32ba965%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[冷启动、热启动的概念](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490496%26idx%3D1%26sn%3Df0bf6f37b78440d12e7ab16eb6121cf0%26chksm%3Dcf111926f866903031e5bfc005972fc5203a5ef2c9cde3b3453abcff88811e440c0b25b51f86%26token%3D1436311520%26lang%3Dzh_CN%23rd)

面试题：[优化View层次过深问题，选择哪个布局比较好？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490496%26idx%3D3%26sn%3Dc86aafd38630fedb18e7b0d475411b17%26chksm%3Dcf111926f866903051e0ec39f8763e3546f48904a369f9d0448ef5bb18fede5beab154a7e7f5%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 第99-106期 开源框架面试题

[Android开源框架面试题：组件化在项目中的意义](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490496%26idx%3D4%26sn%3D86356886ee65f7517c9730b524bf3966%26chksm%3Dcf111926f8669030b4214a894eeae8789c46a91bd9e876d0691a846f01dac9895b9ff0713492%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[Android开源框架面试题：组件化中的ARouter原理](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490496%26idx%3D5%26sn%3D12c298b67a2e81a05a69041bbb7b45fb%26chksm%3Dcf111926f8669030806d16d1602297e8dec40cbf5a2c9e3f83167a202480f22d95d42e21b2e5%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[Android开源框架面试题：谈一下你对APT技术的理解](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490496%26idx%3D6%26sn%3Da3a781888b65b65726101742f98b69d7%26chksm%3Dcf111926f866903092a53b4bca093f534516cf08ff820578724adf4c865b4907fd085fdea363%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[Android开源框架面试题：谈谈Glide框架的缓存机制设计](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490521%26idx%3D1%26sn%3D7ac145753f2cfd0fc6b5ce3e234fdf96%26chksm%3Dcf11193ff8669029c1e35ffea656c27672a45f329e56f4a8d38bf7ecb9bf94f505d4098e32a2%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[Android项目中使用Glide框架出现内存溢出，应该是什么原因？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490521%26idx%3D2%26sn%3Db152e859b365ddf124fb67857f47832c%26chksm%3Dcf11193ff8669029c56f0d06064916a6e161cbccf28910f827c29fa36499751c6acdb3ea4e66%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[Android开源框架面试题：Android如何发起网络请求，你有用过相关框架码？OkHttp框架解决了你什么问题？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490521%26idx%3D3%26sn%3Dcd149c13c2eeca24b71337c28c548f74%26chksm%3Dcf11193ff86690292cf444a34724686ace6567a4e96f636101ce05100cfa952aa18b3fd113c1%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[Android开源框架面试题：RxJava框架线程切换的原理，RxJava1与RxJava2有哪些区别？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490521%26idx%3D4%26sn%3D5b47a34d823a537a5b49de8808b4f4a5%26chksm%3Dcf11193ff866902988408bc6d2e7ef20487cf40dd4214a1e7f5d7632f50c66d025bfdc7fb19c%26token%3D1436311520%26lang%3Dzh_CN%23rd)

[Android开源框架面试题：谈谈LiveData的生命周期是怎么监听的?](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490521%26idx%3D5%26sn%3D6a1fc75afcb22482fdd8a33a69128a9a%26chksm%3Dcf11193ff86690299a0d4b9e2982341f589e902b4dd81ed3847c0bb795508d4ddb345f7f451f%26token%3D1436311520%26lang%3Dzh_CN%23rd)

## 最后

特意整理出了有分类目录的Android面试题，方便大家平时复习和收藏。

- 数据结构与算法面试题
- Java核心基础面试题
- Java深入泛型与注解面试题
- Java并发编程面试题
- Java虚拟机原理面试题
- Java反射类加载与动态代理面试题
- 网络编程面试题
- Kotlin核心面试题
- 高级UI面试题
- Android Framework面试题
- Android组件内核面试题
- 程序性能优化与数据持久化面试题
- 开源框架面试题

**这些Android面试题都整理打包好了→：** [**点击下载**](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg3ODY2MzU2MQ%3D%3D%26mid%3D2247490528%26idx%3D1%26sn%3D2a8e32edfbf228e63ef98747676ccf62%26chksm%3Dcf111906f8669010c0f1ce8c0957e0d5a0daa1e8fa4a9f0b39c10726c3512b6a27e9dc6b8a22%26token%3D1436311520%26lang%3Dzh_CN%23rd)

所有的面试题目都不是一成不变的，面试题目只是给大家一个借鉴作用，最主要的是给自己增加知识的储备，有备无患。


---


