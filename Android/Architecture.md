## Android Architecture

参考：https://source.android.com/devices/architecture

https://blog.csdn.net/qidi_huang/article/details/76653677

当前最新版为 Android P 预览版，与 Google 合作的厂商可能已拿到完整代码，但能查阅到的完整代码库为 Android O，以及 Android P 的变更点，即当前学习总结均以 Android O 为准

![architecture](../png/Android/architecture.png)

Binder 进程间通信（IPC）机制允许应用框架跨越进程边界并调用 Android 系统服务代码，从而使得高级框架 API 能与 Android 系统服务进行交互

------

### 硬件抽象层（HAL）

HAL 可定义一个标准接口以供硬件供应商实现，可让 Android 忽略较低级别的驱动程序实现。借助 HAL，可顺利实现相关功能，而不会影响或更改更高级别的系统；HAL 实现会被封装成模块，并由 Android 系统适时加载

![HAL_组件](..\png\Android\HAL.png)

必须为特定硬件实现相应的 HAL，通常内置在共享库模块（.so 文件）

为保证 HAL 具有可预测的结构，每个特定于硬件的 HAL 接口都要具有 `hardware/libhardware/include/hardware/hardware.h` 中定义的属性，可让 Android 系统以一致的方式加载 HAL 模块的正确版本

#### HAL 模块

被封装且存储为共享库（.so）的 HAL 实现，`hw_module_t` 结构体中定义模块的版本、名称和作者等元数据，Android 会根据这些元数据找到并正确加载 HAL 模块

还包括指向另一个结构体 `hw_module_methods_t` 的指针，此结构体包含一个指向相应模块的 `open` 函数指针；此函数用于与相关硬件建立通信；

每个特定于硬件的 HAL 通常都会使用附加信息为该特定硬件扩展通用的 `hw_module_t` 结构体

实现 HAL 并创建模块结构体时，必须将其命名为 `HAL_MODULE_INFO_SYM` 

#### HAL 设备

设备是产品硬件的抽象表示；如一个音频模块可能包含主音频设备、USB 音频设备或蓝牙 A2DP 音频设备

设备由 `hw_device_t` 结构体表示，定义类似 HAL 模块

#### HAL 模块编译

内置在模块（.so）文件中，并由 Android 适时动态链接；共享库必须以特定格式命名，以方便找到并正确加载，通用模式：==<module_type>.<device_name>== 

------

### HAL 类型

Android O 对底层进行了重构，运行 Android O 的设备必须支持 **绑定式** 或 **直通式** HAL

- 绑定式 HAL：以 HAL 接口定义语言（HIDL）表示的 HAL，取代早期的传统 HAL 和旧版 HAL；Android 框架和 HAL 之间通过 Binder 进程间通信（IPC）调用进行通信
- 直通式 HAL：以 HIDL 封装的传统 HAL 或旧版 HAL，升级到 Android O 的设备使用

**Android 提供的以下 HIDL 接口将一律在绑定模式下使用：android.framework.\*、android.system.\* 和 android.hidl.\***  

------

### Treble

旨在让制造商以更低的成本、更轻松、更快速将设备更新到新版 Android 系统

利用新的供应商接口，将供应商实现（由芯片制造商编写的设备专属底层软件）与 Android 操作系统框架分离开来

------

### HIDL

用于指定 HAL 和其用户之间的接口的一种接口描述语言，允许指定类型和方法调用，是用于在可以独立编译的代码库之间进行通信的系统

可指定数据结构和方法签名，这些内容会整理归类到接口中，而接口会汇集到软件包中

#### HIDL 设计

Android 框架可以在无需重新构建 HAL 的情况下进行替换，HAL 由供应商或 SOC 制造商构建，放置在设备的 `/vendor` 分区中

- 互操作性：在可以使用各种架构、工具链和编译配置来编译的进程之间创建可互操作的可靠接口，HIDL 接口是分版本的，发布后不得再进行更改
- 效率：尽可能减少复制操作的次数；HIDL 定义的数据以 C++ 标准布局数据结构传递至 C++ 代码；还提供共享内存接口；支持 共享内存 和 快速消息队列 两种无需使用 RPC 调用的数据传输方法
- 直观：通过仅针对 RPC 使用 `in` 参数，数据传递过程中都不会改变数据的所有权，所有权始终属于调用函数，数据仅需要在函数被调用期间保留，可在被调用的函数返回数据后立即清除

