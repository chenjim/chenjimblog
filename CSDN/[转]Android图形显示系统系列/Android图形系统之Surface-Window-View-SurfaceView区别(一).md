**原文** [Android图形系统之Surface-Window-View-SurfaceView区别(一)](https://unbroken.blog.csdn.net/article/details/119489616)  

本篇目的：理解Android Surface/Window/View/SurfaceView区别

    
    
    Activity获得一块显存(Surface && FrameBuffer)，然后在上面绘图(OpenGL && GPU)，最后交给设备
    去显示(Display设备)。

###### **1.Surface**

**一个Surface就是一个对象，该对象持有一群像素(pixels)，这些像素是要被组合到一起显示到屏幕上的。你在u上看到的每一个window(如对话框、全屏的activity、状态栏)都有唯一一个自己的surface，window将自己的内容(content)绘制到该surface中。Surface
Flinger根据各个surface在Z轴上的顺序(Z-order)将它们渲染到最终的显示屏上。**

**一个surface通常有两个缓冲区以实现双缓冲绘制：当应用正在一个缓冲区中绘制自己下一个UI状态时，Surface
Flinger可以将另一个缓冲区中的数据合成显示到屏幕上，而不用等待应用绘制完成。**

**2.Window**

**一个window恰如你在计算机中看到的一个window。它拥有唯一一个用以绘制自己的内容的surface。应用通过 Window
Manager创建一个window，Window Manager
为每一个window创建一个surface，并把该surface传递给应用以便应用在上面绘制。应用可以在surface上任意进行绘制。对于Window
Manager来说，surface就是一个不透明的矩形而已。**

**3.View**

**一个view就是一个window中可交互的UI元素。每个window都有唯一一个附着于它的view
hierarchy(这个还是不翻更好理解吧)。该view hierarchy提供了window所有的行为。当一个window需要重绘时(比如一个view
通过invalidate方法使自己失效了)就要进入到window的surface中去完成了。首先，该window的surface会被锁定，锁定的同时会返回一个canvas，该canvas可被用来在surface上绘制内容。该canvas会沿着view
hierarchy遍历传递给每一个view，好让每个view绘制自己的UI部分。当这个过程完成时，surface将会被解锁和提交(posted)，提交的目的是将刚刚绘制好的缓冲区交换到前台，然后让Surface
Flinger利用该缓冲区的数据刷新window的显示。**

**4.SurfaceView**

**一个SurfaceView就是一个被特殊实现的View，它拥有自己专门的一个surface，以便让应用直接在里面绘制内容。该SurfaceView是独立于其所属window的view
hierarchy的，view hierarchy中的view们共享window那一个surface。SurfaceView
的工作原理比你想的要简单——SurfaceView所做的全部就是要求Window Manager创建一个window，并告诉Window
Manager所创建的window的Z轴顺序(Z-order)，这个Z轴顺序可以帮助Window
Manager决定将新建的window置于SurfaceView所属window的前面还是后面。然后，Window
Manager会将新建的window放置到SurfaceView在所属window中的位置。如果新建window在SurfaceView所属window后面，SurfaceView会将它在所属window中占据的部分变透明，以便让后面的window显示出来。**

**5.SurfaceFlinger**  
**SurfaceFlinger的作用是接受来自多个来源的数据缓冲区，对它们进行合成，然后发送到显示设备。  
当别的服务向SurfaceFlinger请求一个绘图Surface时，它会创建一个主要组件是BufferQueue的新的层（这里的层博主感觉指的是一层层的视图的意思），同时把自己设定为消耗方。大多数应用通常在屏幕上有三个层：屏幕顶部的状态栏、底部或侧面的导航栏以及应用的界面。**

**6.BufferQueue**

**BufferQueue类是 Android
中所有图形处理操作的核心。它的作用很简单：将生成图形数据缓冲区的一方（生产方）连接到接受数据以进行显示或进一步处理的一方（消耗方）。几乎所有在系统中移动图形数据缓冲区的内容都依赖于BufferQueue。  
简而言之，BufferQueue是图形生产者与消费者模型中沟通的桥梁。  
它的基本用法很简单：生产方请求一个可用的缓冲区
(dequeueBuffer())，并指定一组特性，包括宽度、高度、像素格式和用法标记。生产方填充缓冲区并将其返回到队列
(queueBuffer())。随后，消耗方获取该缓冲区 (acquireBuffer()) 并使用该缓冲区的内容。当消耗方操作完毕后，将该缓冲区返回到队列
(releaseBuffer())。**

