通常使用的 WIFI 芯片多为高通芯片，如 WCN3XXX 或 QCA61XX 系列，其他还有博通 BCMXXXXX，联发科或者展讯，这里记录 debug 或者查看相关信息

------

## 高通芯片

> Firmware build info
>
> - 使用 iwpriv
>
>   ```
>   adb root && adb shell && iwpriv wlan0 version
>   ```
>
> - 开关 WIFI，查看 kmsg