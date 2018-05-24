## BQB概况及测试方法

#### BQB概况  
> BQB(Bluetooth Qualified Body)，即通常所说的蓝牙认证，若产品具有蓝牙功能并且在外观上标明蓝牙标志，必须通过BQB认证，认证PASS后，还需要在Bluetooth SIG(蓝牙兴趣小组/蓝牙联盟)购买并取得QDID，能够在授权产品额官方数据库中列出对应产品，即常见的具有蓝牙功能的产品都必须取得BQB证书，尤其是销往国际市场。
> 另外，取得BQB证书，必须成为Bluetooth SIG成员，可申请为团体或个人成员，个人成员免费，可获取相关的资讯、蓝牙常识等，获取BQB证书必须付费购买，并非任何测试实验室的认证报告均可以获取Bluetooth SIG的认可，只有取得认证资质的公司才可以出具BQB认证报告，当前中国大陆具有资质的单位如下：
> [Bluetooth SIG List of BQTFs](https://www.bluetooth.org/apps/qualification/bqtf.aspx)  
> ![Bluetooth SIG List of BQTFs](../png\BQTFs.png)  
> 当前客户产品均通过图中 TUV Rheinland(上海)进行BQB认证
> ![TUV](../png\BQB_TUV.png)  
> #### TUV认证过程  
> 1. 准备
>   开始验证前的必要条件为产品硬件/软件基本稳定，RF需要完成必须的OTA报告，软件应具备基本功能，并且能够开启进入相应测试模式(一般使用某个平台解决方案，如高通等，类似的RF软件测试部分，均已完善，软件可稍早验证此功能是否正常运行，相应RF测试模式，另有文档专门介绍) 
>   准备1：BQB认证文档，此文档面向TUV实验室，主要是告知产品的基本信息，特别是本产品蓝牙相关信息，如支持的协议版本(4.2?5.0)，支持的profile(A2DP?HFP)，方便实验室依据此文档生成初步的测试要求，最后的BQB证书也会根据此文档生成，最后还有相应蓝牙芯片的DID，如蓝牙分为Host/Controller
>   两部分，需要提供两部分的DID，获取QDID的方法，一是访问Bluetooth SIG网站，搜索使用的蓝牙芯片对应的DID([搜索DID](https://launchstudio.bluetooth.com/listings/search))，二是询问芯片厂商，直接提交case或者当下已确定的KBA(kba-170531225907_qualcomm_qdid_lost_complete_for_android)  
> 2. 问题
>   提交实验室所需文档后，实验室根据文档生成所必须的测试项即开始测试，测试项目基本划分为两大类，Core
>   测试项目包括 RF、BB、LM、L2CAP、SDP和GAP，以及其他扩展测试，包括Profile、Protocol测试和Profile IOP互通性测试。
>   出现问题为实验室无法展开测试：
>   1)实验室要求设备开启并进入测试模式
>   蓝牙测试模式测试前已确保无问题，但实验室测试系统仍无法运行
>   2)RF同事跟进实验室状态
>   ①测试设备需要串口与测试电脑相连，通过工具进行测试
>   ②之前项目(WCN3660)同样方法进行
>   ③测试模式指令使用蓝牙测试模式指令即可
>   ④测试仪器是CBT，另上海Sporton实验室也需要测BLE，测试仪器是CMW500
>   通过以上信息，查阅高通相关文档以及提交高通case，共获取以下有用文档
>   `kba-170629211501_1_how_to_do_bluetooth_signaling_and_non-signaling_test.pdf`
>   `如何用CMW500测试Qualcomm芯片的蓝牙40功能.pdf`
>   `80-ya512-13_e_wcn39xx_bluetooth,_fm,_and_ant+_feature_specification_guide.pdf` 
>   `80-wl300-20_a_wcn3660_bluetooth_low_energy_(ble)_direct_test_mode.pdf` 
>   `80-wl400-19_a_application_note__bluetooth_low_energy_direct_test_mode.pdf` 
>   高通case：03265584 + 03278890 + 03278784 + 03278590
>   依据以上项目中得到的信息，提供了SOP，但根据RF反馈，仍然无法正常测试，并得到更进一步信息，确定使用仪器为CBT，要求进入BLE测试 模式，依据此要求，再次根据测试仪器及BLE模式，更新SOP，得到反馈，一是不需要单独的指令，而是非直接使用的CBT，而是使用的一个测试系统
>   通过与实验室直接沟通，明确测试设备、测试仪器以及测试电脑的连接，提交case：03289193 + 03287208  
> 3. 解决
>   1)通过高通确认，使用CBT测试BLE，需要开启设备的NMEA port，此port在Android N时，通过指令即可开启，但MSM8998以及WCN3990芯片又有所不同，相同指令不适合，特提交USB case要求启用NMEA port
>   2)根据USB patch，编译版本，发现并非NMEA port而是GPS port，实验室使用此版本进行测试，无果，后发现此GPS port并不能代替NMEA port，通过进一步排查，MSM8998 Android N下还应添加相应的kernel patch；同时，确定实验室测试系统为 7Layers InterLab Solution，高通同实验室进行直接沟通  
>   3)USB方面进一步debug，最终确定Android O下NMEA port的正确完整patch，具体通信方式需要联系GPS；另一方面由于使用的 7Layers，高通默认的CBT测试方案也不能完全适合，同时，高通发现测试系统其实可以不同过NMEA进行通信，使用adb通信即可，要求实验室使用adb指令，实验室始终无法验证adb指令在测试系统中的运行
>   4)实验室 确定了通过手动输入adb指令可以执行，但测试系统始终无法执行，高通共享了内部的测试截图及结果，也下载了相应的测试软件：Automation Explorer - InterLab Bluetooth RF Test Solution，手动测试无问题，将详细测试步骤及截图共享，最终通过远程操作，实验室可正常使用adb指令，同时将测试过程中的指令进一步准确化，实验室无法运行的原因应当是实验室环境电脑问题
>   最终实验室通过在测试系统中使用adb指令使设备进入BLE测试模式及开启TX/RX对应测试，得以
>   正常进行测试，测试开始后，并无问题，很快，测试PASS  
> #### TUV InterLab Bluetooth RF Test Solution 测试设置及方法  
> 1. 获取ADB工具，手动执行测试，保证adb执行OK及测试设备可以执行相应的指令  
> 2. 测试系统选择 adb 作为 interface，而非RS232  
> 3. 根据测试步骤输入相应的adb指令，包括send/receive  
> 4. 开始测试，控制测试仪器进行  
> 5. 实际测试需要测试多个蓝牙信道，每个频道频宽为 2MHz，共40个频道，始于2402MHz，终于2480MHz  
>
> ![TUV_BQB](../png\TUV_BQB.png)  
>
> 设备端：  
> ```
> adb root
> adb shell
> wdsdaemon -su
> ```
> 测试系统：  
> ```
> Send Command:
> HCI Reset: btconfig rawcmd 0x03 0x0003
> LE Tx test: btconfig rawcmd 0x08 0x001E <F,H> <L,H> <P,H>
> LE Rx test: btconfig rawcmd 0x08 0x001D <F,H>
> Test End: btconfig rawcmd 0x08 0x001F
> 
> Receive:
> LE Reset: 0e 04 01 03 0c 00
> LE Tx Test: 0e 04 01 1e 20 00
> LE Rx Test: 0e 04 01 1d 20 0c
> LE Test end: 0e 06 01 1f 20 00 <RESULT,H,L,#>
> ```
#### 总结
>1. 必须尽可能早的了解熟知问题发生的具体细节，如实验室所用仪器、所用测试工具，如有可能，需直接与相应实验室沟通  
>2. 尽可能多的了解掌握产品所必须的测试，测试内容，测试方法，可以提交高通case，或者查找相应文档  
>3. 高通平台测试方面较为成熟，但可能某个平台或者某款芯片需要做改动，如提交case，必须明确足够信息以提高效率
