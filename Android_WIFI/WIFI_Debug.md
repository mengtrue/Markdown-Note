通常使用的 WIFI 芯片多为高通芯片，如 WCN3XXX 或 QCA61XX 系列，其他还有博通 BCMXXXXX，联发科或者展讯，这里记录 debug 或者查看相关信息

------

## 高通芯片

> **Firmware build info**
>
> - 使用 iwpriv
>
>   ```
>   adb root && adb shell && iwpriv wlan0 version
>   ```
>
> - 开关 WIFI，查看 kmsg
>
>   ```
>   adb shell cat /proc/kmsg
>   ```
>
> **Android host logs**
>
> - Android system logging
>
>   ```
>   adb logcat -v threadtime > logcat.txt
>   adb shell cat /proc/kmsg > dmesg.txt
>   adb shell dmesg | tee "dmesg.log"            // kernel log
>   adb logcat -b radio > radiologs.txt          // radio log
>   adb shell top -t > threads.txt               // thread information
>   adb logcat -b events > events.txt            // event logs
>   adb bugreport > bugreport.txt
>   adb pull /data/anr                           // traces txt
>   adb pull /data/tombstones                    // tombstones
>   ```
>
> - wpa_supplicant logging
>
>   ```
>   wpa_cli ifname=wlan0 log_level EXCESSIVE             // wlan0 interface logging
>   wpa_cli ifname=p2p0 log_level EXCESSIVE              // p2p0 interface logging
>   ```
>
> - WLAN host driver debug logs
>
>   WLAN host driver prints kernel logs and routes to the QXDM interface.
>
>   修改 log debug level，三种方法：
>
>   - iwpriv
>
>     ```
>     iwpriv wlan0 setwlandbg <module> <loglevel> <enable/disable>
>     ```
>
>     | Value | Module   | Log Level | Disable/Enable |
>     | ----- | -------- | --------- | -------------- |
>     | 0     | BAP      | NONE      | Disable        |
>     | 1     | TL       | FATAL     | Enable         |
>     | 2     | WDI      | ERROR     |                |
>     | 3     | RSV3     | WARN      |                |
>     | 4     | RSV4     | INFO      |                |
>     | 5     | HDD      | INFO_HIGH |                |
>     | 6     | SME      | INFO_MED  |                |
>     | 7     | PE       | INFO_LOW  |                |
>     | 8     | WDA      | DEBUG     |                |
>     | 9     | SYS      | ALL       |                |
>     | 10    | VOSS     |           |                |
>     | 11    | SAP      |           |                |
>     | 12    | HDD_SAP  |           |                |
>     | 13    | PMC      |           |                |
>     | 14    | HDD_DATA |           |                |
>
>     其中 WDI 模块还有额外的指令
>
>     | Value | WDI module            | Log level | Disable/Enable |
>     | ----- | --------------------- | --------- | -------------- |
>     | 0     | eWLAN_MODULE_DAL      | NONE      | Disable        |
>     | 1     | eWLAN_MODULE_DAL_CTRL | FATAL     | Enable         |
>     | 2     | eWLAN_MODULE_DAL_DATA | ERROR     |                |
>     | 3     | eWLAN_MODULE_PAL      | WARN      |                |
>     | 4     |                       | INFO      |                |
>     | 5     |                       | INFO_HIGH |                |
>     | 6     |                       | INFO_MED  |                |
>     | 7     |                       | INFO_LOW  |                |
>     | 8     |                       | DEBUG     |                |
>     | 9     |                       | ALL       |                |
>
>   - WLAN host driver source code
>
>     vendor/qcom/opensource/wlan/prima/CORE/VOSS/src/vos_trace.c
>
>   - WCNSS_qcom_cfg.ini
>
> **MAC 抓取 air log**
>
> - 打开 无线诊断  （点击无线图标，或按住 option）
> - 打开 嗅探器，选择要抓取的 频段 和 频宽，开始抓取，完毕后，结束
> - 抓取的 log 文件保存在 /var/tmp，在 Finder 中，shift + command + g，输入路径
>
> **WIFI 测试模式**
>
> ```
> adb root && adb shell
> ifconfig wlan0 up
> echo 5 > /sys/module/wlan/parameters/con_mode
> ftmdaemon -n -dd
> ```
>
> 