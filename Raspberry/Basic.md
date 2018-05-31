## Raspberry Basic knowledge

Raspberry Pi，中文为 树莓派，简写为 RPi，或 RasPi，卡片式电脑，系统级芯片为 Broadcom，具有 ARM CPU及 GPU，接口有 USB2.0 、HDMI、3.5mm、百兆以太网及蓝牙、WIFI，GPIO，以SD卡启动并作为系统存储，因此不存在成砖的风险，具有高灵活性和扩展性。另，官方推荐使用 Python 语言

官方网站：https://www.raspberrypi.org

树莓派3 教程：https://www.ncnynl.com/category/Raspberry-Pi-tutorial/

硬件信息：https://elinux.org/RPi_VerifiedPeripherals

------

### 系统安装

官方推荐安装 RASPBIAN，以 Debian 为基础，另外还支持 Android、Windows 10 IOT等。

下载链接：https://www.raspberrypi.org/downloads/

有两种方式安装，官方推荐使用简易安装方式 NOOBS，以及面向高级用户的 system  image 方式。

#### NOOBS

> 1. 下载 NOOBS，并解压
>
>    有两种 NOOBS，一种为完整安装包，一种为在线安装
>
> 2. 准备SD卡
>
>    最好 8G 以上，先使用 SD Formatter 进行格式化，FAT32
>
> 3. 将所有 NOOBS 文件拷入 SD 卡中，后插入开发板卡槽
>
> 4. 接上显示器（HDMI）、鼠标、键盘，USB 5V 供电
>
> 5. 启动后，可根据向导进行系统安装
>
> 6. 安装结束后，默认登陆用户名是 pi，密码是 raspberry，若进入图形界面，可输入 startx
>
> 7. 远程登陆使用
>
>    本地安装 putty，填写 IP 地址，默认 port：22

#### System Image

> 1. 下载系统镜像，及 Etcher 工具，以及准备格式化后的 SD 卡
> 2. 使用 Etcher 把镜像写入 SD 卡
> 3. 结束后，插入开发板卡槽，插入USB电源即可开机使用

------

### Camera Sensor

官方介绍：https://www.raspberrypi.org/documentation/raspbian/applications/camera.md

树莓派官方的摄像头模块为 Pi Cam，为 CSI 接口，树莓派系统默认包含驱动及使用工具，可即插即用，进行拍照、录像及视频流传输，还可直接获取 raw 数据，参考网址：https://linux.cn/article-3650-1.html / http://www.cnblogs.com/uestc-mm/p/7587783.html  /  https://www.raspberrypi.org/documentation/usage/camera/README.md

当前还有 USB WebCam，CSI 接口可直接使用GPU，USB需要使用CPU进行压缩处理

RPi 默认已经支持较多的 USB Webcam，可从https://elinux.org/RPi_USB_Webcams 查询，包括但不限于

USB WebCam 使用参考：https://www.raspberrypi.org/documentation/usage/webcams/ / https://linux.cn/article-5312-1.html

#### Pi Cam

> 支持 FHD 1080P，可编程
>
> 1. 连接摄像头模块与开发板
>
> 2. 升级最新固件
>
>    ```
>    sudo apt-get update
>    sudo apt-get upgrade
>    ```
>
> 3. 激活摄像头模块
>
>    ```
>    sudo raspi-config
>    ```
>
>    Enable Camera 并重启
>
> 4. 拍照 （raspistill）
>
>    ```
>    raspistill -o keychain.jpg -t 2000
>    ```
>
>    以上命令为 在 2000ms 后拍摄一张照片，保存为 keychain.jpg
>
>    **raspiyuv 工具用法类似，得到未处理过的 raw 图像**
>
> 5. 录像（raspivid）
>
>    ```
>    raspivid -o keychain.h264
>    ```
>
>    按照默认配置（长度 5s，分辨率 1920 x 1080，比特率 17Mbps）拍摄，视频为未压缩（raw），且不含声音
>
>    **-t** 设置录制时间，**-w** 和 **-h** 设置分辨率
>
>    若进行播放，需要使用 gpac 包中所带的 MP4Box 应用
>
>    ```markdown
>    ## 安装 gpac
>    sudo apt-get install -y gpac
>    ## 转换为每秒 30 帧的 mp4 视频
>    MP4Box -fps 30 -add keychain.h264 keychain.mp4
>    ```

### Movidius Neural Compute Stick

英特尔 Movidius NCS 低功耗的视觉处理单元，以 USB 棒形式，可灵活扩展在其他低功耗的嵌入式系统

Raspberry Pi Support：https://ncsforum.movidius.com/discussion/118/movidius-nc-sdk-1-07-07-with-raspberry-pi-support

在 Raspberry 上运行，有两种方式：

- 完整 SDK 模式  https://developer.movidius.com/start
- 仅 API 模式，安装速度快，但无法对图形文件进行分析、验证和神经计算

#### API 模式

> 1. 在 RPI 安装 Python 依赖
>
>    ```
>    sudo apt-get install -y libusb-1.0-0-dev libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler libatlas-base-dev git automake byacc lsb-release cmake libgflags-dev libgoogle-glog-dev liblmdb-dev swig3.0 graphviz libxslt-dev libxml2-dev gfortran python3-dev python-pip python3-pip python3-setuptools python3-markdown python3-pillow python3-yaml python3-pygraphviz python3-h5py python3-nose python3-lxml python3-matplotlib python3-numpy python3-protobuf python3-dateutil python3-skimage python3-scipy python3-six python3-networkx python3-tk
>    ```
>
> 2. 下载 NCS SDK
>
>    ```markdown
>    
>    ```
>
> 3. 编译安装 NCS SDK API
>
>    ```
>    
>    ```
>
> 4. 实例工程验证
>
>    ```
>    cd ~/workspace
>    git clone https://github.com/movidius/ncappzoo
>    cd ncappzoo/apps/hello_ncs_py
>    python3 hello_ncs.py
>    ```
>
>    如果出现如下输出，则环境已经OK
>
>    ```
>    Hello NCS! Device opened normally.
>    Goodbye NCS! Device closed normally.
>    NCS device working.
>    ```
>
>    在下面进行图形识别前，还必须得到基于 GoogleNet 的图形文件，此图形文件需要在 SDK 完整工具包生成，即需要在完整安装了 SDK 的开机机器上生成，然后拷贝到 RPI
>
>    生成图形文件：https://movidius.github.io/ncsdk/tools/compile.html
>
>    ```
>    mvNCCompile deploy.prototxt -w bvlc_googlenet.caffemodel -s 12 -in input -on prob -is 224 224 -o GoogLeNet.graph
>    ```
>
> 5. 进行图形分类
>
>    https://movidius.github.io/blog/ncs-image-classifier/
>
>    深入学习使用多个相互连接的神经元层，每个层使用特定的计算机算法识别和分类特定描述符。深神经网络 DNN 将任务分解成许多简单算法的能力可与更大的描述符集一起工作
>
>    图像分类：假定整个图像只有一个对象
>
>    图像检测：可处理同一图像中的多个对象，还可识别出在图像中的位置
>
>    