## Raspberry Basic knowledge

Raspberry Pi，中文为 树莓派，简写为 RPi，或 RasPi，卡片式电脑，系统级芯片为 Broadcom，具有 ARM CPU及 GPU，接口有 USB2.0 、HDMI、3.5mm、百兆以太网及蓝牙、WIFI，GPIO，以SD卡启动并作为系统存储，因此不存在成砖的风险，具有高灵活性和扩展性。另，官方推荐使用 Python 语言

官方网站：https://www.raspberrypi.org

树莓派3 教程：https://www.ncnynl.com/category/Raspberry-Pi-tutorial/

硬件信息：https://elinux.org/RPi_VerifiedPeripherals

本地 PC 和 树莓派之间可通过 SSH 进行远程访问，以及使用 SFTP 进行文件共享，可使用 Filezilla 进行访问，仅需要在主机中输入 `sftp:\\xx.xx.xx.xx` ，默认用户名为：`pi` ，默认密码是：`raspberry` 

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

### 安装 Windows 10 IoT

参考：https://docs.microsoft.com/en-us/windows/iot-core/getstarted

https://docs.microsoft.com/en-us/windows/iot-core/tutorials/quickstarter/devicesetup#using-the-iot-dashboard-raspberry-pi-and-minnowboard

https://www.windowscentral.com/how-install-windows-10-iot-raspberry-pi-3

> 1. 安装工具  IoT Dashboard
> 2. 在 IoT Dashboard 设置新设备
> 3. 进行安装
> 4. 开机后，远程访问用户名：Administrator，密码：p@ssw0rd

------

### 安装 Ubuntu

下载 Ubuntu 镜像：https://ubuntu-mate.org/download/

安装方法：https://ubuntu-mate.org/raspberry-pi/

需要用到工具：Win32 Disk Imager：https://sourceforge.net/projects/win32diskimager/

------

### 安装 CentOS

安装参考：https://wiki.centos.org/SpecialInterestGroup/AltArch/Arm32/RaspberryPi3

下载：http://mirror.centos.org/altarch/7/isos/armhfp/

