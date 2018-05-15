这里记录 Bluetooth debug 相关

------

## 高通芯片

### Firmware build info

> **WCN36X0**
>
> - 使用 iwpriv 工具
>
>   ```
>   adb root && adb shell && iwpriv wlan0 version 
>   ```
>
> - 开关 WIFI 抓取 kmsg，查看 firmware version

> **WCN39X0 and QCA61X4**
>
> - ```
>   adb shell cat /data/misc/bluetooth/bt_fw_version.txt
>   ```
>
> - 打开蓝牙，抓取 logcat，查看 firmware version

### Logcat

```
adb logcat -b all -v threadtime > logcat.txt
```

> Enable Bluetooth stack (Bluedroid/Fluoride) verbose info
>
> - Basic stack info, change all "TRC_xx" to **6** in "/etc/bluetooth/bt_stack.conf"
>
>   关闭并再次打开蓝牙，生效
>
> - BT audio debug info
>
>   add "TRC_LATENCY_AUDIO=6" in "/etc/bluetooth/bt_stack.conf"
>
> - Inquiry debug logs           /system/bt/stack/btm/btm_inq.c
>
>   ```
>   #define BTM_INQ_DEBUG TRUE
>   ```
>
> - PAN debug logs
>
>   add "TRC_BNEP=6" and "TRC_PAN=6" in "/etc/bluetooth/bt_stack.conf"
>
> - HID/HOGP debug logs
>
>   to understand whether BT report the key code properly, add logs in **bta_hh_co_write()** of system/bt/btif/co/bta_hh_co.c to print the reports
>
>   And make sure **BTA_HH_DEBUG** is defined as TRUE
>
> - LE debug logs
>
>   system/bt/include/bt_target.h
>
>   ```
>   #define SMP_DEBUG TRUE
>   #define SMP_DEBUG_VERBOSE TRUE
>   ```
>
>   system/bt/bta/include/bta_gatt_api.h
>
>   ```
>   #define BTA_GATT_DEBUG TRUE
>   ```
>
> - SAP debug logs
>
>   add "TRC_SAP=6" in "/etc/bluetooth/bt_stack.conf"
>
> - HFP debug logs
>
>   enable **BTA_AG_RESULT_DEBUG, BTA_AG_DEBUG, BTA_AG_SCO_DEBUG** in external/bluetooth/bluedroid/bta/ag/bta_ag_main.c
>
> - A2DP debug logs
>
>   define **BTA_AV_DEBUG** as true in external/bluetooth/bluedroid/bta/av/bta_av_int.h

