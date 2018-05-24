### 单天线

>gEnable2x2 表示是否使能双天线，若是 0，则允许使能单个天线
>```
># These parameters are used only if gEnable2x2 is 0
># Valid values are 1,2
># Set gSetTxChainmask1x1=1 or gSetRxChainmask1x1=1 to select chain0.
># Set gSetTxChainmask1x1=2 or gSetRxChainmask1x1=2 to select chain1.
>gSetTxChainmask1x1=1
>gSetRxChainmask1x1=1
>```
### MAC Spoof
>gEnableMacAddrSpoof 表示在 probe request中填入随机的MAC地址，确保收到的设备无法确定真实的MAC
>```
># Enable or Disable Random MAC (Spoofing)
># 1=Enable, 0=Disable (default)
>gEnableMacAddrSpoof=0
>```
### Fast Roaming Support
>WIFI漫游，当存在多个同名同加密方式及密码的AP，当所连AP RSSI较弱时，自动连接至更强的RSSI
>```
># Legacy (non-ESE, non-802.11r) Fast Roaming Support
># To enable, set FastRoamEnabled=1
># To disable, set FastRoamEnabled=0
>FastRoamEnabled=1
>FastTransitionEnabled=1
>```
