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