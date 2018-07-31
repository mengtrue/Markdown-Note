突出 Android 系统及 APP 掌握及经验

- 手机研发相关 （相关API：WIFI/BT，系统架构）
- Feature 开发    （系统应用，调试，通信）
- APP 开发          （架构，线程，锁，性能）

重点知识点

> **Android 基本**
>
> APP 集中启动  ***
>
> 界面绘制 *
>
> viewgroup 触摸事件 **
>
> input 事件 ***
>
> 并发处理 ***
>
> AIDL ***
>
> BLE/WIFI ***

> **Andriod 进阶**
>
> ANR ***
>
> OOM *
>
> 性能优化 *
>
> 设计模式、框架模式、系统裁剪 *

> **Java 相关**
>
> 锁 常用锁 重入锁 synchronize ***
>
> 线程 多线程通信 线程池 ***
>
> 堆栈 **
>
> Hashmap *
>
> 虚拟机 内存管理 垃圾回收 *

> **数据结构 算法**
>
> 数组 队列 链表 ***
>
> 二叉树 **
>
> 随机数生成 （不重复）

## Android 基本

### APP 集中启动 

APP 启动类型为 `冷启动` `热启动` 

冷启动为 启动应用时，后台没有该应用的进程，系统会创建一个新的进程分配给该应用，先创建和初始化 `Application` ，再创建和初始化 `MainActivity` ，最后显示在界面上

热启动为 启动应用时，后台已有该应用的进程，则从已有的进程中启动，会跳过 `Application` ，直接运行 `MainActivity` 

> ```markdown
> Application 构造函数
> Application.attachBaseContext()
> Application.onCreate()
> Activity 构造函数
> Activity.setTheme()
> Activity.onCreate()
> Activity.onStart()
> Activity.onResume()
> Activity.onAttachedToWindow()
> Activity.onWindowFocusChanged()
> ```

### 界面绘制

用户界面框架采用 MVC（Model-View-Controller）模型，处理用户输入的控制器，显示界面内容的试图，模型层是应用程序的核心，数据和代码保存在模型中

界面显示为：应用程序把经过 **测量（Measure）、布局（Layout）、绘制（Draw）** 后的 surface 缓存数据，通过 SurfaceFlinger 把数据渲染到显示屏幕上，通过 Android 的刷新机制刷新数据；即应用程序负责绘制，系统层负责渲染，通过进程间通信把需要绘制的数据传递给系统层服务，由系统层的 SurfaceFlinger 服务绘制到屏幕上

**层级越深，元素越多，耗时越长**

==Measure== 深度优先原则递归得到所有视图的宽、高，直到所有子视图大小都测量为止

==Layout== 深度优先原则递归得到所有视图的位置，一个子视图在左上角位置确定后，结合确定的宽和高，就可确定在窗口中的布局

==Draw== CPU 和 GPU（显示与绘制效率较高） 两种方式，

### ViewGroup 触摸事件传递

触摸事件类型：`ACTION_DOWN` `ACTION_MOVE` `ACTION_POINTER_DOWN` `ACTION_POINTER_UP` `ACTION_UP` 

事件传递三个阶段：分发（`Dispatch` ，对应 `dispatchTouchEvent` ），决定直接处理这个事件或将事件继续分发给子视图处理；拦截（`Intercept` ，对应 `onInterceptTouchEvent` ），仅在 viewgroup 及其子类存在，用以判断是否拦截某个事件，如果拦截，则在同一序列事件中，此方法不会被再次调用；处理/消费（`Consume` ，对应 `onTouchEvent` ），返回结果表示是否处理了此事件

拥有事件传递处理能力包括：`Activity` `ViewGroup` `View` 

事件传递，首先类型是 `down->move->up` ，在 `Activity` 中，`onTouch` 优先于 `onClick` 

传递顺序从 `Activity` 到 `ViewGroup` ，再由 `ViewGroup` 递归传递给子 `View` ，`ViewGroup` 通过拦截事件，进行是否继续传递

处理事件，则是在最小单位 view中，是否进行消费，若返回 false 或使用 super，则逐层往上进行消费处理

`ViewGroup` 默认不拦截任何事件

### Input 事件

由 InputManager 进行事件读取、分发，在 APP 中可进行监听与处理

### 并发处理

**使用线程池** ：`new Thread()` 耗费性能，缺乏管理，野线程，无限制创建，相互竞争，过多占用系统资源，不利于扩展（定时、中断），线程池的优点为 可重用存在的线程，减少开销，性能佳，有效控制最大并发线程数，提过系统资源使用率，避免过多资源竞争，避免堵塞，提供定时、定期执行、单线程、并发数控制等功能