步骤：[参考](https://blog.csdn.net/elesos/article/details/80514659) 

- 格式化 SD 卡
- 下载系统，并解压得到 raw 文件
- 使用 Win32DiskImager，写入系统镜像

CentOS，采用 `yum` 安装软件，官方源速度过慢，可使用国内镜像，阿里 和 网易

[替换 yum 源](https://www.cnblogs.com/renpingsheng/p/7845096.html) 

[CentOS 介绍](https://wiki.centos.org/SpecialInterestGroup/AltArch/armhfp#head-1c815b610e02202bb55dc1bb1f830b11b9099e28) 

------

### 扩充SD卡空间

默认只支持 4G 空间，可通过 `fdisk` 重写分区

[参考](https://blog.csdn.net/xmm1981/article/details/79472095) 

> 在终端中输入`fdisk /dev/mmcblk0`进入硬盘分区软件  
>
> 在软件中输入：
>
>  `p`——查看旧分区情况 
>
>  `d`——删除分区，并按照提示删除第三个分区 
>
>  `n`——添加一个分区，空间起始位置按照系统默认（默认是最大空间）
>
>  `p`——查看新分区情况  `w`——写入分区信息并退出软件  
>
> 在终端中输入：`reboot`重启树莓派  
>
> 在树莓派开机后在终端输入：`resize2fs /dev/mmcblk0p3`重新加载分区信息 

### Camera Sensor

官方介绍：https://www.raspberrypi.org/documentation/raspbian/applications/camera.md

树莓派官方的摄像头模块为 Pi Cam，为 CSI 接口，树莓派系统默认包含驱动及使用工具，可即插即用，进行拍照、录像及视频流传输，还可直接获取 raw 数据，参考网址：https://linux.cn/article-3650-1.html / http://www.cnblogs.com/uestc-mm/p/7587783.html  /  https://www.raspberrypi.org/documentation/usage/camera/README.md

当前还有 USB WebCam，CSI 接口可直接使用GPU，USB需要使用CPU进行压缩处理

RPi 默认已经支持较多的 USB Webcam，可从https://elinux.org/RPi_USB_Webcams 查询，包括但不限于

USB WebCam 使用参考：https://www.raspberrypi.org/documentation/usage/webcams/ / https://linux.cn/article-5312-1.html

板载定焦镜头，支持 1080p30，720p60，640x480p60/90

camera python 编程参考：https://blog.csdn.net/talkxin/article/details/50504601

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

### OpenCV

安装参考：https://blog.csdn.net/u010429424/article/details/76824521

openCV Github：https://github.com/opencv/opencv/tree/3.4.1

如果连接失败，可切换镜像源：https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/

### Movidius Neural Compute Stick

英特尔 Movidius NCS 低功耗的视觉处理单元，以 USB 棒形式，可灵活扩展在其他低功耗的嵌入式系统

Raspberry Pi Support：https://ncsforum.movidius.com/discussion/118/movidius-nc-sdk-1-07-07-with-raspberry-pi-support

在 Raspberry 上运行，有两种方式：

- 完整 SDK 模式  https://developer.movidius.com/start
- 仅 API 模式，安装速度快，但无法对图形文件进行分析、验证和神经计算

API 模式，仅能完成学习部分，与完整 SDK 模式，仅安装不同

#### 安装

> 1. 在 RPI 安装 Python 依赖
>
>    ```
>    sudo apt-get install -y libusb-1.0-0-dev libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler libatlas-base-dev git automake byacc lsb-release cmake libgflags-dev libgoogle-glog-dev liblmdb-dev swig3.0 graphviz libxslt-dev libxml2-dev gfortran python3-dev python-pip python3-pip python3-setuptools python3-markdown python3-pillow python3-yaml python3-pygraphviz python3-h5py python3-nose python3-lxml python3-matplotlib python3-numpy python3-protobuf python3-dateutil python3-skimage python3-scipy python3-six python3-networkx python3-tk
>    ```
>
> 2. 下载 NCS SDK
>
>    ```markdown
>    mkdir -p ~/workspace
>    cd ~/workspace
>    git clone https://github.com/movidius/ncsdk
>    ```
>
> 3. 编译安装 NCS SDK API
>
>    ```
>    cd ~/workspace/ncsdk/api/src
>    make
>    sudo make install
>    ```
>
> 4. 安装 NCSDK
>
>    ```
>    cd ~/workspace/ncsdk
>    make install
>    ```
>
> 5. 实例工程验证
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

#### 图形分类

> 在下面进行图形识别前，还必须得到基于 GoogleNet 的图形文件，此图形文件需要在 SDK 完整工具包生成，即需要在完整安装了 SDK 的开机机器上生成，然后拷贝到 RPI
>
> 1. 生成图形文件：https://movidius.github.io/ncsdk/tools/compile.html
>
>    ```
>    mvNCCompile deploy.prototxt -w bvlc_googlenet.caffemodel -s 12 -in input -on prob -is 224 224 -o GoogLeNet.graph
>    ```
>
> 2. 构建图形分类
>
>    参考：https://movidius.github.io/blog/ncs-image-classifier/
>
>    深入学习使用多个相互连接的神经元层，每个层使用特定的计算机算法识别和分类特定描述符。深神经网络 DNN 将任务分解成许多简单算法的能力可与更大的描述符集一起工作
>
>    图像分类：假定整个图像只有一个对象
>
>    图像检测：可处理同一图像中的多个对象，还可识别出在图像中的位置
>
>    ```markdown
>    cd ~/workspace
>    git clone https://github.com/movidius/ncappzoo
>    cd ncappzoo/apps/image-classifier
>    python3 image-classifier.py
>    
>    ## output
>    ------- predictions --------
>    prediction 1 is n02123159 tiger cat
>    prediction 2 is n02124075 Egyptian cat
>    prediction 3 is n02113023 Pembroke, Pembroke Welsh corgi
>    prediction 4 is n02127052 lynx, catamount
>    prediction 5 is n02971356 carton
>    ```
>
> 3. 构建图像分类器，只需几行 Python 脚本，以下为可配置的图像分类器参数
>
>    - GRAPH_PATH：图形文件的位置
>
>      默认设置为 ~/workspace/ncappzoo/caffe/GoogLeNet/graph
>
>    - IMAGE_PATH：进行分类的图像的位置
>
>      默认设置为 ~/workspace/ncappzoo/data/image/cat.jpg
>
>    - IMAGE_DIM：所选用的神经网络定义的图像尺寸
>
>      GoogLeNet 使用 224 x 224 像素，AlexNet 使用 227 x 227 像素
>
>    - IMAGE_STDDEV：所选用的神经网络定义的标准偏差
>
>      GoogLeNet 不使用缩放因子，InceptionV3 使用 128 （stddev = 1/128）
>
>    - IMAGE_MEAN：平均减法，深度学习中用于数据中心的常用方法
>
>      ILSVRC 数据集，平均值 B = 102，绿色 = 117，红色 = 123

#### SDK 推断

首先需要从 mvnc 库中导入 mvncapi 模块

```python
import mvnc.mvncapi as mvnc
```

> 1. 打开枚举的设备
>
>    调用 API 查找枚举的 NCS 设备
>
>    ```python
>    # Look for enumerated Intel Movidius NCS device(s); quit program if none found.
>    devices = mvnc.EnumerateDevices()
>    if len( devices ) == 0:
>        print( 'No devices found' )
>        quit()
>    
>    ## only select one NCS
>    # Get a handle to the first enumerated device and open it
>    device = mvnc.Device( devices[0] )
>    device.OpenDevice()
>    ```
>
>    **可将多个神经计算棒连接到同一个处理器，以扩展性能**
>
> 2. 将图形文件加载到 NCS
>
>    这里，使用预先编译的 AlexNet 模型（在 ncappzoo 文件夹中运行 `make` ）
>
>    ```python
>    # Read the graph file into a buffer
>    with open( GRAPH_PATH, mode='rb' ) as f:
>        blob = f.read()
>    
>    # Load the graph buffer into the NCS
>    graph = device.AllocateGraph( blob )
>    ```
>
> 3. 将单个映像使用 NCS 进行推断
>
>    所有的神经网络处理由 NCS 单独完成，释放处理器的 CPU 和内存资源
>
>    需要对图像进行预处理
>
>    - 调整/裁剪图像以匹配预编译的模型   **IMAGE_DIM**
>    - 从整个数据集中平均减法  **IMAGE_MEAN**
>    - 将图像转换为半精度浮点型（fp16）数组，使用 `LoadTensor` 函数将图像加载到 NCS
>
>    ```Python
>    # Read & resize image [Image size is defined during training]
>    img = print_img = skimage.io.imread( IMAGES_PATH )
>    img = skimage.transform.resize( img, IMAGE_DIM, preserve_range=True )
>    
>    # Convert RGB to BGR [skimage reads image in RGB, but Caffe uses BGR]
>    img = img[:, :, ::-1]
>    
>    # Mean subtraction & scaling [A common technique used to center the data]
>    img = img.astype( numpy.float32 )
>    img = ( img - IMAGE_MEAN ) * IMAGE_STDDEV
>    
>    # Load the image as a half-precision floating point array
>    graph.LoadTensor( img.astype( numpy.float16 ), 'user object' )
>    ```
>
> 4. 从 NCS 读取和打印推理结果
>
>    可选择使用阻塞或非阻塞函数调用加载图像及获取推断结果，这里使用 阻塞调用
>
>    ```python
>    # Get the results from NCS
>    output, userobj = graph.GetResult()
>    
>    # Print the results
>    print('\n------- predictions --------')
>    
>    labels = numpy.loadtxt( LABELS_FILE_PATH, str, delimiter = '\t' )
>    
>    order = output.argsort()[::-1][:6]
>    for i in range( 0, 5 ):
>        print ('prediction ' + str(i) + ' is ' + labels[order[i]])
>    
>    # Display the image on which inference was performed
>    skimage.io.imshow( IMAGES_PATH )
>    skimage.io.show( )
>    ```
>
> 5. 卸载并关闭
>
>    为避免内存泄漏或分割错误，应关闭所有打开的文件或资源，并解除分配任何已用的内存
>
>    ```python
>    graph.DeallocateGraph()
>    device.CloseDevice()
>    ```
>

### NCS

参考：[高速目标检测](http://www.cnblogs.com/hellocwh/p/8587013.html) 