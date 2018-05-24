## BLE Direct Test Mode
### BLE DTM 概况

>1. 支持BLE的设备均需要进行BLE测试，尤其是进行 non-signaling(非信令)测试，特别是产品面上市场之际，需要满足入网、当地市场准入标准，均会由具有认证资质的实验室进行测试并获取测试报告  
>2. 测试内容分为 TX 和 RX，必须有另外的可进行蓝牙BLE收发数据的测试仪器搭配测试，常见的仪器有 Rohde & Schwarz CBT & Anritsu MT8852B，当前，联想客户要求的专业BLE认证在 TUV 实验室进行，其测试仪器为 CBT，另外在昆山 Sporton 实验室测试 FCC 等认证，测试仪器为 CMW500，青岛及北京致真大厦拥有CMW500
### 高通平台BLE DTM测试方法
>根据是否使用 QRCT 分为两类，使用 QRCT 的方法 和 不适用 QRCT 的方法  
>
>**使用QRCT**：需要结合 Vector Signal Analyzer(VSA，矢量信号分析仪)，QRCT 发送 EPTM 命令到DUT，VSA从DUT收集数据，不常用  
>![BLE_DTM_QRCT_VSA](../png\BLE_DTM_QRCT_VSA.png)  
>![BLE_DTM_QRCT](../png\BLE_DTM_QRCT.png)  
>![BLE_DTM_QRCT_LE](../png\BLE_DTM_QRCT_LE.png)  
>Step 4中 Transmitter Test，调用库函数为  
>`QLIB_FTM_BT_LE_HCI_Transmitter_Test(
>HANDLE hResourceContext, unsigned char
>iTestFrequency, unsigned char
>iTestPayloadLength,
>unsigned char iTestPayload);`  
>![BLE_DTM_QRCT_LE_RX](../png\BLE_DTM_QRCT_LE_RX.png)  
>Step 2中 Receiver Test，调用库函数为  
>`QLIB_FTM_BT_LE_HCI_Receiver_Te
>st(HANDLE hResourceContext,
>unsigned char iTestFrequency);`  
>Step 3中 End Test，调用库函数为  
>`QLIB_FTM_BT_LE_HCI_Test_End(
>HANDLE hResourceContext,
>unsigned short* noOfPackets);`    
>
>**不使用QRCT**： 根据使用单一工具进行还是使用测试系统分为两类，上海TUV实验室为使用7Layers InterLab Solution，目前仅获知此一家，其余实验室或歌尔内部均使用单一测试仪器及其工具进行，使用单一工具根据所使用的测试仪器分类，大体分为三种：Anritsu MT8852B(安立蓝牙测试装置)、Rohde & Schwarz CBT(罗德与施瓦茨蓝牙测试仪，已停产)、CMW500(罗德与施瓦茨宽带无线电通信测试仪)，三种方式中设备与测试电脑连接方式相同，设备端操作相同  
>
>
>1. MT8852B  
>
>![MT8852B](../png\MT8852B.png)  
>
>
>[Anritsu MT8852B 介绍](https://www.anritsu.com/zh-CN/test-measurement/products/mt8852b) https://www.anritsu.com/zh-CN/test-measurement/products/mt8852b  
>
>
>可用于产品设计验证和生产测试，支持蓝牙基本速率(BR)、增强数据速率(EDR)和蓝牙低功耗(BLE)射频测试规范要求的传输功率、频率、调制和接收机灵敏度，通过UART、USB或USB-Adaptor HCI控制接口对测试设备进行初始化和控制，已能够支持蓝牙5.0的测试规范，支持可显著改善电池寿命的2Mbps低功耗PHY(2LE)，并支持超过500米通信范围的蓝牙长距离(BLR)测试  
>
>
>![MT8852B Setup](../png\MT8852B_setup.png)  
>
>
>2. CBT  
>![CBT](../png\CBT.png)  
>[CBT 介绍](https://www.rohde-schwarz.com.cn/product/cbt_cbt32-productstartpage_63493-7927.html) https://www.rohde-schwarz.com.cn/product/cbt_cbt32-productstartpage_63493-7927.html  
>可测试蓝牙 1.2、2.0、2.1、3.0+HS、4.0、4.1以及4.2核心协议，此仪器已停产，后代产品为 [CMW270 Wireless Connectivity Tester](https://www.rohde-schwarz.com.cn/product/cmw270-productstartpage_63493-9552.html) https://www.rohde-schwarz.com.cn/product/cmw270-productstartpage_63493-9552.html  
>![CBT Setup](../png\CBT_setup.png)  
>3. CMW500  
>![CMW500](../png\CMW500.png)  
>当前只支持串口控制设备，[CMW500 介绍](https://www.rohde-schwarz.com.cn/product/cmw500-productstartpage_63493-10341.html) https://www.rohde-schwarz.com.cn/product/cmw500-productstartpage_63493-10341.html  
>CBT 和 CMW500详细测试步骤另有文档专门介绍  
>
>以上三种方式，设备端均需要如下操作 (**保证测试设备 NMEA port 正常**)  
>1)关闭蓝牙  
>2)进入蓝牙测试模式  
>```
>adb root
>adb shell
>ftmdaemon -n -dd
>```
>3)打开高通BLE控制软件，因为测试仪器均是通过HCI Command控制测试设备，高通BLE控制软件(**QC.BluetoothLE_DirectMode.exe**)可发送相应的HCI  Command给测试设备  
>![QC.BluetoothLE_DirectMode](../png\QC.BluetoothLE_DirectMode.png)  
>![QC.BluetoothLE_DirectMode Setup](../png\QC.BluetoothLE_DirectMode_setup.png)  
>上图中，COM应选择对应的测试仪器连接测试电脑的RS232扣，Handshake选择 RequestToSendXOnXOff
