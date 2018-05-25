## Build & Download

> **Build environment**
>
> ```c
> //安装 OpenJDK8 并选择默认 jdk 环境为 1.8
> sudo add-apt-repository ppa:openjdk-r/ppa
> sudo apt-get update
> sudo apt-get install openjdk-8-jdk
> sudo update-alternatives --config java
> sudo update-alternatives --config javac
> java -version
> 
> //安装 GCC4.9 并配置
> sudo add-apt-repository ppa:ubuntu-toolchain-r/test
> sudo apt-get update
> sudo apt-get install gcc-4.9 g++-4.9
> sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 10   
> sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 20
> sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 10
> sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.9 20
> sudo update-alternatives --config gcc
> sudo update-alternatives --config g++
> ```
>
> **Build**
>
> ```c
> //进入到源码根目录
> source build/ensetup.sh
> lunch
> 
> //选择合适的编译项，如msm8998
> make -j8 2>&1 | tee build.log
> 
> //re-generate system.img
> make snod
> ```
>
> **Some build errors**
>
> ```markdown
> ## 编译 Android 源码需要配置的安装包，可参考
>  http://wiki.cyanogenmod.org/w/Build_for_hammerhead#Install_the_SDK
> 
> ## 编译log中出现 /bin/bash: prebuilts/misc/linux-x86/bison/bison: No such file or directory
> -->由于系统中未找到 bison 库
> --> sudo apt-get install g++-multilib gcc-multilib lib32ncurses5-dev lib32readline-gplv2-dev lib32z1-dev
> 
> ## 编译log中出现 flex-2.5.39: fatal internal error, exec of /usr/bin/m4 failed
> --> 缺少  bison
> --> sudo apt-get install bison
> 
> ## 编译log中出现 /bin/bash: xmllint: command not found
> --> 缺少libxml2-utils
> --> sudo apt-get  install libxml2-utils
> 
> ## 编译log中出现 fatal error: openssl/opensslv.h: No such file or directory
> -->缺少 libssl-dev
> -->sudo apt-get install libssl-dev
> 
> ## 编译log中出现 android build  Communication error with Jack server (52)
> --> jack 未启动
> --> jack-admin start-server
> 
> ## 编译log中出现 Try increasing heap size with java option '-Xmx<size>  /  Out of space?  /Out of memory error
> -->内存不足
> -->export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
> -->jack-admin kill-server
> -->jack-admin start-server
> ```
>
> **Version number**
>
> ```objective-c
> adb shell getprop ro.build.version.incremental
> adb shell getprop ro.build.fingerprint
> 删去 out/target/product/msm8998/system/build.prop.bakforspec
> 修改 build/core/version-defaults.mk     BUILD_NUMBER := CowboyERRVD3
> ```
>
> **Download**
>
> ```objective-c
> adb reboot bootloader
> fastboot flash boot boot.img
> fastboot flash system system.img
> fastboot flash userdata userdata.img
> fastboot flash cache cache.img
> fastboot flash recovery recovery.img
> ```
>
> **Qualcomm download**
>
> ```markdown
> ## 进入烧录模式
> adb reboot edl
> ```
>
> 