线程池任务提交过程：`ThreadPoolExecutor()` 

线程池中实际线程数量小于 核心线程数，启动一个新的线程进行任务的处理

线程池中实际线程数量大于等于 核心线程数，将任务放置到任务队列中进行处理

如果任务队列已经满，无法再存放新的任务，则判断线程池中线程数量是否大于最大线程数，如果小于，则创建新的线程执行，否则将执行决绝策略

### AIDL(Android Interface definition language)

IPC（Inter-Process Communication）一种，服务器端和客户端

服务器端：新建 AIDL 文件，声明该服务需要向客户端提供的接口；在 Service 子类中实现定义的接口方法（实例化相应的 Stub 类）；在 AndroidMainfest.xml 注册服务已经声明为远程服务

客户端：拷贝服务端的 AIDL 文件到目录下；使用 Stub.asInterface 接口获取服务器的 Binder；通过 Intent 指定服务端的服务名称和所在包，绑定远程 Service

### 性能优化

==APP 启动优化== ：

Application 创建过程中尽量少进行耗时操作，UI布局层次不宜过多（影响测量、布局、绘制效率），各组件生命周期方法减少耗时操作

```markdown
# 启动并显示启动时间
adb shell am start -W package_name/.activity.MainActivity
# 强制结束
adb shell am force-stop package_name
```

`TotalTime` 表示新应用启动耗时，包括新进程的启动和 `Activity` 的启动

在代码中，冷启动起始时间点可记录为 `application.attachBaseContext()` ，热启动起始时间点可记录为 `activity.onRestart()`，而 `onResume` 方法制视完成了自身的一些配置，如主题、windows属性等，其后各个 view 也要进行测量、布局、绘制，可在 `Activity.onWindowFocusChanged()` 记录，但要注意此函数在焦点变化时就会触发

需要隐藏显示某些元素，除了 `INVISIBLE` 和 `GONE` 属性外，（两者效率较低，因为界面初始化时均已加载），可使用 `ViewStub` ，不占布局位置，占用资源小，只有调用 `ViewStub.inflate()` 或 `setVisbility()`  指向的布局才会加载和实例化，即所谓的延迟加载

### Android P

应用不能对大部分的广播进行静态注册，如开机广播可以进行注册并能正常接收

## Android 进阶

### ANR

/data/anr 目录下 trace.txt 文件，记录发生问题进程的虚拟机相关信息和线程的堆栈信息

常见的 接收广播操作不能超过10s，以及内存 too many memory，导致无响应或app被杀死

### OOM （Out of Memory）

Android 虚拟机是基于寄存器的 Dalvik，最大堆大小一般是 16M，或24M，通常是由于 堆内存溢出

每个应用使用一个专有的 Dalvik 虚拟机实例运行，由 Zygote 服务进程孵化，每个应用程序都是在属于自己的进程中运行，系统为不同类型的进程分配了不同的内存上限，若运行过程中出现了内存泄漏（不再使用某个对象，但仍然有引用指向，导致 JAVA 垃圾回收器无法回收）造成内存超过上限，被系统 kill

常见原因：

`static` ：是否引用一些资源耗费过多的实例（Context），可使用 `Application Context` 

`context` ：内部类持有外部对象，常见为内部线程；线程内部采用弱引用保存 context 引用，将线程内部类改为静态内部类

`bitmap` ：占用内存大；及时销毁（`recycle()` ），设置一定的采样率（显示区域并不要求完全显示，`inSampleSize` ），使用软引用

`InputStream/OutputStream` ：未关闭

`unregisterReceiver` ：声明周期内注销 receiver

`adapter 缓存 contentView` ：当 listView 非常多时，需要采用一定的策略重用

`数据库 cursor` ：没有关闭

### 设计模式

https://blog.csdn.net/happy_horse/article/details/50908439

常用的有 单例模式（系统级服务）、建造者模式（构建与表示分离 AlertDialog.Builder）、适配器模式（ListView/GridView 的 Adapter）、代理模式（Binder 机制）、观察者模式（数据库注册 DataSetObserver、事件通知）、备忘录模式（onSaveInstanceState/onRestoreInstanceState）、状态模式（View.onVisibilityChange）

### 框架模式

> MVC（Model-View-Controller）
>
> 将 M 和 V 的实现代码分离，使同一个程序可以使用不同的表现形式，C 则确保 M 和 V 的同步
>
> View 视图层，采用 xml 文件进行界面的描述；Controller 控制层，Activity，交割业务逻辑至 Model ；Model 模型层，对数据库、网络以及业务计算等操作放在该层，数据绑定等即采用 MVC

> MVP（Model-View-Presenter）
>
> P 类似 C，但 View 不直接使用 Model，之间通信通过 Presenter，所有交互都发生在 Presenter 内部