[^RPC]: Remote Procudue call，远程过程调用

#### HIDL 语法

与 C 语言类似，但不使用 C 预处理器

- `/** */` 表示文档注释，`/* */` 表示多行注释，`//` 表示注释一直持续到行结束
- 不含可变参数
- 逗号用于分割序列元素，分号用于终止各个元素，包括最后的元素

**异步回调** ：由 HAL 用户提供、传递给 HAL 并由 HAL 调用以随时返回数据的接口

**同步回调** ：将数据从服务器的 HIDL 方法实现返回到客户端，不用于返回无效值或单个原始值得方法

**客户端** ：调用特定接口方法的进程，HAL 进程或框架进程可是一个接口的客户端和另一个接口的服务器

**扩展** ：表示向另一接口添加方法或类型的接口，一个接口只能扩展另一个接口

**生成** ：表示将值返回给客户端的接口方法，要返回一个非原始值或多个值，生成同步回调函数

**接口** ：方法和类型的集合，转换为 C++ 或 Java 中的类；接口中的所有方法均按同一方向调用；客户端进程会调用由服务器进程实现的方法

**单向** ：表示该方法既不返回任何值也不会造成阻塞

**直通式** ：HIDL 的一种模式，服务器是共享库，由客户端进行 `dlopen` 处理，客户端和服务器是相同的进程，但代码库不同，仅用于将旧版代码库并入 HIDL 模型

**服务器** ：实现接口方法的进程

**传输** ：在服务器和客户端之间移动数据的 HIDL 基础架构

**版本** ：由两个整数组成：Major.Minor，Minor 版本可以添加（但不会更改）类型和方法

------

### 接口和软件包

HIDL 是围绕接口进行编译，接口是面向对象语言使用的一种用来定义行为的抽象类型

#### 软件包

可以具有子级，已发布的 HIDL 软件包的根目录是 `hardware/interfaces` 或 `vendor/vendorName` ；

