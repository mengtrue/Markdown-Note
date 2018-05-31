## WIFI 源码分析

这里记录 Android 源码中 WIFI 相关。

从 Android N 开始，WIFI 模块进行了重构：

1. 将某些功能单独取出成为独立模块，如扫描等
2. 增加新的控制管理，如 WifiInjector，WifiConnectivityManager
3. 其他基本调用流程不变，大致为 WifiService -> WifiStateMachine -> WifiNative -> JNI -> wpa_supplicant -> kernel -> driver

Android O 开始，又一次对 WIFI 进行重构

1. 加载驱动流程变更
2. 添加判断 STA/AP 是否共存
3. 在临近 HAL 层之上 framework 增加了 HAL 层的接口及设备管理接口
4. 使用 HIDL，不再使用 com_android_server_wifi_WifiNative.cpp，从 system.img 移除 hal ，方便升级

------



### 开机启动 Service

>SystemServer 是系统中非常核心的进程，在 ZygoteInit 中创建并启动，后启动所有系统相关 service
>
>WIFI 相关 service 在 `startOtherServices()` 中启动
>
>```java
>private static final String WIFI_SERVICE_CLASS =
>            "com.android.server.wifi.WifiService";
>private static final String WIFI_AWARE_SERVICE_CLASS =
>            "com.android.server.wifi.aware.WifiAwareService";
>private static final String WIFI_P2P_SERVICE_CLASS =
>            "com.android.server.wifi.p2p.WifiP2pService";
>// Wifi Service must be started first for wifi-related services.
>traceBeginAndSlog("StartWifi");
>mSystemServiceManager.startService(WIFI_SERVICE_CLASS);
>traceEnd();
>traceBeginAndSlog("StartWifiScanning");
>mSystemServiceManager.startService(
>        "com.android.server.wifi.scanner.WifiScanningService");
>traceEnd();
>
>if (!disableRtt) {
>    traceBeginAndSlog("StartWifiRtt");
>    mSystemServiceManager.startService("com.android.server.wifi.RttService");
>    traceEnd();
>}
>
>if (context.getPackageManager().hasSystemFeature(
>        PackageManager.FEATURE_WIFI_AWARE)) {
>    traceBeginAndSlog("StartWifiAware");
>    mSystemServiceManager.startService(WIFI_AWARE_SERVICE_CLASS);
>    traceEnd();
>} else {
>    Slog.i(TAG, "No Wi-Fi Aware Service (Aware support Not Present)");
>}
>
>if (context.getPackageManager().hasSystemFeature(
>        PackageManager.FEATURE_WIFI_DIRECT)) {
>    traceBeginAndSlog("StartWifiP2P");
>    mSystemServiceManager.startService(WIFI_P2P_SERVICE_CLASS);
>    traceEnd();
>}
>```

#### WifiService

> 最重要的 WIFI Service，代码在
>
> ```markdown
> frameworks/opt/net/wifi/service/java/com/android/server/wifi/WifiService.java
> frameworks/opt/net/wifi/service/java/com/android/server/wifi/WifiServiceImpl.java
> ```
>
> 即三方 APP 获取的 Wifi Service，抽象出的 API 均在 WifiManager
>
> WifiService 继承于 SystemService，具体实现于 WifiServiceImpl

##### 开机是否打开 WIFI

> 开机后，是否打开 WIFI，在 `checkAndStartWifi()` 中判定
>
> ```java
> // Check if wi-fi needs to be enabled
> boolean wifiEnabled = mSettingsStore.isWifiToggleEnabled();
> setWifiEnabled(mContext.getPackageName(), wifiEnabled);
> ```
>
> 查看关机时的WIFI状态（Settings.Global.WIFI_SAVED_STATE） > 是否飞行模式（是否开启 WIFI）> 查看 Settings.Global.WIFI_ON

#### WifiScanningService

> 处理 WIFI 扫描相关

#### RttService

> Round-Trip-Time，为 802.11mc 标准中的测量协议，提供误差在 1~2米的室内定位
>
> 下一代wifi定位标准为 802.11az，2021年推出

#### WifiAwareService