> MVVM（Model-View-ViewModel）
>
> 将视图 模型数据 双向绑定，View 和 Model 之间无联系，通过 ViewModel 进行交互，视图的的数据变化会同时修改数据源，而数据源的数据的变化会立即反应到View

### 系统裁剪

体现产品特性，优化产品

浅层次：去除不必要的 apk，深层次：编译系统和框架层

`build/target/product` 关键 mk 文件，base.mk

`framework/base/data` framework层裁剪

`init.rc` 本地服务    `system.prop` 属性内容

## JAVA

### 锁

一种线程同步机制，比 `synchronized` 同步块更复杂  [AQS：Abstract Queue Synchronizer]

> 公平锁：加锁前检查是否有排队等待的线程，先来先得；非公平锁：不考虑排队等待，直接尝试获取锁，获取不到自动到队尾；`synchronized` 非公平锁，`ReentrantLock` 通过 AQS 实现线程调度，可变为公平锁
>
> 可重入锁：同一个线程在外层方法获取锁的时候，进入内层会自动获取锁，可一定程度避免死锁（线程阻塞）
>
> 独享锁：一次只能被一个线程所持有，`synchronized` ；共享锁：可被多个线程所持有，`ReadWriteLock` 中读锁是共享锁，可保证并发读非常高效，写锁是独享锁，通过 AQS 实现
>
> 从看待并发同步的角度，悲观锁认为对同一个数据的并发操作，一定发生修改，即需要枷锁，适合写操作非常多的场景；乐观锁认为不会发生修改，适合读操作非常多的场景，可带来性能提升，通常通过 CAS （Compare and Swap）自旋实现原子操作的更新
>
> 分段锁：不是具体的一种锁，是一种锁的设计，细化锁的粒度，当操作不需要更新整个数组的时候，仅仅针对数组中的一项进行加锁操作
>
> 针对 `synchronized` 锁的状态，通过对象监视器在对象头中的字段表明，偏向锁：一段同步代码一直被一个线程所访问，该线程自动获取锁，降低获取锁的代价；轻量级锁：当锁是偏向锁时，被另一个线程访问，偏向锁升级为轻量级锁，其他线程通过自旋的形式尝试获取锁，不会阻塞，提高性能；重量级锁指当锁为轻量级锁的时候，另一个线程虽是自锁，但自旋不会一直持续下去，到一定次数，还没有获取到锁，变为重量级锁，其他申请的线程进入阻塞，性能降低
>
> 自旋锁：尝试获取锁的线程不会立即阻塞，而是采用循环的方式尝试获取，减少线程上下文切换的消耗，但消耗CPU

### 栈 堆

`stack` “后进先出” （LIFO），只能在一端进行插入（压栈）或删除（出栈）；函数中定义的一些基本类型的变量和对象的引用变量都在函数的栈内存中分配，当超过变量的作用域后，JAVA 自动释放其内存空间

堆 存放由 new 创建的对象和数组，由 JAVA 虚拟机的自动垃圾回收器管里

作用域 为变量所在代码块的语句块的范围，形式上用 `{}` 

### Hashmap

在 `map` 中，通过对象对对象进行索引，用来索引的对象叫做 key，对应的对象叫做 value，Hashmap 通过 hashcode 对其内容进行快速查找，Treemap 中所有元素保持固定的顺序

### 链表

一种动态的数据结构，操作需要通过指针进行，链表内存的分配不是在创建时一次完成，每添加一个节点就分配一次内存，没有闲置内存，空间效率比数组高

## 题目

`taskAffinity` 属性，Activity 希望进入的 task，若没有显式指明，则等于 Application 指明的 taskAffinity，若 Application 也没有指明，则等于包名；Task 的 affinity 属性等于根 Activity 的 taskAffinity 值

> Activity 四种启动模式
>
> 在 AndroidManifest.xml 中 launchMode 属性
>
> `standard` ：默认启动模式，可被多次实例化，每个实例都会处理一个 Intent 对象
>
> `singleTop` ：若实例已经存在于任务栈的栈顶，再次启动时，不会创建新的实例，会重用栈顶的实例，会调用该实例的 onNewIntent 方法将 Intent 对象传递到此实例中；若不在栈顶，则与 `standard` 相同
>
> `singleTask` ：保证 Task 栈中只有一个该 Activity 实例，但有可能位于栈顶或者栈底，但再次运行时，会 destroy 之上的 activity，保证整个栈中，只有一个实例
>
> `singleInstance` ：总在新任务中开启，并且新的任务中只有一个实例，由此实例启动的其他 activity 会自动运行于另一个任务中