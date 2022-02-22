# QA

## Java
### [WIP]数据加密
#### [WIP]单向加密，双向加密
#### [WIP]des和3des区别
#### [WIP]des加密过程
### [WIP]说说http协议
### [WIP]说说xmpp协议
### [WIP]垃圾回收器
> A    JVM的垃圾原理是这样的，它把对象分为年轻代（Young）、年老代（Tenured）、持久代（Perm），对不同生命周期的对象使用不同的垃圾回收算法。

- 年轻代(Young)

> 年轻代分为三个区，一个eden区，两个Survivor区。程序中生成的大部分新的对象都在Eden区中，当Eden区满时，还存活的对象将被复制到其中一个Survivor区，当此Survivor区的对象占用空间满了时，此区存活的对象又被复制到另外一个Survivor区，当这个Survivor区也满了的时候，从第一个Survivor区复制过来的并且此时还存活的对象，将被复制到年老代。

- 年老代（Tenured）

> 年老代存放的是上面年轻代复制过来的对象，也就是在年轻代中还存活的对象，并且区满了复制过来的。一般来说，年老代中的对象生命周期都比较长。

- 持久代（Perm）用于存放静态的类和方法，持久代对垃圾回收没有显著的影响。


## Framework
### [WIP]activity和fragment的生命周期有哪些不同

### OnMeasure 方法几个参数对应含义
> （这个题问的最多的所以我把答案贴上O(∩_∩)O~
首先我们要理解的是widthMeasureSpec, heightMeasureSpec这两个参数是从哪里来的？onMeasure()函数由包含这个View的具体的ViewGroup调用，因此值也是从这个ViewGroup中传入的。这里我直接给出答案：子类View的这两个参数，由ViewGroup中的layout_width，layout_height和padding以及View自身的layout_margin共同决定。权值weight也是尤其需要考虑的因素，有它的存在情况可能会稍微复杂点。 了解了这两个参数的来源，还要知道这两个值的作用。我们只取heightMeasureSpec作说明。这个值由高32位和低16位组成，高32位保存的值叫specMode，可以通过如代码中所示的MeasureSpec.getMode()获取；低16位为specSize，同样可以由MeasureSpec.getSize()获取。那么specMode和specSize的作用有是什么呢？要想知道这一点，我们需要知道代码中的最后一行，所有的View的onMeasure()的最后一行都会调用setMeasureDimension()函数的作用——这个函数调用中传进去的值是View最终的视图大小。也就是说onMeasure()中之前所作的所有工作都是为了最后这一句话服务的。
> 我们知道在ViewGroup中，给View分配的空间大小并不是确定的，有可能随着具体的变化而变化，而这个变化的条件就是传到specMode中决定的，specMode一共有三种可能：
> MeasureSpec.EXACTLY：父视图希望子视图的大小应该是specSize中指定的。
> MeasureSpec.AT_MOST：子视图的大小最多是specSize中指定的值，也就是说不建议子视图的大小超过specSize中给定的值。
> MeasureSpec.UNSPECIFIED：我们可以随意指定视图的大小。)


### [WIP]广播怎么不跨进程
> 本地广播

### [WIP]两列Recyclerview 如果是表格布局怎么添加header view


### [WIP]Thread 和intent service

### [WIP]自定义权限

### App 性能优化

#### UI优化
- a.合理选择RelativeLayout、LinearLayout、FrameLayout,RelativeLayout会让子View调用2次onMeasure，而且布局相对复杂时，onMeasure相对比较复杂，效率比较低，LinearLayout在weight>0时也会让子View调用2次onMeasure。LinearLayout weight测量分配原则。
- b.使用标签<include><merge><ViewStub>
- c.减少布局层级，可以通过手机开发者选项>GPU过渡绘制查看，一般层级控制在4层以内，超过5层时需要考虑是否重新排版布局。
- d.自定义View时，重写onDraw()方法，不要在该方法中新建对象，否则容易触发GC，导致性能下降
- e.使用ListView时需要复用contentView，并使用Holder减少findViewById加载View。
- f.去除不必要背景，getWindow().setBackgroundDrawable(null)
- g.使用TextView的leftDrawabel/rightDrawable代替ImageView+TextView布局

#### 内存优化
> 主要为了避免OOM和频繁触发到GC导致性能下降

- a.Bitmap.recycle(),Cursor.close,inputStream.close()
- b.大量加载Bitmap时，根据View大小加载Bitmap，合理选择inSampleSize，RGB_565编码方式；使用LruCache缓存
- c.使用 静态内部类+WeakReference 代替内部类，如Handler、线程、AsyncTask
- d.使用线程池管理线程，避免线程的新建
- e.使用单例持有Context，需要记得释放，或者使用全局上下文
- f.静态集合对象注意释放
- g.属性动画造成内存泄露
- h.使用webView，在Activity.onDestory需要移除和销毁，webView.removeAllViews()和webView.destory()

**备：使用LeakCanary检测内存泄露**

#### 响应速度优化
> Activity如果5秒之内无法响应屏幕触碰事件和键盘输入事件，就会出现ANR，而BroadcastReceiver如果10秒之内还未执行操作也会出现ANR，Serve20秒会出现ANR 为了避免ANR，可以开启子线程执行耗时操作，但是子线程不能更新UI，因此需要Handler消息机制、AsyncTask、IntentService进行线程通信。

**备：出现ANR时，adb pull data/anr/tarces.txt 结合log分析**

#### 其他性能优化
- a.常量使用static final修饰
- b.使用SparseArray代替HashMap
- c.使用线程池管理线程
- d.ArrayList遍历使用常规for循环，LinkedList使用foreach
- e.不要过度使用枚举，枚举占用内存空间比整型大
- f.字符串的拼接优先考虑StringBuilder和StringBuffer
- g.数据库存储是采用批量插入+事务

### [WIP]内存泄漏会不会引起app卡顿

### [WIP]app卡顿检测


### [WIP]view绘制流程

```java
ViewRoot ->
 performTraversal()->
  performMeasure()->
   performLayout()->
    perfromDraw()->
     View/ViewGroup measure()-> 
     View/ViewGroup onMeasure()->
      View/ViewGroup layout()->
       View/ViewGroup onLayout()->
        View/ViewGroup draw()->
         View/ViewGroup onDraw()
```

## ThirdPart
### [WIP]Rxjava 操作符

### [WIP]Rxjava 1 2 的区别

### [WIP]rxjava多个界面进度通知