| 软件包前缀           | 位置                                 |
| -------------------- | ------------------------------------ |
| android.hardware.*   | hardware/interfaces/*     (WIFI)     |
| android.frameworks.* | frameworks/hardware/interfaces/*     |
| android.system.*     | system/hardware/interfaces/*  (WIFI) |
| android.hidl.*       | system/libhidl/transport/*           |

软件包目录中包含扩展为 .hal 的文件，每个文件必须包含一个指定文件所属的软件包和版本的 package 语句，文件 `types.hal` （如果存在）并不定义接口，而是定义软件包中每个接口可以访问的数据类型

#### 接口定义

除 `types.hal` 外，其他 .hal 文件均定义一个接口

不含显式 `extend` 声明的接口会从 `android.hidl.base@1.0::IBase` 

#### 导入

`import` 语句是用于访问其他软件包中的软件包接口和类型的 HIDL 机制，涉及两个实体：

- 导入实体：软件包或接口
- 被导入实体：软件包或接口

当导入语句位于软件包的 `types.hal` 中时，导入的内容对整个软件包可见；当位于接口文件时，接口级导入

导入情形：

- 完整软件包导入：将整个软件包导入
- 部分导入：如果为一个接口，则将该软件包的 `types.hal` 和该接口导入；如果为 `types.hal` 定义的类型，则仅将该类型导入
- 仅类型导入：为 `types` ，仅导入指定软件包的 `types.hal` 中的类型

#### 接口继承

接口可以是之前定义的接口的扩展：

- 向其他接口添加功能，并按原样纳入其 API
- 软件包可向其他软件包添加功能，并按原样纳入其 API
- 接口可从软件包或特定接口导入类型

接口只能扩展一个其他接口（不支持多重继承），扩展并不意味着生成的代码中存在代码库依赖关系或跨 HAL 包含关系，接口扩展只是在 HIDL 级别导入数据结构和方法定义

HAL 中的每个方法必须在相应 HAL 中实现

#### 供应商扩展

某些情况下，供应商扩展会作为 代表其扩展的核心接口的基础对象 的子类予以实现，同一对象会同时在基础 HAL 名称和版本下，以及扩展的 HAL 名称和版本下注册

#### 版本编号

两个整数表示：Major.Minor

- Major 版本不向后兼容，递增将会使 Minor 版本号重置为0
- Minor 版本向后兼容，递增则兼容之前的版本，可添加新的数据结构和方法，但不能更改现有的数据结构或方法

#### 版本控制文件

`current.txt` 为版本控制文件，应包含使用 `hidl-gen` 中的 `-Lhash` 创建的接口的哈希

一般在大的独立的包名下，如 `/hardware/interfaces/current.txt` ，包含了大多模块的 HAL

------

### 接口哈希

一种旨在防止意外更改接口并确保接口更改经过全面审查的机制。HIDL 接口带有版本编号，接口一经发布不得再更改，但不会影响应用二级制接口（ABI）的情况

#### 布局

每个软件包根目录（即映射到 `hardware/interfaces` 的 `android.hardware` 或映射到 `vendor/foo/hardware/interfaces` 的 `vendor.foo` ）都必须包含一个列出所有发布 HIDL 接口文件的 `current.txt` 文件

<!--为便于跟踪各个哈希的来源，Google 将 HIDL current.txt 分为不同的部分：第一部分列出在 Android O 中发布的接口文件，第二部分列出在 Android O MR1 中发布的接口文件-->

#### 使用 `hidl-gen` 添加哈希

添加哈希至 `current.txt` 中，可手动添加，也可使用 `hidl-gen` 添加

<!--不能更换之前发布的接口的哈希，如需更改，需要在 current.txt 文件末尾添加新的哈希-->

`hidl-gen` 生成的每个接口定义库都包含哈希，通过调用 `IBase::getHashChain` 可检索这些哈希；`hidl-gen` 编译接口时，会检查 HAL 软件包根目录中的 `current.txt` 文件，以查看 HAL 是否已被更改

- 若没有找到 HAL 的哈希，则接口会被视为未发布，编译继续
- 若找到相应哈希，对当前接口进行检查：
  - 若与哈希匹配，编译继续
  - 若不匹配，编译暂停，意味着之前发布的接口被更改

#### ABI 稳定性

应用二进制接口（ABI）包括二进制关联/调用规范，若 ABI/API 发生更改，则相应接口不再适用于官方接口编译的常规 `system.img` 

确保接口带有版本号且 ABI 稳定至关重要：

- 确保能够通过供应商测试套件（VTS）测试，通过测试后，可能够正常进行仅限框架的 OTA
- 作为原始设备制造商（OEM），能够提供简单易用且符合规定的板级支持包（BSP）
- 有助于跟踪哪些接口可以发布

------

### 服务和数据传输

#### 注册服务

HIDL 接口服务器（实现接口的对象）可注册为已命名的服务，注册的名称不需要与接口或软件包名称相关，若没有指定名称，则使用默认的名称。

```c++
// hardware/interfaces/wifi/1.1/default/service.cpp
int main(int, char** argv) {
    ...;
    // Setup hwbinder service
    android:sp<android::hardware::wifi::V1_1::IWifi> service =
        new android:hardware::wifi::V1_1::implementation::Wifi();
    CHECK_EQ(service->registerAsService(), android::NO_ERROR)
        << "Failed to register with HAL";
}
```

<!--sp (strong pointer) 强指针，通过引用计数来记录有多少使用者在使用一个对象，如果所有使用者都放弃了该对象的引用，则该对象将被自动销毁-->

<!--wp (weak pointer) 弱指针，仅仅记录该对象的地址，不能通过弱指针来访问该对象-->

HIDL 接口的版本包含在接口本身，版本自动与服务注册关联，可通过每个 HIDL 接口上的方法调用（`android::hardware::IInterface::getInterfaceVersion()`) 进行检索

服务器对象不需要注册，可通过 HIDL 方法参数传递到其他进程，相应的接收进程会向服务器发送 HIDL 方法调用

#### 发现服务

客户端代码按名称和版本请求指定的接口，并对所需的 HAL 类调用 `getService` 

```java
// framework/opt/net/wifi/service/java/com/android/server/wifi/HalDeviceManager.java
/**
 * Wrapper function to access the HIDL service
 */
