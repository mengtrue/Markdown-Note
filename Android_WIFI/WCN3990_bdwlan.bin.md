## 使用 bdwlan.bin

>WCN3990 的 WIFI NV文件 bdwlan.bin，包含了 Country Code及相关WIFI芯片配置  
如要修改，需要使用 高通自动的 bin 和 txt 转换工具，将其转换为 txt 后修改，后再转回 bin

### 单天线

>一般 2X2 天线分为 chain0 和 chain1，可以结合原理图、结构图确定，比较慢，可以设置后，在硬件上拆掉某根天线验证，具体修改为
>```
>使能 chain0
>concurrencyModeMask__0_0, 0x04
>txMask2G_T0_0_0, 0x1
>rxMask2G_T0_0_0, 0x1
>txMask5G_T0_0_0, 0x1
>rxMask5G_T0_0_0, 0x1
>使能 chain1
>concurrencyModeMask__0_0, 0x04
>txMask2G_T0_0_0, 0x2
>rxMask2G_T0_0_0, 0x2
>txMask5G_T0_0_0, 0x2
>rxMask5G_T0_0_0, 0x2
>```