> Neighbor Awareness Network （NAN）协议，若工作在 2.4G，则使用信道 6（2.437 GHz），若工作在 5G，则使用信道 44（5.220 GHz）或 信道 149（5.745 GHz），工作方式类似 BLE，即在不连接时，可发现周围设备，可进行数据广播或者同步信息

#### WifiP2pService 

> 用于 WIFI  Direct，扫描及连接

------

### WIFI Enabled/Disabled

> 1. WifiManager(WifiService)[WifiController send CMD_WIFI_TOGGLED]
>
> 2. WifiController 是一个状态机，默认状态为 ApStaDisabledState，需要 `transitionTo DeviceActiveState` 
>
>    <u>状态机切换时，使用 ==transitionTo== 转换，但转换时需要注意状态机的初始化中，是否有这两个状态的直接转化，若没有，则需要由目的状态的父状态寻找，直到源状态的父状态，一般简单状态机仅需要一个或两个过渡即可</u>
>
>    首先，ApStaDisabledState -> DefaultState -> StaEnabledState[WifiStateMachine.setSupplicantRunning(true)] -> DeviceActiveState[WifiStateMachine.setOperationalMode(CONNECT), setHighPerfModeEnabled(false)]
>
> 3. WifiStateMachine 启动 Supplicant，默认在 InitialState，接收到 CMD_START_SUPPLICANT
>
>    - 加载驱动   WifiNative.setupForClientMode

------

### 扫描

共分为以下 4 中场景，优先级由高到低

#### 亮屏且处于WIFI 界面

> 固定时间扫描，时间间隔为 10s
>
> 由 framework/base/settingslib/WifiTracker.Scanner 控制并处理
>
> ```java
> // Combo scans can take 5-6s to complete - set to 10s.
> private static final int WIFI_RESCAN_INTERVAL_MS = 10 * 1000;
> ```

#### 亮屏非WIFI界面

> 二进制指数退避扫描，$interval * (2^n)$ ，称为 periodic scan
>
> 最小间隔为 20s，最大间隔 160s
>
> 由 framework/opt/wifi/WifiConnectivityManager 管理
>
> ```java
> public static final int PERIODIC_SCAN_INTERVAL_MS = 20 * 1000; // 20 seconds
> public static final int MAX_PERIODIC_SCAN_INTERVAL_MS = 160 * 1000; // 160 seconds
> ```

#### 灭屏有保存网络

> 若已连接，则不扫描
>
> 若未连接，则只扫描已保存的网络，称为 PNO scan，若无已保存网络，则不扫描
>
> 若未找到，则以 20s 间隔扫描，扫描到时，未连接成功，则以最小间隔 20s，最大间隔 80s扫描
>
> 由 framework/opt/wifi/WifiConnectivityManager 管理
>
> ```java
> // PNO scan interval in milli-seconds. This is the scan
> // performed when screen is off and disconnected.
> private static final int DISCONNECTED_PNO_SCAN_INTERVAL_MS = 20 * 1000; // 20 seconds
> // When a network is found by PNO scan but gets rejected by Wifi Network Selector due
> // to its low RSSI value, scan will be reschduled in an exponential back off manner.
> private static final int LOW_RSSI_NETWORK_RETRY_START_DELAY_MS = 20 * 1000; // 20 seconds
> private static final int LOW_RSSI_NETWORK_RETRY_MAX_DELAY_MS = 80 * 1000; // 80 seconds
> ```

#### 无保存网络

> 亮屏不在 WIFI 界面时，与第 2 种情况冲突
>
> 固定间隔 5 分钟扫描，用于通知用户周围存在可用 open network，称为 No network periodic scan
>
> 由 framework/opt/wifi/WifiStateMachine 管理
>
> ```java
> mNoNetworksPeriodicScan = mContext.getResources().getInteger(
>                 R.integer.config_wifi_no_network_periodic_scan_interval);
> 
> DisconnectedState
>    case CMD_NO_NETWORKS_PERIODIC_SCAN
> ```
>
> /frameworks/base/core/res/res/values
>
> ```java
> <!-- Integer indicating the framework no networks periodic scan interval in milliseconds. -->
> <integer translatable="false" name="config_wifi_no_network_periodic_scan_interval">300000</integer>
> ```

由以上可知，在原始代码情况下，仍有功耗优化的空间，如非 WIFI 界面，无网络，或仅保存一个且已连接等