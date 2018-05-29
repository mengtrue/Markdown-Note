## Bluetooth 概念理论

Bluetooth 当前已经发展为 BT 5.0，于2016由蓝牙技术联盟 SIG 提出，针对低功耗设备速度有相应提升和优化，并结合 WIFI 可对室内定位进行辅助，提高传输速度。

所使用的频段在 2.4GHz 的 ISM Band（全球范围内无需取得执照），2400~2483.5 MHz，信道间隔 1MHz

使用 *跳频* 技术，将传输的数据分割成数据包，通过 79 个指定的蓝牙频道分别传输数据包；在连接状态的子状态 synchronization train 和 synchronization scan 每秒 1600次，在 inquiry 和 page 子状态时，每秒 3200次

==主要功能==

> 针对低功耗设备，有更广的覆盖范围和相较现在 4倍的速度提升
>
> 结合 WIFI 可实现精度小于 1米的室内定位
>
> 传输速度上限为 24Mbps （3MB/s），为之前 BT4.2 LE 版本两倍
>
> 传输级别达到无损级别
>
> 有效工作距离可达 300米，是之前 BT4.2 LE 版本 4倍

### 概述

#### 连接方式：

![Bluetooth_connect_type](..\png\Bluetooth\connect_type.png)

> 由上可知，连接分为 master/slave，由当前进行的功能以区分，特别是当下的 TWS 耳机，TWS 耳机分为左右耳，左右耳之间连接以进行音频的转发，一个作为 master，一个 slave，但当 master 连接手机，此 master 又会作为 slave，即同一个设备，可同时作为 master / slave，上图可知，一个 master 可连接多个slave，多个设备可组网，根据实际应用看，并不存在一个slave连接多个master，若存在，均是如 TWS 耳机一样同时作为两个 role，并不会只做 slave 连接多个 master
>
> 一个 master 最多可和 **==7==** 个 slave 保持同步并通信，最多可和 **==超过200==** 个 slave 保持同步不通信

#### 时钟

> 每一个蓝牙设备有一个内部系统时钟，用来决定收发器的时序和跳频，该时钟不会被调整或关掉，可作为一个 28位计数器使用，其 LSB(Least Significant Bit)计数周期是 312.5us，即时钟频率为 3.2kHz
>
> ![bluetooth_clock](..\png\Bluetooth\bluetooth_clock.png)
>
> 有 4 个重要周期，**312.5us, 625us, 1.25ms, 1.28s**

#### 地址

> 每个蓝牙设备都应该有唯一的 48bit 设备地址（BD_ADDR）
>
> ![bd_addr](..\png\bluetooth\bd_addr.png)
>
> LAP: Lower Address Part, UAP: Upper Address Part, NAP: Non-significant Address Part
>
> LAP 有 64个保留地址（0x9E8B00-0x9E8B3F），通常用的 inquiry LAP 是 0x9E8B33
>
> 共有两种类型的蓝牙地址：**Public Bluetooth Address** 和 **Random Bluetooth Address**，均为 48bits，所有蓝牙设备至少包含其中一种，上图所示为 public bluetooth address，BR/EDR 和 LE 设备均可使用。Random Bluetooth Address 仅 LE 设备使用，其包含两个子类型：**Static address** 和 **Private address**
>
> ![static_address](..\png\bluetooth\static_address.png)
>
> 在 power cycle 后，static address 可进行改变，但重连功能将会受限
>
> private address 又分为 **Non-resolvable private assress** 和 **Resolvable private address**
>
> ![non-resolvable](../png/bluetooth/non-resolvable_private_address.png)
>
> 可定时更新，更新周期由 GAP规定，实际产品并不常见
>
> ![resolvable_address](../png/bluetooth/resolvable_private_address.png)
>
> 通过一个随机数和 IRK（Identity resolving key）生成，只能被拥有相同 IRK 的设备扫描到，可防止被未知设备扫描和追踪，也可定时更新，不能单独使用，需要配合其他地址类型一起使用

#### 物理信道

> 蓝牙设备通过时分复用以支持多个操作的同时进行，共定义 五种 物理信道
>
> | Physical Channel                      | Work                             |
> | ------------------------------------- | -------------------------------- |
> | basic piconet physical channel        | piconet中已连接设备通信          |
> | adapted piconet physical channel      | piconet中已连接设备通信          |
> | page scan physical channel            | 用于连接设备                     |
> | inquiry scan physical channel         | 用于发现远端设备                 |
> | synchronization scan physical channel | 获取广播物理链路的时间和频率信息 |
>
> <!--piconet: 小范围内装有蓝牙单元的各种电器组成的微型网络，俗称微微网-->

#### 物理链路

> 两个设备间的基带连接，总是与某个特定的物理信道相关联

#### 逻辑传输层

