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