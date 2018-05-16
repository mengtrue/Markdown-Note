### 获取 Log

> Android system logging
>
> ```
> adb logcat -v threadtime > logcat.txt
> adb shell cat /proc/kmsg > dmesg.txt           // kernel log, it only show next kernel
> adb shell dmesg | tee "dmesg.log"              // kernel log, include more content
> adb logcat -b radio > radiologs.txt
> adb shell top -t > threads.txt
> adb logcat -b events > events.txt
> adb bugreport > bugreport.txt
> adb pull /data/anr
> adb pull /data/tombstones
> ```
>
> dumpsys
>
> ```
> dumpsys bluetooth_manager
> dumpsys wifi
> dumpsys wifip2p
> dumpsys power
> dumpsys battery                      // check any service information
> 
> dumpsys meminfo
> dumpsys meminfo com.android.bluetooth   check any app (package name) memory information
> 
> dumpsys package (activity) package_name
> dumpsys netstats
> 
> dumpsys procstats
> dumpsys procstats com.android.bluetooth
> dumpsys procstats --hour 3
> 
> showmap "pid-of-com.android.bluetooth"
> ```
>
> 解锁
>
> ```
> adb root
> adb disable-verity
> adb reboot
> ```
>
> 

### 蓝牙

> 查看蓝牙状态
>
> ```
> adb shell dumpsys bluetooth_manager | grep state:
> ```
>
> 打开蓝牙设置界面
>
> ```
> adb shell am start -a android.settings.BLUETOOTH_SETTINGS
> ```
>
> 关闭蓝牙
>
> ```
> adb root
> adb wait-for-device shell service call bluetooth_manager 8
> ```
>
> 打开蓝牙
>
> ```
> adb root
> adb wait-for-device shell service call bluetooth_manager 6
> ```
>
> 谷歌 Daydream 允许关闭蓝牙(因只允许控制器进行控制，蓝牙需要保持开启)
>
> ```
> adb root
> adb wait-for-device
> adb shell am startservice \
>     -n com.google.android.gms/.config.ConfigService \
>     -a com.google.android.gms.config.OVERRIDE \
>     --es __package__ com.google.vr.vrcore \
>     --es don_auto_enable_bluetooth "false"
> adb shell am startservice \
>     -n com.google.android.gms/.config.ConfigService \
>     -a com.google.android.gms.config.OVERRIDE \
>     --es __package__ com.google.vr.vrcore \
>     --es __namespace__ configns:p4 \
>     --es controller_recovery_enabled "false"
> adb reboot
> ```
>
> 谷歌 Daydream 还原关闭蓝牙设置
>
> ```
> adb root && adb wait-for-device
> adb shell am startservice \
>     -n com.google.android.gms/.config.ConfigService \
>     -a com.google.android.gms.config.OVERRIDE \
>     --es __package__ com.google.vr.vrcore \
>     --es __namespace__ configns:p4 \
>     --ez don_auto_enable_bluetooth false
> adb shell am startservice \
>     -n com.google.android.gms/.config.ConfigService \
>     -a com.google.android.gms.config.OVERRIDE \
>     --es __package__ com.google.vr.vrcore \
>     --es __namespace__ configns:p4 \
>     --ez controller_recovery_enabled false
> adb reboot
> ```
>
> 