> 在 master 和 slave 之间，可建立不同类型的逻辑传输层
>
> - SCO：Synchronous connection-oriented logical transport，用于点对点传输，通常用于有时间限制的数据（如语音和同步数据），master 通过定期预留时缝（reserved slots）维护
> - eSCO：Extended synchronous connection-oriented logical transport，比 SCO 多了一个重传
> - ACL：Asynchronous connection-oriented logical transport，用于点对点传输，但无预留时缝，master可在任意 slot建立和 slave的连接
> - ASB：Active slave Broadcast logical transport，用于 master 和 active slave 通信
> - PSB：Parked slave Broadcast logical transport，用于 master 和 parked slave 通信
> - CSB：Connectionless slave Broadcast logical transport，用于 master 发送 profile 广播

### 设备发现与同步

#### 简介

> 蓝牙设备在建立连接以前，通过在固定的一个频段内选择跳频频率或由被查询的设备地址决定，迅速交换握手信息时间和地址，快速取得设备的时间和频率同步；建立连接后，设备双方根据信道跳变序列改变频率，使跳频频率呈现随机特性

#### 状态

> 状态转换图如下：
>
> ![bluetooth_state](../png/bluetooth/state.png)
>
> 1. 查询/查询扫描 （Inquiry / Inquiry Scan）
>
>    **Inquiry** ：扫描周围的设备，处于 Inquiry Scan 的设备可回应查询，再经过必要的协商，可进行连接；此状态下，会重复发送 Inquiry 的 msg，但无任何和 source 相关的信息，有可能包含哪一类的设备应当响应这个 msg；若想搜索到所有频点的设备，需要保证 Inquiry 的最短时间是 **==10.24s==** ；每隔 **312.5us** 选择一个新的频率发送查询，被查询设备每隔 **1.28s** 选择一次新的监听频率，使用通用查询接入码作为查询地址，产生32个查询跳变序列均匀分布在 **79** 个频率信道上
>
>    **Inquiry scan** ：可被发现，可但非必须对 Inquiry 的 msg 进行响应
>
>    Inquiry 后，不需要进入 Page 就可连接
>
> 2. 寻呼/寻呼扫描（Page / Page Scan）
>
>    **Page** ：进行连接/激活对应的 slave 的操作称为 page，每隔 **312.5us** 选择一个新的频率发送寻呼，在寻呼扫描时，被寻呼设备每隔 **1.28s** 选择一个新的监听频率，使用被寻呼设备地址的 **低28个比特** 产生的寻呼跳变序列，各个频点均居分布在 2.4G 的 79 个频率信道上
>
>    **Page Scan** ：和 page 状态对应，等待被 page 的 slave 所处的状态
>
> 3. 响应（Response）
>
>    **Inquiry response** ：在 Inquiry scan 的设备收到 Inquiry 的 msg 后，发送 Inquiry response 的 msg，之后进入到 Inquiry response 状态
>
>    **Slave response** ：在 Page 过程中，slave 收到 master 的 page msg，回应对应的 page response msg，同时进入到 slave response 状态
>
>    **Master response** ：master 收到 slave response msg 后，进入到 master response 状态，同时发送一个 [^ FHS] 的 packet
>
>    [^ FHS]: 表明蓝牙设备地址和发送方时钟的特殊控制数据包，用于寻呼主单元响应、查询响应和跳频同步，占据一个时隙
>
> 4. 寻呼连接完整过程
>
>    - 一个设备（源）寻呼另一个设备（目的），源处于 Page
>    - 目的设备接收到该寻呼，目的处于 Page scan
>    - 目的设备发送对源设备的回复，处于 slave response
>    - 源设备发送 FHS 包到目的设备，处于 master response
>    - 目的设备发送第二个回复给源设备，处于 slave response
>    - 目的和源设备切换并采用源信道的参数
>
> 5. 连接状态（Connection）
>
>    通信双方每隔 **625us** 改变一个频率，使用主设备地址的最低28位有效位，产生的信道跳变序列周期非常长，而且 79 个跳变序列在任何的一小段时间内都是接近均匀分布
>
> 6. 暂停状态（Park）
>
>    最节能状态，放弃 MAC 地址，偶尔收听 master 的消息并恢复同步、检查广播消息

#### 工作模式

> 连接状态下，分为 **四种** 工作模式
>
> ![connection_state](../png/bluetooth/connection_state.png)
>
> 1. Active Mode
>
>    所有从设备都可和主设备通信，所有的通信都由主设备主导，从设备在 主设备->从设备 时隙上监听数据包，如果一个从设备没有被寻址，将等待下一个数据传输；从设备可从主设备传输的包头获取传输占用的时隙，在此期间没有被寻址的设备将会等待传输时隙
>
> 2. Sniff Mode
>
>    从设备不需要时刻监听主设备发送的数据包，可降低设备功耗，主设备将每隔 T~sniff~ 向从设备发送数据包，所以从设备每隔 T~sniff~ 监听即可；只能应用于异步传输
>
> 3. Hold Mode
>
>    从机和主机协商一个保持时间，在此期间从设备进入低功耗模式，但仍然保持 LT-ADDR
>
> 4. Connectionless Slave Broadcast Mode
>
>    传输特性广播数据