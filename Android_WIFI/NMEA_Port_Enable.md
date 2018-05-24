## NMEA Com Port

> 高通平台 Modem、GPS以及BLE相关测试中会用到NMEA Com Port，一般默认情况下均不使能  
NMEA 使能后：  
![NMEA](./png\NMEA_COM_Port.png)  
### Android O 使能
>1. 修改代码  
```
9025: DIAG + ADB + DUN + NMEA + QMI_RMNET
9026: DIAG + DUN + NMEA + QMI_RMNET
---
--- a/rootdir/etc/init.msm.usb.configfs.rc
+++ b/rootdir/etc/init.msm.usb.configfs.rc
     write /config/usb_gadget/g1/UDC ${sys.usb.controller}
     setprop sys.usb.state ${sys.usb.config}

on property:sys.usb.config=diag,serial_cdev,serial_cdev,rmnet,adb && property:sys.usb.configfs=1
       start adbd

on property:sys.usb.ffs.ready=1 && property:sys.usb.config=diag,serial_cdev,serial_cdev,rmnet,adb && property:sys.usb.configfs=1
       write /config/usb_gadget/g1/configs/b.1/strings/0x409/configuration "9025 without MSC"
       rm /config/usb_gadget/g1/configs/b.1/f1
       rm /config/usb_gadget/g1/configs/b.1/f2  
       rm /config/usb_gadget/g1/configs/b.1/f3  
       rm /config/usb_gadget/g1/configs/b.1/f4
       rm /config/usb_gadget/g1/configs/b.1/f5
       rm /config/usb_gadget/g1/configs/b.1/f6  
       rm /config/usb_gadget/g1/configs/b.1/f7
       rm /config/usb_gadget/g1/configs/b.1/f8    
       write /config/usb_gadget/g1/idVendor 0x05C6  
       write /config/usb_gadget/g1/idProduct 0x9025
       symlink /config/usb_gadget/g1/functions/diag.diag /config/usb_gadget/g1/configs/b.1/f1
       symlink /config/usb_gadget/g1/functions/ffs.adb /config/usb_gadget/g1/configs/b.1/f2  
       symlink /config/usb_gadget/g1/functions/cser.dun.0 /config/usb_gadget/g1/configs/b.1/f3  
       symlink /config/usb_gadget/g1/functions/cser.nmea.1 /config/usb_gadget/g1/configs/b.1/f4  
       symlink /config/usb_gadget/g1/functions/${sys.usb.rmnet.func.name}.rmnet /config/usb_gadget/g1/configs/b.1/f5
       write /config/usb_gadget/g1/UDC ${sys.usb.controller}
       setprop sys.usb.state ${sys.usb.config}

on property:sys.usb.config=diag,serial_cdev,serial_cdev,rmnet && property:sys.usb.configfs=1
       write /config/usb_gadget/g1/configs/b.1/strings/0x409/configuration "9026 without MSC"
       rm /config/usb_gadget/g1/configs/b.1/f1
       rm /config/usb_gadget/g1/configs/b.1/f2  
       rm /config/usb_gadget/g1/configs/b.1/f3  
       rm /config/usb_gadget/g1/configs/b.1/f4  
       rm /config/usb_gadget/g1/configs/b.1/f5
       rm /config/usb_gadget/g1/configs/b.1/f6  
       rm /config/usb_gadget/g1/configs/b.1/f7
       rm /config/usb_gadget/g1/configs/b.1/f8  
       write /config/usb_gadget/g1/idVendor 0x05C6  
       write /config/usb_gadget/g1/idProduct 0x9026  
       symlink /config/usb_gadget/g1/functions/diag.diag /config/usb_gadget/g1/configs/b.1/f1
       symlink /config/usb_gadget/g1/functions/cser.dun.0 /config/usb_gadget/g1/configs/b.1/f2
       symlink /config/usb_gadget/g1/functions/cser.nmea.1 /config/usb_gadget/g1/configs/b.1/f3
       symlink /config/usb_gadget/g1/functions/${sys.usb.rmnet.func.name}.rmnet /config/usb_gadget/g1/configs/b.1/f4
       write /config/usb_gadget/g1/UDC ${sys.usb.controller}
       setprop sys.usb.state ${sys.usb.config}
+
 on property:sys.usb.tethering=true
     write /sys/class/net/rndis0/queues/rx-0/rps_cpus ${sys.usb.rps_mask}
```
>2. 输入以下指令启动 NMEA port  
```
adb root
/* 即时生效，但不保存，重启即失效*/
adb shell setprop sys.usb.config diag,serial_cdev,serial_cdev,rmnet,adb
/* 生效并保存，重启仍有效*/
adb shell setprop persist.sys.usb.config diag,serial_cdev,serial_cdev,rmnet,adb  
```
### MSM8998_Android_N 使能
>1. 修改代码  
```
Kernel
CONFIG_USB_CONFIGFS_SERIAL=y
CONFIG_USB_U_SERIAL=y
CONFIG_USB_F_SERIAL=y

sys.usb.config=test，其中test可为自定义的属性名，无重复即可
--- a/rootdir/etc/init.msm.usb.configfs.rc
+++ b/rootdir/etc/init.msm.usb.configfs.rc
@@ -87,6 +87,27 @@ on property:sys.usb.config=diag && property:sys.usb.configfs=1
write /config/usb_gadget/g1/UDC ${sys.usb.controller}
setprop sys.usb.state ${sys.usb.config}

+on property:sys.usb.config=test && property:sys.usb.configfs=1
+ start adbd
+
+on property:sys.usb.ffs.ready=1 && property:sys.usb.config=test && property:sys.usb.configfs=1
+ write /config/usb_gadget/g1/configs/b.1/strings/0x409/configuration "Test"
+ rm /config/usb_gadget/g1/configs/b.1/f1
+ rm /config/usb_gadget/g1/configs/b.1/f2
+ rm /config/usb_gadget/g1/configs/b.1/f3
+ rm /config/usb_gadget/g1/configs/b.1/f4
+ rm /config/usb_gadget/g1/configs/b.1/f5
+ write /config/usb_gadget/g1/idVendor 0x05C6
+ write /config/usb_gadget/g1/idProduct 0x9018
+ symlink /config/usb_gadget/g1/functions/diag.diag /config/usb_gadget/g1/configs/b.1/f1
+ symlink /config/usb_gadget/g1/functions/ffs.adb /config/usb_gadget/g1/configs/b.1/f2
+ symlink /config/usb_gadget/g1/functions/cser.dun.0 /config/usb_gadget/g1/configs/b.1/f3
+ symlink /config/usb_gadget/g1/functions/gser.0 /config/usb_gadget/g1/configs/b.1/f4
+ symlink /config/usb_gadget/g1/functions/mass_storage.0 /config/usb_gadget/g1/configs/b.1/f5
+ write /config/usb_gadget/g1/UDC ${sys.usb.controller}
+ setprop sys.usb.state ${sys.usb.config}
+
+
on property:sys.usb.config=diag,serial_cdev,rmnet_gsi,adb && property:sys.usb.configfs=1
start adbd

--- a/rootdir/etc/init.qcom.usb.rc
+++ b/rootdir/etc/init.qcom.usb.rc
@@ -49,6 +49,7 @@ on boot
mkdir /config/usb_gadget/g1/functions/diag.diag
mkdir /config/usb_gadget/g1/functions/cser.dun.0
mkdir /config/usb_gadget/g1/functions/cser.nmea.1
+ mkdir /config/usb_gadget/g1/functions/gser.0
mkdir /config/usb_gadget/g1/functions/gsi.rmnet
mkdir /config/usb_gadget/g1/functions/gsi.rndis
mkdir /config/usb_gadget/g1/functions/gsi.dpl
```
>2. 输入以下指令启动NMEA
```
adb root
adb shell setprop sys.usb.config test
adb shell setprop persist.sys.usb.config test
```
### MSM8953_Android_N 使能
> 此平台，代码中已包含 NMEA 的内容，仅需要 adb 指令启动即可
```
adb root
adb shell setprop sys.usb.config diag,diag_mdm,serial_hsic,serial_tty,rmnet_hsic,mass_storage,adb
adb shell setprop persist.sys.usb.config diag,diag_mdm,serial_hsic,serial_tty,rmnet_hsic,mass_storage,adb
```
