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
> ```markdown
> dumpsys bluetooth_manager
> dumpsys wifi
> dumpsys wifip2p
> dumpsys power
> 
> ## /sys/class/power_supply/battery/capacity
> dumpsys battery         // check any service information
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
> 
> ## top，查看当前CPU，-d 间隔，-m 数量，-t 当前进程中所有线程
> adb shell top -d 3 -t | grep com.android.settings
> ```
>
> icnss log
>
> ```
> adb shell cat /d/ipc_logging/icnss/log_cont > icnss.log
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

### Mount

> 挂载 firmware
>
> ```objective-c
> adb shell mount -o rw,remount /firmware
> ```
>

### aapt

> ```objective-c
> aapt dump badging xxx.apk
> ```

### Battery

> device battery path
>
> ```
> /sys/class/power_supply/battery/capacity
> ```
>
> charge
>
> ```
> /sys/class/power_supply/battery # echo  1 > input_suspend    //不充电
> /sys/class/power_supply/battery # echo  0 > input_suspend    //充电
> ```
>

### Boot time

> ```markdown
> adb shell
> ## boot_progress_start : screen is on, start to show animation
> ## boot_progress_enable_screen : the boot complete
> logcat -d -b events | grep boot_progress
> ```

### Power off

>```
>adb reboot -p
>adb shell svc power shutdown
>```

### 使用 WIFI ADB

> ```markdown
> ## phone and PC connect to same WIFI AccessPoint
> adb root
> ## get phone WIFI IP address (IPXX)
> adb shell ifconfig wlan0
> ## set adb port
> adb IPXX 5555
> ## disconnect USB
> adb connect IPXX:5555
> ```
>

### 截图/录屏

> ```
> adb shell screencap -p /sdcard/Pictures/screenshot.png
> adb shell screenrecord --time-limit value(second) --size 1280*960 /sdcard/Movies/screenrecord.mp4
> ```

### WIFI

> ```
> adb shell svc wifi enable/disable
> ```

### USB

> ```markdown
> ## get current usb function
> adb shell svc usb getFunction
> ## set usb function (none, adb, mtp, ptp, midi, charging)
> adb shell svc usb setFunction mtp
> ```

### am

> ```markdown
> ## 发送广播
> adb shell am broadcast -a <ACTION> -es <EXTRA_KEY> <STRING_VALUE> -ez <EXTRA_KEY> <BOOLEAN_VALUE> -ei <EXTRA_KEY> <INT_VALUE>
> ## 启动 Activity
> adb shell am start -n package/activity
> ## 启动 Service
> adb shell am startservice -n package/service
> ```

### 查看系统信息

> ```markdown
> ## 查看版本
> adb shell getprop ro.build.version.incremental
> ```
>
> 