> Bluetooth Application/Framework logs
>
> - Settings (Turning on/off, Searching, Pairing)
>
>   define DBG/VDBG or V/D as true in the files below
>
>   KK, L release:
>
>   ```
>   frameworks/base/core/java/android/bluetooth/BluetoothAdapter.java
>   frameworks/base/services/java/com/android/server/BluetoothManagerService.java
>   packages/apps/Settings/src/com/android/settings/bluetooth/
>   	LocalBluetoothAdapter.java
>   	CachedBluetoothDevice.java
>   	LocalBluetoothManager.java
>   	BluetoothEventManager.java
>   	LocalBluetoothProfileManager.java
>   	BluetoothDevicePreference.java
>   		getConnectionSummary(): print "cachedDevice", "profile", "connectionStatus"
>   	Utils.java
>   packages/apps/Bluetooth/src/com/android/bluetooth/btservice/
>   	AdapterProperties.java
>   	BondStateMachine.java
>   	AdapterService.java
>   	AdapterState.java
>   	AdapterApp.java
>   ```
>
>   M, N release:
>
>   ```
>   frameworks/base/core/java/android/bluetooth/BluetoothAdapter.java
>   frameworks/base/services/core/java/com/android/server/BluetoothManagerService.java
>   frameworks/base/packages/SettingsLib/src/com/android/settingslib/bluetooth/
>   	Utils.java
>   	CachedBluetoothDevice.java
>   	LocalBluetoothProfileManager.java
>   frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/policy/BluetoothControllerImpl.java
>   packages/apps/Bluetooth/src/com/android/bluetooth/btservice/
>   	AdapterState.java
>   	AdapterProperties.java
>   	AdapterService.java
>   	ProfileService.java
>   	RemoteDevices.java
>   packages/apps/Bluetooth/jni/com_android_bluetooth_btservice_AdapterService.cpp
>   ```
>
> - SPP/Bluetooth Socket
>
>   define VDBG as true in the files below
>
>   ```
>   frameworks/base/core/java/android/bluetooth/BluetoothInputStream.java
>   frameworks/base/core/java/android/bluetooth/BluetoothOutputStream.java
>   frameworks/base/core/java/android/bluetooth/BluetoothSocket.java
>   ```
>
>   to dump data via SPP link:
>
>   ```
>   import android.util.Log;
>   private static final String TAG = "BluetoothInputStream";
>   public int read(byte[] b, int offset, int length) throws IOException {
>   	if (b == null) {
>   		throw new NullPointerException("byte array is null");
>   	}
>   	if ((offset | length) < 0 || length > b.length - offset) {
>   		throw new ArrayIndexOutOfBoundsException("invalid offset or length");
>   	}
>   	//return mSocket.read(b, offset, length);
>   	
>   	Log.d(TAG, "read: begin>>>>");
>   	int ret = mSocket.read(b, offset, length);
>   	Log.d(TAG, "read: end>>>>");
>   	Log.d(TAG, "read: b = ");
>   	for (int i : b) System.out.println(i);
>   	Log.d(TAG, "read: offset = " + offset);
>   	Log.d(TAG, "read: length = " + length);
>   	return ret;
>   }
>   ```
>
>   ```
>   private static final String TAG = "BluetoothOutputStream";
>   public void write(byte[] b, int offset, int count) throws IOException {
>   	if (b == null) {
>   		throw new NullPointerException("buffer is null");
>   	}
>   	if ((offset | count) < 0 || count > b.length - offset) {
>   		throw new IndexOutOfBoundsException("invalid offset or length");
>   	}
>   	Log.d(TAG, "write: b = ");
>   	for (int i : b) System.out.println(i);
>   	Log.d(TAG, "write: offset = " + offset);
>   	Log.d(TAG, "write: count = " + count);
>   	Log.d(TAG, "write: begin<<<<");
>   	mSocket.write(b, offset, count);
>   	Log.d(TAG, "write: end<<<<");
>   }
>   ```
>
> - FTP
>
>   - define DEBUG/VERBOSE as true
>
>     ```
>     vendor/qcom/opensource/bluetooth/src/org/codeaurora/bluetooth/ftp/
>     	BluetoothFtpObexServer.java
>     	BluetoothFtpService.java
>     ```
>
>   - ```
>     setprop BluetoothFtp VERBOSE
>     ```
>
>     重新打开蓝牙
>
> - OPP
>
>   - define DEBUG/VERBOSE as true
>
>     ```
>     packages/apps/Bluetooth/src/com/android/bluetooth/opp/
>     	BluetoothOppService.java
>     	BluetoothOppRfcommListener.java
>     	BluetoothOppObexServerSession.java
>     	BluetoothOppUtility.java
>     	BluetoothOppObexClientSession.java
>     	Constants.java
>     frameworks/base/obex/javax/obex/ServerSession.java
>     frameworks/base/btobex/javax/btobex/
>     	ServerOperation.java
>     	ClientOperation.java
>     	ObexHelper.java
>     ```
>
>   - ```
>     setprop BluetoothOpp VERBOSE
>     ```
>
> - PBAP
>
>   - define DEBUG/VERBOSE or DBG/VDBG as true
>
>     ```
>     packages/apps/Bluetooth/src/com/android/bluetooth/pbap/
>     	BluetoothPbapService.java
>     	BluetoothPbapObexServer.java
>     	BluetoothPbapAuthenticator.java
>     	BluetoothPbapRfcommTransport.java
>     	BluetoothPbapReceiver.java
>     frameworks/base/core/java/android/bluetooth/BluetoothPbap.java
>     packages/apps/Settings/src/com/android/settings/bluetooth/
>     	BluetoothPermissionActivity.java
>     	BluetoothPermissionRequest.java
>     	Utils.java
>     ```
>
>   - ```
>     setprop BluetoothPbap VERBOSE
>     ```
>
> - MAP
>
>   - define DEBUG/VERBOSE as true
>
>     ```
>     packages/apps/Bluetooth/src/com/android/bluetooth/map/BluetoothMapService.java
>     vendor/qcom/opensource/bluetooth/src/org/codeaurora/bluetooth/map/
>     	BluetoothMasService.java
>     	MapUtils/EmailUtils.java
>     	BluetoothMasAppEmail.java
>     	BluetoothMasObexServer.java
>     	BluetoothMns.java
>     	BluetoothMasReceiver.java
>     ```
>
>   - ```
>     setprop BluetoothMap VERBOSE
>     ```
>
> - HFP/ HFP-C
>
>   - define DEBUG/VERBOSE or DBG/VDBG as true
>
>     ```
>     packages/apps/Bluetooth/src/com/android/bluetooth/hfp/
>     	HeadsetService.java
>     	HeadsetStateMachine.java
>     	AtPhonebook.java
>     packages/apps/Bluetooth/jni/com_android_bluetooth_hfp.cpp
>     packages/apps/Bluetooth/src/com/android/bluetooth/hfpclient/
>     	HandsfreeClientStateMachine.java
>     	HandsfreeClientService.java
>     packages/apps/Settings/src/com/android/settings/bluetooth/HeadsetProfile.java
>     packages/apps/Phone/src/com/android/phone/BluetoothPhoneService.java
>     frameworks/base/core/java/android/bluetooth/
>     	BluetoothHeadset.java
>     	BluetoothHandsfreeClient.java
>     ```
>
>   - ```
>     setprop Handsfree VERBOSE
>     ```
>
> - A2DP (SRC  SNK)
>
>   define DEBUG/VERBOSE or DBG/VDBG as true
>
>   ```
>   packages/apps/Bluetooth/src/com/android/bluetooth/a2dp/A2dpStateMachine.java
>   packages/apps/Bluetooth/jni/com_android_bluetooth_a2dp.cpp
>   packages/apps/Settings/src/com/android/settings/bluetooth/A2dpProfile.java
>   frameworks/base/core/java/android/bluetooth/BluetoothHeadset.java
>   frameworks/base/core/java/android/bluetooth/BluetoothA2dp.java
>   ```
>
> - AVRCP
>
>   define DEBUG/VERBOSE or DBG/VDBG as true
>
>   ```
>   packages/apps/Bluetooth/src/com/android/bluetooth/avrcp/
>   	Avrcp.java
>   	AvrcpControllerService.java
>   packages/apps/Bluetooth/jni/com_android_bluetooth_avrcp.cpp
>   frameworks/base/media/java/android/media/RemoteControlClient.java
>   frameworks/base/core/java/android/bluetooth/BluetoothAvrcpController.java
>   ```
>
> - SAP
>
>   M, N and later release, the SAP is from Google, define DEBUG/VERBOSE or V as true
>
>   ```
>   packages/apps/Bluetooth/src/com/android/bluetooth/sap/
>   	SapService.java
>   	SapMessage.java ← createSolicited()
>   	SapRilReceiver.java
>   	SapServer.java
>   hardware/ril/libril/
>   	RilSapSocket.cpp
>   	ril_event.cpp
>   	ril.cpp
>   vendor/qcom/proprietary/qcril/qcril_qmi/qcril_log.c
>   ```
>
>   或 
>
>   ```
>   setprop BluetoothSap VERBOSE
>   ```
>
> - DUN
>
>   open logs below:   Redefine port_log_dflt/port_log_err/port_log_high/port_log_low to LOGE
>
>   ```
>   vendor/qcom/proprietary/bt/dun/port-bridge/src/portbridge_common.h
>   ```
>
>   define DEBUG/VERBOSE or DBG/VDBG as true
>
>   ```
>   vendor/qcom/opensource/bluetooth/src/org/codeaurora/bluetooth/dun/
>   	BluetoothDunReceiver.java
>   	BluetoothDunPermissionActivity.java
>   	BluetoothDunService.java
>   frameworks/base/core/java/android/bluetooth/
>   	BluetoothDun.java
>   ```
>
> - PAN
>
>   define DEBUG/VERBOSE or DBG/VDBG as true
>
>   ```
>   packages/apps/Bluetooth/src/com/android/bluetooth/pan/PanService.java
>   packages/apps/Bluetooth/jni/com_android_bluetooth_pan.cpp
>   packages/apps/Settings/src/com/android/settings/bluetooth/PanProfile.java
>   packages/apps/Settings/src/com/android/settings/WirelessSettings.java
>   frameworks/base/core/java/android/bluetooth/
>   	BluetoothPan.java
>   	BluetoothTetheringDataTracker.java
>   ```
>
>   open Android Data/Network related logs below
>
>   ```
>   frameworks/base/services/java/com/android/server/
>   	NetworkManagementService.java ("NetdConnector")
>   	ConnectivityService.java ("ConnectivityService")
>   frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/policy/
>   	NetworkControllerImpl.java ← Enable "CHATTY"
>   system/netd/TetherController.cpp ("TetherController")
>   external/dnsmasq/("dnsmasq")
>   ```
>
> - HID
>
>   define DEBUG/VERBOSE or DBG/VDBG as true
>
>   ```
>   packages/apps/Bluetooth/src/com/android/bluetooth/hid/HidService.java
>   packages/apps/Bluetooth/jni/com_android_bluetooth_hid.cpp
>   frameworks/base/core/java/android/bluetooth/BluetoothInputDevice.java
>   ```
>
>   open Android input subsystem related logs below
>
>   ```
>   frameworks/native/services/inputflinger/
>   	EventHub.cpp
>   	InputReader.cpp
>   (Enable DEBUG_RAW_EVENTS and other macros)
>   ```
>
> - BLE
>
>   define DEBUG/VERBOSE or DBG/VDBG as true
>
>   ```
>   packages/apps/Bluetooth/src/com/android/bluetooth/btservice/
>   	AdapterService.java
>   	AdapterProperties.java
>   packages/apps/Bluetooth/src/com/android/bluetooth/gatt/
>   	GattService.java
>   	GattServiceConfig.java
>   frameworks/base/core/java/android/bluetooth/
>   	BluetoothGatt.java
>   	BluetoothAdapter.java
>   	BluetoothManager.java
>   frameworks/base/core/java/android/bluetooth/le/BluetoothLeScanner.java
>   ```
>
> - hci_qcomm_init logs
>
>   Bluetooth turning on failure for fault during firmware downloading
>
>   > 1. vendor/qcom/proprietary/bt/hci_qcomm_init/bthci_qcomm_linux.cpp
>   >
>   >    ```
>   >    int verbose = 3;
>   >    ```
>   >
>   > 2. by default hci_qcomm_init use fprintf() to output logs to device "stderr"
>   >
>   > 3. modify DEBUGMSG/BTHCI_QCOMM_ERROR as 2 in vendor/qcom/proprietary/bt/hci_qcomm_init/bthci_qcomm_pfal.h
>   >
>   > 4. remove all "printf" and use "sprintf" instead like (2) to write logs into a file
>
>   > 1. device/qcom/common/rootdir/etc/init.qcom.rc
>   >
>   >    ```
>   >    -- service hciattach /system/bin/sh /system/etc/init.qcom.bt.sh
>   >    ++ service hciattach /system/bin/logwrapper /system/bin/sh /system/etc/init.qcom.bt.sh
>   >    ```
>   >
>   > 2. vendor/qcom/proprietary/bt/hci_qcomm_init/bthci_qcomm_linux.cpp
>   >
>   >    ```
>   >    int verbose = 3;
>   >    ```
>
> - BTsnoop
>
>   设置正确的时间
>
>   修改 /etc/bluetooth/bt_stack.conf
>
>   ```
>   BtSnoopLogOutput=true
>   BtSnoopSaveLog=true
>   BtSnoopExtDump=false
>   BtSnoopFileName=/sdcard/btsnoop_hci.cfa
>   ```
>
> - QCA61X4 QXDM logs
>
>   ```
>   setprop enablebtsoclog true
>   setprop enablesnooplog true
>   setprop persist.service.bdroid.soclog true
>   setprop persist.service.bdroid.snooplog true
>   ```
>
>   QXDM logs 保存在设备上
>
>   ```
>   diag_mdlog -f /sdcard/diag_logs/diag.cfg -b -c &
>   diag_mdlog -k
>   ```
>
> - BTC/CxM logs
>
>   关闭 WIFI，输入以下指令再打开 WIFI
>
>   ```
>   iwpriv wlan0 dump 13 6 6 1
>   ```
>
> - Kmsg
>
>   ```
>   adb cat /proc/kmsg
>   ```
>
> - Bluetooth logs
>
>   enable pr_debug() or dev_dbg()
>
>   ```
>   echo -n "file bluetooth-power.c +p" ">" /sys/kernel/debug/dynamic_debug/control
>   ```
>
> - UART logs
>
>   enable pr_debug() or dev_dbg()
>
>   ```
>   echo -n "file msm_serial_hs.c +p" ">" /sys/kernel/debug/dynamic_debug/control
>   ```

### Bluetooth stack related *.so with symbols

```
out\target\product\<target>\symbols\system\lib\hw\bluetooth.default.so
out\target\product\<target>\symbols\system\lib\libbluetooth_jni.so
out\target\product\<target>\symbols\system\vendor\lib\libbt-vendor.so
```

### dumpsys

>Bluetooth info
>
>```
>dumpsys bluetooth_manager
>```
>
>Analyze application memory usage over time
>
>```
>dumpsys procstats
>dumpsys procstats com.android.bluetooth
>dumpsys procstats --hours 3
>```
>
>Analyze application memory usage at a particular snapshot in time
>
>```
>dumpsys meminfo
>dumpsys meminfo com.android.bluetooth
>```
>
>showmap
>
>```
>showmap "pid-of-com.android.bluetooth"
>```
>
>