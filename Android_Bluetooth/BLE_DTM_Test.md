支持 BLE 的设备必须进行 DTM（Direct Test Mode） 测试，此项测试，一般为在三方实验室进行，条件允许的设备生产厂家也有可能在内部进行；三方实验室面对通用设备，即设备必须满足在三方实验室条件，能够在实验室环境下进行测试。

------

## Android/高通平台设备/CBTgo/CBT

设备平台：高通  Android

PC软件：CBTgo

测试仪器：CBT

连接图如下：

![CBT_device_connections](..\png\Android_Bluetooth\cbt_device_connections.png)

> 测试步骤
>
> 1. 测试设备关闭 蓝牙，可通过 UI 操作，或提供相应指令进行，查看是否关闭：
>
>    ```
>    adb shell dumpsys bluetooth_manager | grep state:
>    ```
>
> 2. 打开设备的 NMEA port
>
>    ```
>    adb shell setprop sys.usb.config ble_nmea
>    ```
>
>    **此指令，不同高通平台，可能不同，可以咨询高通 USB team**
>
>    指令之后，查看 PC 设备管理器，是否出现 GPS 或 NMEA COM
>
> 3. 开启设备蓝牙测试模式
>
>    ```
>    adb root
>    adb shell
>    ftmdaemon -n -dd
>    ```
>
> 4. 运行 BLE 测试模式
>
>    安装 QDART 后，在默认安装路径下，**C:\Program Files(x86)\Qualcomm\QDART\bin\QC.BluetoothLE_DirectMode.exe** 
>
>    ![qcom_ble_direct_mode](..\png\Android_Bluetooth\qcom_ble_direct_mode.png)
>
>    图中，COM Port 选择 CBT 连接 PC 的 UART port，点击 Enable，Handshake 选择 RequestToSendXOnXOff
>
> 5. PC 端 CBTgo
>
>    按下图操作：
>
>    ![cbtgo_1](..\png\Android_Bluetooth\cbtgo_1.png)
>
>    ![cbtgo_2](..\png\Android_Bluetooth\cbtgo_2.png)
>
>    以上步骤执行无误后，点击 Start 按钮开始 BLE 测试
>
> 6. 测试完毕后，CBTgo 会输出测试结果
>
>    disable 以断开 Bluetooth LE Direct Mode，结束 adb 中 ftmdaemon 运行