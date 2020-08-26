
<!--- Licensed to the Apache Software Foundation (ASF) under one -->
<!--- or more contributor license agreements.  See the NOTICE file -->
<!--- distributed with this work for additional information -->
<!--- regarding copyright ownership.  The ASF licenses this file -->
<!--- to you under the Apache License, Version 2.0 (the -->
<!--- "License"); you may not use this file except in compliance -->
<!--- with the License.  You may obtain a copy of the License at -->

<!---   http://www.apache.org/licenses/LICENSE-2.0 -->

<!--- Unless required by applicable law or agreed to in writing, -->
<!--- software distributed under the License is distributed on an -->
<!--- "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY -->
<!--- KIND, either express or implied.  See the License for the -->
<!--- specific language governing permissions and limitations -->
<!--- under the License. -->

[参考](https://zhuanlan.zhihu.com/p/71558181)

# Android TVM RPC

This folder contains Android RPC app that allows us to launch an RPC server on a Android device and connect to it through python script and do testing on the python side as normal TVM RPC.

You will need JDK, [Android NDK](https://developer.android.com/ndk) and an Android device to use this.

## Build and Installation

### <a name="buildapk">Build APK</a>

We use [Gradle](https://gradle.org) to build. Please follow [the installation instruction](https://gradle.org/install) for your operating system.

Before you build the Android application, please refer to [TVM4J Installation Guide](https://github.com/dmlc/tvm/blob/master/jvm/README.md) and install tvm4j-core to your local maven repository. You can find tvm4j dependency declare in `app/build.gradle`. Modify it if it is necessary.

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:26.0.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    compile 'com.android.support:design:26.0.1'
    compile 'ml.dmlc.tvm:tvm4j-core:0.0.1-SNAPSHOT'
    testCompile 'junit:junit:4.12'
}
```

Now use Gradle to compile JNI, resolve Java dependencies and build the Android application together with tvm4j. Run following script to generate the apk file.

```bash
export ANDROID_HOME=[Path to your Android SDK, e.g., ~/Android/sdk]
cd apps/android_rpc
gradle clean build
```

In `app/build/outputs/apk` you'll find `app-release-unsigned.apk`, use `dev_tools/gen_keystore.sh` to generate a signature and use `dev_tools/sign_apk.sh` to get the signed apk file `app/build/outputs/apk/tvmrpc-release.apk`.

Upload `tvmrpc-release.apk` to your Android device and install it.

### Build with OpenCL

This application does not link any OpenCL library unless you configure it to. In `app/src/main/jni/make` you will find JNI Makefile config `config.mk`. Copy it to `app/src/main/jni` and modify it.

```bash
cd apps/android_rpc/app/src/main/jni
cp make/config.mk .
```

Here's a piece of example for `config.mk`.

```makefile
APP_ABI = arm64-v8a

APP_PLATFORM = android-17

# whether enable OpenCL during compile
USE_OPENCL = 1

# the additional include headers you want to add, e.g., SDK_PATH/adrenosdk/Development/Inc
ADD_C_INCLUDES = /opt/adrenosdk-osx/Development/Inc

# the additional link libs you want to add, e.g., ANDROID_LIB_PATH/libOpenCL.so
ADD_LDLIBS = libOpenCL.so
```

Note that you should specify the correct GPU development headers for your android device. Run `adb shell dumpsys | grep GLES` to find out what GPU your android device uses. It is very likely the library (libOpenCL.so) is already present on the mobile device. For instance, I found it under `/system/vendor/lib64`. You can do `adb pull /system/vendor/lib64/libOpenCL.so ./` to get the file to your desktop.

After you setup the `config.mk`, follow the instructions in [Build APK](#buildapk) to build the Android package.

## Cross Compile and Run on Android Devices

### Architecture and Android Standalone Toolchain

In order to cross compile a shared library (.so) for your android device, you have to know the target triple for the device. (Refer to [Cross-compilation using Clang](https://clang.llvm.org/docs/CrossCompilation.html) for more information). Run `adb shell cat /proc/cpuinfo` to list the device's CPU information.

Now use NDK to generate standalone toolchain for your device. For my test device, I use following command.

```bash
cd /opt/android-ndk/build/tools/
./make-standalone-toolchain.sh --platform=android-24 --use-llvm --arch=arm64 --install-dir=/opt/android-toolchain-arm64
```

If everything goes well, you will find compile tools in `/opt/android-toolchain-arm64/bin`. For example, `bin/aarch64-linux-android-g++` can be used to compile C++ source codes and create shared libraries for arm64 Android devices.

### Cross Compile and Upload to the Android Device

First start an RPC tracker using

```python -m tvm.exec.rpc_tracker --port [PORT]```

and connect your Android device to this RPC tracker via the TVM RPC application. Open the app,
set the `Address` and `Port` fields to the address and port of the RPC tracker respectively.
The key should be set to "android" if you wish to avoid modifying the default test script.

After pushing "START RPC" button on the app, you can check the connect by run

```python -m tvm.exec.query_rpc_tracker --port [PORT]```

on your host machine.
You are supposed to find a free "android" in the queue status.

```
...

Queue Status
-------------------------------
key       total  free  pending
-------------------------------
android   1      1     0
-------------------------------
```


Then checkout [android\_rpc/tests/android\_rpc\_test.py](https://github.com/dmlc/tvm/blob/master/apps/android_rpc/tests/android_rpc_test.py) and run,

```bash
# Specify the RPC tracker
export TVM_TRACKER_HOST=0.0.0.0
export TVM_TRACKER_PORT=[PORT]
# Specify the standalone Android C++ compiler
export TVM_NDK_CC=/opt/android-toolchain-arm64/bin/aarch64-linux-android-g++
python android_rpc_test.py
```

This will compile TVM IR to shared libraries (CPU, OpenCL and Vulkan) and run vector addition on your Android device. To verify compiled TVM IR shared libraries on OpenCL target set [`'test_opencl = True'`](https://github.com/dmlc/tvm/blob/master/apps/android_rpc/tests/android_rpc_test.py#L25) and on Vulkan target set [`'test_vulkan = False'`](https://github.com/dmlc/tvm/blob/master/apps/android_rpc/tests/android_rpc_test.py#L27) in  [tests/android_rpc_test.py](https://github.com/dmlc/tvm/blob/master/apps/android_rpc/tests/android_rpc_test.py), by default on CPU target will execute.
On my test device, it gives following results.

```bash
Run CPU test ...
0.000962932 secs/op

Run GPU(OpenCL Flavor) test ...
0.000155807 secs/op

[23:29:34] /home/tvm/src/runtime/vulkan/vulkan_device_api.cc:674: Cannot initialize vulkan: [23:29:34] /home/tvm/src/runtime/vulkan/vulkan_device_api.cc:512: Check failed: __e == VK_SUCCESS Vulan Error, code=-9: VK_ERROR_INCOMPATIBLE_DRIVER

Stack trace returned 10 entries:
[bt] (0) /home/user/.local/lib/python3.6/site-packages/tvm-0.4.0-py3.6-linux-x86_64.egg/tvm/libtvm.so(dmlc::StackTrace[abi:cxx11]()+0x53) [0x7f477f5399f3]
.........

You can still compile vulkan module but cannot run locally
Run GPU(Vulkan Flavor) test ...
0.000225198 secs/op
```

You can define your own TVM operators and test via this RPC app on your Android device to find the most optimized TVM schedule.


* 步骤
  * 1. 编译 Android TVM RPC apk，安装到安卓平台，打开配置tvm服务器信息
  * 2. 服务器端编译TVM，启动Tracker
  * 3. 启动Auto-TVM tuning

## 1. 编译Android TVM RPC apk
   * 已经生成好的 apk
       
   * 依赖环境：JDK, Android SDK, Android NDK ，Gradle， export ANDROID_HOME=[Path to your Android SDK, e.g., ~/Android/sdk]
   * 使用docker 环境
     * 安装docker：
       * 一键安装docker命令： curl -sSL https://get.daocloud.io/docker | sh
     * 编译docker环境：
       * cd 到tvm主目录，可能需要修改./docker/build.sh 为unix格式
       * 执行： ./docker/build.sh demo_android -it bash 
       * 有问题！ Processing triggers for libc-bin (2.23-0ubuntu11.2) ...
   * 安装openjdk
     * apt install openjdk-8-jdk
   * 安装Gradle java编译工具
     * wget https://services.gradle.org/distributions/gradle-5.5.1-bin.zip
     * mkdir /opt/gradle
     * unzip -d /opt/gradle gradle-5.5.1-bin.zip
     * ls /opt/gradle/gradle-5.5.1  
     * export PATH=$PATH:/opt/gradle/gradle-5.5.1/bin
   * 安装Android SDK 和 Android NDK
     * 下载
       * Android SDK LINUX版本地址 : https://dl.google.com/android/android-sdk_r24.4.1-linux.tgz  tar -zxvf 解压，包含adb工具等
         * sdkmanager 工具在 Android SDK Tools 软件包（25.2.3 及更高版本）中提供，并位于 android_sdk/tools/bin/ 下
         * 另外 sdkmanager 工具 也可以在 sdk-tools-linux 中找到 https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip?utm_source=androiddevtools&utm_medium=website
       * Android NDK LINUX版本地址 ：https://developer.android.google.cn/ndk/downloads/
       * 单独的 Android ADB LINUX版本地址 ：https://dl.google.com/android/repository/platform-tools-latest-linux.zip
     * 安装
       * 编辑 /etc/profile
       
        vim /etc/profile
        
        export ANDROID_HOME=xxxx/android-sdk-linux
        
        export PATH=$ANDROID_HOME/tools:$PATH
        
        export PATH=$ANDROID_HOME/platform-tools:$PATH
        
        export ANDROID_NDK=xxxx/android-ndk-r16-linux-x86_64
        
        export PATH=$ANDROID_NDK:$PATH
        
       * 生效
         source /etc/profile
       * 更新 Android SDK 并获取许可
         * 解压android-sdk_r24.4.1-linux.tgz
         * cd android-sdk-linux/
         * 更新依赖 tools/android update sdk --no-ui   耗时太长！
         * 获取 许可
           * 将 sdk-tools-linux/tools 拷贝到 android-sdk-linux/tools
           * cd android-sdk-linux/tools/bin
           * 获取许可 ./sdkmanager --licenses
       
   * 编译 RPC apk
     * 编译安装 tvm4j-core， [参考](https://github.com/apache/incubator-tvm/blob/master/jvm/README.md)
       * 编译tvm后 再编译 tvm4j-core
       * 安装依赖环境：
         * 安装 JDK 1.6+
           * apt-get install openjdk-8-jdk
         * 安装maven 3
           * wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
           * mv apache-maven-3.6.1-bin.tar.gz /usr/local/
           * tar -zxvf apache-maven-3.6.3-bin.tar.gz
           * 编辑 /etc/profile 文件
             * 添加如下内容：
               export M2_HOME=/usr/local/apache-maven-3.6.3
               export PATH=${M2_HOME}/bin:$PATH
           * maven 3环境生效：
             source /etc/profile
       * 编译 jvmpkg
         * 编译 cd tvm & make jvmpkg  
         * 测试 sh tests/scripts/task_java_unittest.sh
         * 安装 make jvminstall 
         * 这将编译、打包并安装 tvm4j 到本地 maven 仓库
     * 编译apk
       * cd  xxx/tvm/apps/android_rpc
       * gradle clean build
       * 生成未签名的 app/build/outputs/apk/release/app-release-unsigned.apk
       * 编译遇到的问题：
         * 1. sdk未获取许可证 
         * 2. app/src/main/jni/build.sh 编码格式 unix 
         * 3. app/src/main/jni/build.sh 21行  org_apache_tvm_native_c_api.h 文件不存在
         * 4. 实际在 jvm/native/linux-x86_64/target/custom-javah/org_apache_tvm_native_c_api.h 
         * 5. 拷贝到 apps/android_rpc/app/src/main/jni/
     * 编译opencl版本的apk
       * 修改编译配置文件 app/src/main/jni/make/config.mk
       * 获取目标手机的  libOpenCL.so
         * 查询手机libOpenCL.so位置：adb shell dumpsys | grep GLES
         * 可能在：                  /system/vendor/lib64/libOpenCL.so
         * 下载手机的 libOpenCL.so： adb pull /system/vendor/lib64/libOpenCL.so ./
         * 上面 ADD_LDLIBS = libOpenCL.so 写好 相应的地址
         
```
# app/src/main/jni/make/config.mk   OpenCL版本
APP_ABI = arm64-v8a
APP_PLATFORM = android-17

# whether enable OpenCL during compile
USE_OPENCL = 1

# the additional include headers you want to add, e.g., SDK_PATH/adrenosdk/Development/Inc
ADD_C_INCLUDES = /opt/adrenosdk-osx/Development/Inc

# the additional link libs you want to add, e.g., ANDROID_LIB_PATH/libOpenCL.so
ADD_LDLIBS = libOpenCL.so
```
   * apk 签名&安装到安卓平台
     * 使用 apps/android_rpc/dev_tools 对rpc apk进行签名
       * 生成签名文件         ./dev_tools/gen_keystore.sh   需要输入密码 和姓氏等信息
       * 给apk进行签名认证    ./dev_tools/sign_apk.sh       需要上面的密码
       * 生成的签名的apk地址：app/build/outputs/apk/release/tvmrpc-release.apk
     * 使用 adb install xxx.apk 安装到安卓上
     
   * 为手机安装独立的交叉编译链  方便主机端搜索到代码后直接编译成目标平台代码
     * cd $ANDROID_NDK/build/tools/
     * ./make-standalone-toolchain.sh --platform=android-24 --use-llvm --arch=arm64 --install-dir=xxx/android-toolchain-arm64
     * 生成的工具链在 xxx/android-toolchain-arm64/bin/*
     * 例如 bin/aarch64-linux-android-g++ 可以为 arm64手机 编译C++程序
     * 导入环境变量
       *  export ANDROID_TC=xxx/android-toolchain-arm64
          export PATH=${ANDROID_TC}/bin:$PATH
   * 建立连接
     * 启动代理服务器并使用我们的应用进行连接：
       * 服务器 python3 -m tvm.exec.rpc_proxy --host=0.0.0.0 --port=9190 
     * apk连接服务器：
       * 安卓app 打开RPC app 输入ip、端口号等信息连接服务器
       * 第一个参数Address ip   : 10.110.6.16   是PC在局域网中的IP地址
       * port : 9190                            port为启动tracker的端口（默认为9190）
       * key  : android                         Key为自选的密钥（PC与android通讯时需要进行匹配）
     * 在本地主机查看所有的注册设备：
       * python3 -m tvm.exec.query_rpc_tracker --host=0.0.0.0 --port=9190
       *  出现
       
Tracker address 0.0.0.0:9190

Server List
----------------------------
server-address	key
----------------------------
10.110.6.136:57764	server:android

       
