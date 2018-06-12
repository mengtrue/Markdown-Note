# Android Bluetooth

Android 使用了 Bluedroid 协议栈，包括：实现核心蓝牙功能的蓝牙嵌入式系统（BTE）与 Android 框架应用通信的蓝牙应用层（BTA）

## 概览

### 架构

蓝牙系统服务通过 JNI 与蓝牙协议栈进行通信，并通过 Binder IPC 与应用通信

![architecture](..\png\Android_Bluetooth\architecture.png)

**应用框架** 利用 `android.bluetooth` API 与蓝牙硬件进行交互，内部通过 Binder IPC 机制调用蓝牙进程

**蓝牙系统服务** （`package/apps/Bluetooth` ) 被打包为 Android 应用，并在 Android 框架层实现蓝牙服务和配置文件，通过 JNI 调用 HAL 层

**JNI** `packages/apps/Bluetooth/jni` ，当发生特定蓝牙操作时，调用 HAL 层并从 HAL 接收回调

**HAL** 定义了 `android.bluetooth` API 和蓝牙进程会调用的标准接口，并且必须实现该接口才能使蓝牙硬件正常工作，头文件为 `hardware/libhardware/include/hardware/bluetooth.h` ，另外，其他 `bt_*.h` 也需要关注

**协议栈** `system/bt` ，实现常规蓝牙 HAL，并通过扩展程序和更改配置对其进行定义

**供应商扩展程序** 可创建一个 `libbt-vendor` 模块添加并指定自定义扩展程序和用于跟踪 HCI

### 实现 HAL

`bluetooth.h` 包含蓝牙协议栈的基本接口，必须实现其功能

| 文件                                          | 作用                                        |
| --------------------------------------------- | ------------------------------------------- |
| bt_av.h                                       | A2DP 配置及接口定义                         |
| bt_gatt.h  bt_gatt_client.h  bt_gatt_server.h | GATT 配置及接口定义                         |
| bt_hf.h                                       | HFP 配置及接口定义                          |
| bt_hh.h                                       | HID 配置及接口定义                          |
| bt_hl.h                                       | Heath Device Profile (HDP) 配置及接口定义   |
| bt_mce.h                                      | Message Access Profile (MAP) 配置及接口定义 |
| bt_pan.h                                      | Personal Area Network (PAN) 配置及接口定义  |
| bt_rc.h                                       | AVRCP 配置及接口定义                        |
| bt_sock.h                                     | RFCOMM 套接字接口定义                       |

### 自定义原生蓝牙协议栈

对原生协议栈进行自定义设置：

- 自定义蓝牙配置文件——必须提供 SDK 插件下载方式，以使配置文件可供应用开发者使用，相应的 API 在蓝牙系统->进程应用（`packages/apps/Bluetooth`) 间可用，并添加到默认协议栈（`system/bt`)
- 自定义供应商扩展程序和配置更改——创建 `libbt-vendor` 以添加内容，如额外的 AT 命令或特定于设备的配置更改，参考：`hardware/broadcom/libbt`
- 主机控制器接口（HCI）——创建一个主要用于调试跟踪的 `libbt-hci` 模块提供自定义的 HCI，参考：`system/bt/hci`

------

## 服务

通过蓝牙，设备可以传输数据并供各种交互式服务（如音频、短信和电话）

### 音频

Android 设备是音频源，呈现设备（音响、耳机等）是接收器

#### 绝对音量控制

> 在 Android M 及更高版本中，协议栈允许音频源设置绝对音量，以便用户准确控制音频音量
>
> 音频源设备将音量信息和未衰减的音频发送到接收器，接收器根据音量信息放大音频，以便用户听到准确的播放音量
>
> 音频源设备还可注册接收音量通知，当用户使用接收器上的控件更改音量时，接收器便会向音频源发送通知，音频源能够准确在界面上显示音量信息
>
> 绝对音量控制默认处于开启状态，设置->系统->开发者选项->绝对音量功能

#### 高级音频编解码器

> Android O 中，使用 A2DP 的设备可以支持额外的音频编解码器，当设备连接到远程音频接收器时，协议栈支持音频编解码器协商，该协商旨在选择发送器和接收器都支持的最佳编解码器，以提供高质量的音频，之后所有音频都会先路由选定的编码器，然后再发送到接收器

####  实现

> 运行 Android O 且支持 A2DP 的设备会自动获得额外编码器支持，设备制造商可能需要针对某些专有音频编解码器获得单独的许可和二进制 BLOB(Binary Large Object)
>
> ```xml
> <!-- /packages/apps/Bluetooth/res/values/config.xml -->
> <!-- Configuring priorities of A2DP source codecs.
>      Large value means higher priority. -1 means the codecs is disabled.
>      Value 0 is reserved and should not be used here -->
> <integer name="a2dp_source_codec_priority_sbc">1001</integer>
> <integer name="a2dp_source_codec_priority_aac">2001</integer>
> <integer name="a2dp_source_codec_priority_aptx">3001</integer>
> <integer name="a2dp_source_codec_priority_aptx_hd">4001</integer>
> <integer name="a2dp_source_codec_priority_ldac">5001</integer>
> ```