protected IWifi getWifiServiceMockable() {
    try {
        return IWifi.getService();
    } catch (RemoteException e) {
        Log.e(TAG, "Exception getting IWifi service:" + e);
        return null;
    }
}
```

每个版本的 HIDL 接口都会被视为单独的接口，大多数情况下，注册或发现服务均无需提供名称参数

#### 服务终止通知

客户端接收服务终止时的通知，必须：

1. 将 HIDL 类/接口 `hidl_death_recipient` (位于 C++ 代码中，非 HIDL)，归入子类
2. 替换其 `serviceDied()` 方法
3. 实例化 `hidl_death_recipient` 子类对象
4. 在要监控的服务上调用 `linkToDeath()` 方法，并传入 `IDeathRecepient` 接口对象

```java
// HalDeviceManager.java
private final HwRemoteBinder.DeathRecipient mIWifiDeathRecipient =
    cookie -> {
        Log.e(TAG, "IWifi HAL service died! Have a listener for it ... cookie = " + cookie);
};

if (!mWifi.linkToDeath(mIWifiDeathRecipient, 0)) {
    Log.e(TAG, "Error on linkToDeath on IWifi - will retry later");
    return;
}
```

#### 数据转移

可通过调用 `.hal` 文件内的接口中定义的方法将数据发送到服务：

- **阻塞** 方法会等到服务器产生结果
- **单向** 方法仅朝一个方向发送数据且不阻塞；如果 RPC (Remote Procedure Call) 调用中正在传输的数据量超过实际限制，则调用可能会阻塞或返回错误指示

不返回值但未声明为 `oneway` 的方法仍会阻塞

在 HIDL 接口中声明的所有方法都是单向调用，可从 HAL 发出，也可到 HAL，没有指定具体调用方向

##### 回调

> **同步回调** 用于返回数据的一些 HIDL 方法，返回多个值（或返回非基元类型的一个值）的 HIDL 方法会通过回调函数返回其结果；服务器实现 HIDL 方法，客户端实现回调
>
> **异步回调** 允许 HIDL 接口的服务器发起调用，通过第一个接口传递第二个接口的实例完成，第一个接口的客户端必须作为第二个接口的服务器，第一个接口的服务器可在第二个接口对象上调用方法；HAL 实现可以通过在由该进程创建和提供的接口对象上调用方法来讲信息异步发送回正在使用的进程；用于异步回调的接口中的方法可以是阻塞方法（并且可能将值返回到调用程序），也可以是 `oneway` 方法
>
> 方法调用和回调只能接受 `in` 参数，不支持 `out` 或 `inout` 参数

##### 事务限制

> 强制限制在 HIDL 方法和回调中发送的数据量，可能小至 4K，超出这些限制的调用会立即返回失败；另一个限制是可供 HIDL 基础架构处理多个同时进行的事务资源；由于多个线程或进程向同一个进程发送调用或者接收进程未能快速处理多个 `oneway` 调用，因此多个事务可能会同时进行

##### 方法实现

> HIDL 生成以目标语言（C++ 或 JAVA）声明的必要类型、方法和回调的标头文件；客户端和服务器代码的 HIDL 定义方法和回调的原型是相同的；HIDL 系统提供调用程序端的方法代理实现，并将代码存根到被调用程序端
>
> 函数的调用程序（HIDL 方法或回调）拥有对传递到该函数的数据结构的所有权，并在调用后保留所有权；被调用程序在所有情况下都无需释放存储
>
> - 在 C++ 中，数据可能是只读的（尝试写入可能会导致细分错误），并且在调用期间有效。客户端可以深层复制数据，以在调用期间外传播
> - Java 中，代码会接收数据的本地副本（普通 Java 对象），代码可以保留和修改此数据或允许垃圾回收器回收

##### 非 RPC 数据转移

> HIDL 在不使用 RPC 调用的情况下通过两种方法转移数据，C++ 同时支持两种
>
> - **共享内存** 内置 HIDL 类型 `memory` 用于传递表示已分配的共享内存的对象，可在接收进程中使用，以映射共享内存
>
> - **快速消息队列（FMQ）** 一种无等待消息传递的模板化消息队列类型，设置末尾，从而创建可以借助内置 HIDL 类型 `MQDescriptorSync` 或 `MQDescriptorUnSync` 的参数通过 RPC 传递的对象；接收进程可使用此对象设置队列的另一端
>
>   - “已同步”  不能溢出，只能有一个读取器
>   - “未同步”  可以溢出，可以有多个读取器，每个读取器必须及时读取数据，否则数据丢失
>
>   两种队列都不能下溢（从空队列进行读取会失败），只能有一个写入器

