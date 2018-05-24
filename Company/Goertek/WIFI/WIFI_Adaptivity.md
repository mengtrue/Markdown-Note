### WIFI Adaptivity
#### 哪里测，怎么测，什么问题

>1. 哪里测
>
>  昆山Sporton实验室认证需要测试
>
>2. 怎么测
>  连接上相应信道的WIFI热点后，测试仪器将会产生不等的干扰信号，同时检测设备与路由器之间的丢包情况(Ping)，在加入干扰后，测试仪器测试相应指标，与spec做对比，不同设备可能spec不同
>
>3. 什么问题
>  当前设备测试指标偏低，即受干扰影响比预计大，即此项测试的是WIFI相应的抗干扰指标
>  相应的，Bluetooth也有对应的Adaptivity测试


>#### Debug过程
>
>1. 分析问题原因
>  高通case：03255544，经过内部分析，WCN3990 WIFI芯片BDF文件设置默认此抗干扰值，使用对应的bdwlan.bin
>  和bdwlan.txt转换工具转换后，发现以下item：
>
>  ```markdown
>  thr_cca_etsi_ovd_A_0_0 0
>  thr_cca_pri20_A_0_0 0x1e
>  thr_cca_ext20_A_0_0 0x1e
>  thr_cca_ext40_A_0_0 0x1e
>  thr_cca_ext80_A_0_0 0x1e
>  ```
>
>  其中，thr_cca_etsi_ovd_A_0_0 to 1 to enable the override，0x1e是默认值 
>  另外，以上内容为5G Adaptivity设置，2.4G相应设置如下
>
>  ```
>  thr_cca_etsi_ovd_G_0_0 0
>  thr_cca_pri20_G_0_0 0x1e
>  thr_cca_ext20_G_0_0 0x1e
>  thr_cca_ext40_G_0_0 0x1e
>  thr_cca_ext80_G_0_0 0x1e
>  ```
>
>2. 解决过程
>
>  - 问题1：使用bdwlan.bin和bdwlan.txt转换工具转换后，未找到相应的ttr_cca属性
>
>    解决：转换工具过时，应使用最新的转换工具  
>
>  - 问题2：正确转换后，此属性未生效，即转换后，push进系统目录，测试结果同修改前一样，
>
>    并且同高通确认后，使用验证方法测试，设置值为生效
>
>    和高通确定的验证方法为
>
>  ```markdown
>  #设置为其他国家码(当为CN时，此功能受限)
>  iw reg set DE
>  athdiag --get --address 0x0930027c
>  athdiag --get --address 0x0930011c
>  #若以上两条指令可读出设置的属性值，则生效
>  ```
>
>  ​        高通更新了驱动及相关代码，如CR 2055712
>
>  - 问题3：根据fail结果增加1~1.5dB， 依然fail，继续增加，发现测试结果偏差更大
>
>    解决：尝试使用高通默认的设置值，0x1e，结果PASS
#### 总结

>1. 从RF得知，此项测试还需关闭扫描及开启节能模式，同实验室沟通及最后测试，可不必此操作  
>2. 修改bdwlan.bin，务必保证当前版本、代码、转换工具相适配，即开发中需要特别关注转换工具的更新  
>3. 所有测试，欲解决问题，必须清楚测试内容、测试方法(流程)、测试结果，如有可能，可进行修改后的验证测试(Office通过软件方法)  
>4. 修改bdwlan.bin不可完全信任RF，毕竟更不熟悉软件，且工作态度、方式、方法有待提高，最好进行把关处理