#### LDAC 认证

> Android 开源项目纳入了索尼的 LDAC 编解码器，该编解码器不需要单独的许可或 BLOB；
>
> 若需要将 LDAC 编解码器集成到设备中，需要向索尼提交申请，并按照 [LDAC 认证流程](https://www.sony.net/Products/LDAC/aosp/) 进行操作
>
> LDAC 认证网站提供了关于 LDAC 的文档（规范和操作手册），还提供了针对移动设备和平板电脑设备的验证和互操作性测试，需要将表明通过测试的结果发送给索尼，才能完成 LDAC 认证

#### 界面功能

> Android O 提供了 停用高清（HD）蓝牙音频编解码器功能
>
> *对“设置”进行自定义的设备制造商应为用户实现一种停用高清编解码器的方式*

------

### 短信

借助 MAP，可从远程设备阅读、浏览和撰写短信，通常是将手机连接到车载信息娱乐系统

### 电话

除一般意义的 HFP，Android O 还支持手机默认铃声，当通过蓝牙连接的手机收到来电时，接收器上将会播放铃声，默认在 设置->系统->开发者选项，启用手机默认铃声

### 蓝牙功能

经典蓝牙：

| 名称            | 说明                                     | 6.0   | 7.0   | 7.1   | 7.1.2 | 8    |
| --------------- | ---------------------------------------- | ----- | ----- | ----- | ----- | ---- |
| SAP             | SIM 卡访问                               | 1.1   | 1.1   | 1.1   | 1.1   | 1.1  |
| MAP             | 短信访问                                 | 1.2   | 1.2   | 1.2   | 1.2   | 1.2  |
| OPP             | Object Push                              | 1.1   | 1.1   | 1.1   | 1.1   | 1.2  |
| OBEX over L2CAP | 对象交换                                 | Y     | Y     | Y     | Y     | Y    |
| HFP             | 免提                                     | 1.6   | 1.6   | 1.7   | 1.7   | 1.7  |
| HSP             | 耳机                                     | 1.2   | 1.2   | 1.2   | 1.2   | 1.2  |
| A2DP            | 高级音频                                 | 1.2   | 1.2   | 1.2   | 1.2   | 1.2  |
| AVRCP           | 音视频远程控制                           | 1.3   | 1.3   | 1.3   | 1.3   | 1.4  |
| HID             | 人机接口                                 | 1.0   | 1.0   | 1.0   | 1.0   | 1.0  |
| PBAP            | 电话簿访问                               | 1.1.1 | 1.1.1 | 1.1.1 | 1.1.1 | 1.2  |
| HDP             | 健康设备访问                             | 1.0   | 1.0   | 1.1   | 1.1   | 1.1  |
| SPP             | Serial Port                              | 1.2   | 1.2   | 1.2   | 1.2   | 1.2  |
| PAN/BNEP        | Bluetooth Network Encapsulation Protocol | 1.0   | 1.0   | 1.0   | 1.0   | 1.0  |
| DIP             | Device ID                                | 1.3   | 1.3   | 1.3   | 1.3   | 1.3  |
| HOGP 1.0        | HID over GATT                            | Y     | Y     | Y     | Y     | Y    |
| HD Audio        | 高清音频                                 | N     | N     | N     | N     | Y    |

BLE 功能：

| 名称              | 6.0  | 7.0  | 7.1  | 7.1.2 | 8    |
| ----------------- | ---- | ---- | ---- | ----- | ---- |
| BR/EDR 安全连接   | 4.1  | 4.1  | 4.1  | 4.1   | 5.0  |
| LE 隐私           | 4.2  | 4.2  | 4.2  | 4.2   | 5.0  |
| LE 安全连接       | 4.2  | 4.2  | 4.2  | 4.2   | 5.0  |
| 数据包扩展        | 4.2  | 4.2  | 4.2  | 4.2   | 5.0  |
| 32 bit UUID       | Y    | Y    | Y    | Y     | Y    |
| 双模 LE           | Y    | Y    | Y    | Y     | Y    |
| Google HCI        | Y    | Y    | Y    | Y     | Y    |
| LE 连接导向型频道 | N    | N    | N    | N     | Y    |

------

## BLE

Android 4.3及更高版本张提供过的 BLE （低功耗蓝牙）可在多个设备之间实现短促连接，以传输增加的数据量。未建立连接时，BLE 处于休眠模式，可提供比经典蓝牙更低的带宽和功耗，适用于心率检测器或无线键盘

### 实现

所有 BLE 应用基于 GATT (通用属性配置)，当 Android 设备与 BLE 设备交互时，发送信息的设备是服务器，接收信息的设备是客户端；设备制造商需要实现相应的 LE HCI 要求以能够充分利用 BLE API

### 设备模式

使用 BLE 时，Android 设备可发挥外围设备、中心设备的作用，或同时发挥两者的作用；外围设备模式可让设备发送广告包，中心模式可让设备扫描广告；Android 设备可一边在外围设备模式下发送广告，一边与其他 BLE 外围设备进行通信；支持蓝牙 4.1 及更低版本的设备只能在中心模式下使用 BLE

### BLE 扫描

Android 设备可更有效定位和扫描特定的蓝牙设备，BLE API 可让应用开发者创建过滤器，以让主机控制器查找参与度较低的设备

*需要获取位置权限，BLE 扫描功能会识别可用于地理定位的对象，关闭位置信息服务也将一并关闭蓝牙扫描功能*

#### 位置信息扫描

> 用户设备的位置信息服务可使用蓝牙来检测蓝牙信标，并提供更准确的位置信息
>
> 个别应用需要获取位置权限才能使用 BLE 扫描功能，即使只是为了查找要连接的设备，若停用位置扫描功能，或者未向应用提供位置权限，则不会收到任何 BLE 扫描结果
>
> 停用系统级蓝牙后台扫描：设置->安全性和位置信息->位置信息->扫描，不会影响对位置或本地设备的 BLE 扫描

#### 过滤扫描结果

> Android 6.0 及更高版本包括蓝牙控制器上的 BLE 扫描和过滤器匹配，设备可以过滤扫描结果，并将与 BLE 设备相关的 `found` 和 `lost` 事件报告给 AP（应用处理器），BLE 扫描已分流到固件中，过滤功能也适用于批量扫描，有助于节省电量 ，批量扫描可降低 AP 因设备或信标的 BLE 扫描而唤醒的频率
>
> `onFound` 和 `onLost` 功能在蓝牙控制器中实现，然后经过测试以确认在扫描过程中未错过 BLE 设备，可节省电量，以及：
>
> - 对于 `onFound` 事件，AP 在发现特定设备时唤醒
> - 对于 `onLost` 事件，AP 在找不到特定设备时唤醒
> - 当附近设备在检测范围内时，框架应用可收到较少的无用通知
> - 当设备不在检测范围时，连续扫描功能可启动框架应用以收到通知
>
> 扫描过滤器可根据发现的设备广告（`onFound`）创建，Java 层可指定参数，如首次发现的特定广告数量等

------

## BLE 广播

BLE 大部分时间处于休眠模式以节省电量，只会在进行广播和进行连接时唤醒

### Blutooth 5.0 广播

Android O 支持 Bluetooth 5.0，对 BLE 广播功能进行改进，并提供灵活的数据广播功能，支持 BLE 物理层（PHY），其保留了 Bluetooth 4.2 耗电量少，还允许用户选择更大的带宽或范围

#### 实现

> 可使用 `BluetoothAdapter` 方法检查设备是否支持 Bluetooth 5.0 功能
>
> - `isLe2MPhySupported()`   if LE 2M PHY feature is supported
> - `isLeCodedPhySupported()`    if LE Coded PHY feature is supported
> - `isLeExtendedAdvertisingSupported()`     if LE Extended Advertising feature is supported
> - `isLePeriodicAdvertisingSuspported()`    if LE Periodic Advertising feature is supported
>
> 默认，Android O 使用 Bluetooth 4.2 的 LE 1M PHY
>
> `android.bluetooth.le` `frameworks/base/core/java/android/bluetooth/le` 通过以下 API 文件提供广播：
>
> - AdvertisingSet.java
> - AdvertisingSetCallback.java
> - AdvertisingSetParameters.java
> - PeriodicAdvertisingParameters.java
>
> 以下使用 LE 1M PHY 进行广播
>
> ```java
> BluetoothLeAdvertiser advertiser = BluetoothAdapter.getDefaultAdapter().
>     getBluetoothLeAdvertiser();
> AdvertisingSetParameters parameters = (new AdvertisingSetParameters.Builder())
>     .setLegacyMode(true)
>     .setConnectable(true)
>     .setInterval(AdvertisingSetParameters.INTERVAL_HIGH)
>     .setTxPowerLevel(AdvertisingSetParameters.TX_POWER_MEDIUM)
>     .build();
> AdvertiseData data = (new AdvertiseData.Builder()).setIncludeDeviceName(true).build();
> AdvertisingSetCallback callback = new AdvertisingSetCallback() {
>     @Override
>     public void onAdvertisingSetStarted(AdvertisingSet advertisingSet, int txPower, int status) {
>         Log.i(TAG, "onAdvertisingSetStarted, txPower: " + txPower + ", status: " + status);
>         currentAdvertisingSet = advertisingSet;
>     }
>     
>     @Override
>     public void onAdvertisingDataSet(AdvertisingSet advertisingSet, int status) {
>         Log.i(TAG, "onAdvertisingDataSet, status: " + status);
>     }
>     
>     @Override
>     public void onScanResponseDataSet(AdvertisingSet advertisingSet, int status) {
>         Log.i(TAG, "onScanResponseDataSet, status: " + status);
>     }
>     
>     @Override
>     public void onAdvertisingSetStopped(AdvertisingSet advertisingSet) {
>         Log.i(TAG, "onAdvertisingSetStopped");
>     }
> };
> advertiser.startAdvertisingSet(parameters, data, null, null, null, callback);
> 
> // After onAdvertisingSetStarted callback is called
> // can modify the advertising data and scan response data
> currentAdvertisingSet.setAdvertisingData(new AdvertiseData.Builder().
>                                         setIncludeDeviceName(true)
>                                         .setIncludeTxPowerLevel(true)
>                                         .builder());
> currentAdvertisingSet.setScanResponseData(new AdvertiseData.Builder()
>                                          .addServiceUuid(
>                                              new ParcelUuid(UUID.randomUUID()))
>                                          .build());
> advertiser.stopAdvertisingSet(callback);
> ```
>
> 以下应用 LE 2M PHY 进行广播
>
> ```java
> BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
> if (!adapter.isLe2MPhySupported()) {
>     Log.e(TAG, "2M PHY not supported");
>     return;
> }
> if (!adapter.isLeExtendedAdvertisingSupported()) {
>     Log.e(TAG, "LE Extended Advertising not supported");
>     return;
> }
> int maxDataLength = adapter.getLeMaximumAdvertisingDataLength();
> AdvertisingSetParameters.Builder parameters = (new AdvertisingSetParameters.Builder())
>     .setLegacyMode(false)
>     .setInterval(AdvertisingSetParameters.INTERVAL_HIGH)
>     .setTxPowerLevel(AdvertisingSetParameters.TX_POWER_MEDIUM)
>     .setPrimaryPhy(BluetoothDevice.PHY_LE_2M)
>     .setSecondaryPhy(BluetoothDevice.PHY_LE_2M);
> AdvertiseData data = (new AdvertiseData.Builder()).addServiceData(
>     new ParcelUuid(UUID.randomUUID()), "xxxxxx".getBytes()).build();
> advertiser.startAdvertisingSet(parameters.build(), data, null, null, null, callback);
> 
> // also stop and restart the advertising
> currentAdvertisingSet.enableAdvertising(false, 0, 0);
> ```

#### 实现

> Android 通讯测试套件（ACTS）包括针对 Bluetooth 5.0 的测试
>
> `tools/test/connectivity/acts/tests/google/ble/bt5/`

------

## 验证和调试

可使用 AOSP 中提供的工具进行验证和调试蓝牙协议栈

### 测试和验证

#### 单元测试

> 针对默认蓝牙协议栈的功能测试和单元测试，`system/bt/test` 
>
> ```markdown
> ## 停止 Android 运行
> adb shell stop
> ## 测试目录中运行 shell 可执行文件
> ./run_unit_tests.sh TEST_GROUP_NAME TEST_NAME OPTIONS
> ## 测试完成，重新启用 Android
> adb shell start
> ```
>
> 测试名称列表位于以下文件：`/system/bt/test/run_unit_tests.sh`

#### Android 通讯测试套件

> 测试工具需要 adb 和 python，`tools/test/connectivity/acts`
>
> 针对蓝牙和蓝牙低功耗的 ACTS 测试分别位于：`tools/test/connectivity/acts/tests/google/bt` 和 `tools/test/connectivity/acts/tests/google/ble` 

#### Profile Tuning Suite

> 辅助 Bluetooth PTS（Bluetooth Tuning Suite）：
>
> `tools/test/connectivity/acts/tests/google/bt/pts`

#### CTS 测试

> 兼容性测试套件包括针对蓝牙协议栈的测试：
>
> `cts/apps/CtsVerifier/src/com/android/cts/verifier/bluetooth`

### 调试选项

更改不同配置文件的日志记录级别，`system/bt/conf/bt_stack.conf`

从抓取的 snoop log 提取信息收集日志，可使用 `btsnooz` 脚本

```markdown
/system/bt/tools/scripts/btsnooz.py
获取 snoop 日志
btsnooz.py BUG_REPORT.txt > BTSnoop.log
```

