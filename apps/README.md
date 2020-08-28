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

# TVM Application Extensions and Examples
This folder contains various extension projects using TVM,
they also serve as examples on how to use TVM in your own project.

If you are interested in writing optimized kernels with TVM, checkout [TOPI: TVM Operator Inventory](../topi).

- [extension](extension) How to extend TVM C++ api along with python API.
- [ios_rpc](ios_rpc) iOS RPC server.
- [android_rpc](android_rpc) Android RPC server.
- [benchmark](benchmark) Example end to end compilation benchmarks
- [howto_deploy](howto_deploy) Tutorial on how to deploy TVM with minimum code dependency.


# TVM linux 自动调优

* 步骤 
   * 1. 编译 嵌入式linux端 tvm runtime
     * 已经生成好的 嵌入式linux端（59） tvm runtime
       * 16服务器地址：
         * 非opencl版本 /home/wanyouwen/smbshare/tvm/tvm_0821_target
   * 2. 编译 嵌入式linux端 python环境
     * 已经生成好的 嵌入式linux端（59）  python环境
       * /home/wanyouwen/smbshare/tvm/Python-3.7.7/install
   * 3. 服务器端编译TVM，启动Tracker
   * 4. 启动Auto-TVM tuning

## 1. 编译 嵌入式linux端 tvm runtime
   * 设置交叉工具链
     
```sh
# Native Compiler
export AR_host="ar"
export CC_host="gcc"
export CXX_host="g++"
export LINK_host="g++"

# ARMv8 cross compiler
export cross_compiler=/home/jiangxincong/smbshare/compiler/toolchain/aarch64-himix100-linux/bin
export ARCH=arm
export PATH=$cross_compiler:$PATH
export CROSS_COMPILE=aarch64-himix100-linux             
export CC=$cross_compiler/aarch64-himix100-linux-gcc
export CXX=$cross_compiler/aarch64-himix100-linux-g++    
export LD=$cross_compiler/aarch64-himix100-linux-ld
export AR=$cross_compiler/aarch64-himix100-linux-ar
export AS=$cross_compiler/aarch64-himix100-linux-as
export RANLIB=$cross_compiler/aarch64-himix100-linux-ranlib
export STRIP=$cross_compiler/aarch64-himix100-linux-strip
```
  * 编译 target端 tvm runtime
     * 准备 tvm 软件包，复制一份编译host端的tvm
     * 清理编译 cd  build  & make clean & rm -rf *
     * 由于TVM runtime不需要LLVM，清除LLVM选项
       * cp ../cmake/config.cmake .
       * 确保配置为 set(USE_LLVM OFF)
     * 执行编译
       * cmake ..
       * make runtime -j4
     * 生成 target端 tvm runtime库
     
## 2. 嵌入式linux端 安装 python环境  tvm0.7 需要python3.6以上
   * 准备 源码包
     * 下载 [Python3.7.7.tgz](https://www.python.org/downloads/release/python-377/)
       * 16地址：/home/wanyouwen/smbshare/tvm/Python-3.7.7
     * 下载 [openssl-1.1.1g.tar.gz](https://www.openssl.org/source/openssl-1.1.1g.tar.gz)
       * 16地址：/home/wanyouwen/smbshare/tvm/openssl-1.1.1g
     * 下载 [zlib-1.2.11.tar.gz](http://www.zlib.net/zlib-1.2.11.tar.gz)
       * 16地址：/home/wanyouwen/smbshare/tvm/zlib-1.2.11
   * 设置交叉工具链环境
     * 如上 步骤1中一样
   * 交叉编译 openssl-1.1.1b
     * cd openssl-1.1.1g &  mkdir install
     * --prefix 配置安装路径
     * ./config no-asm no-shared --prefix=/home/wanyouwen/smbshare/tvm/openssl-1.1.1g/install
     * 经过上述命令会生成Makefile，然后打开Makefile，找到CROSS_COMPILE的定义，把CROSS_COMPILE的值改为空
     * 删除 Makefile中 -m64 编译选项
     * 编译安装：
       * make & make install 
   * 交叉编译 zlib-1.2.11
     * cd zlib-1.2.11 &  mkdir install
     * --prefix 配置安装路径
     * ./configure --prefix=/home/wanyouwen/smbshare/tvm/zlib-1.2.11/install
     * 编译安装：
       * make & make install 
   * 交叉编译 Python-3.7.7
     * cd Python-3.7.7 &  mkdir install
     * --prefix 配置安装路径 
     * ./configure --host=aarch64-himix100-linux --build=arm --with-openssl=/home/wanyouwen/smbshare/tvm/openssl-1.1.1g/install --with-system-ffi=/home/wanyouwen/smbshare/tvm/libffi-3.3/install/lib --disable-ipv6 ac_cv_file__dev_ptmx=no ac_cv_file__dev_ptc=no --prefix=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install
     * 配置完后，在当前路径下，打开Modules/Setup.dist  & Modules/Setup
     * 配置 openssl  210行左右
       * SSL=/home/wanyouwen/smbshare/tvm/openssl-1.1.1g/install
       * _ssl _ssl.c \
       * 	-DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
       * 	-L$(SSL)/lib -lssl -lcrypto
     * 配置 zlib    338行左右
       * my_prefix=/home/wanyouwen/smbshare/tvm/zlib-1.2.11/install
       * zlib zlibmodule.c -I$(my_prefix)/include -L$(my_prefix)/lib -lz
     * 开始编译
       * make & make install
       * 编译好的 嵌入式端的 python环境在 /home/wanyouwen/smbshare/tvm/Python-3.7.7/install
       * 可能会遇到的问题1
         * threads_pthread.c:(.text+0x18): undefined reference to `pthread_atfork'
         * -lpthread是老版本的gcc编译器用的，在新版本中应该用-pthread取代-lpthread
         * 解决：
           * 修改 生成 的 Makefile -lpthread 替换为 -pthread
           * 233行附近：修改为 LIBS=		-lcrypt -pthread -ldl  -pthread -lutil
           
       * 问题2: Failed to build these modules: _ctypes
         * 下载libffi-3.3: http://www.mirrorservice.org/sites/sourceware.org/pub/libffi/libffi-3.3.tar.gz
         * 16地址: /home/wanyouwen/smbshare/tvm/libffi-3.3
         * 编译：
           * cd libffi-3.3 & mkdir install
           * ./configure --host=aarch64-himix100-linux --prefix=/home/wanyouwen/smbshare/tvm/libffi-3.3/install 
           * make && make install 
           * 编译好的以及复制到 /home/wanyouwen/smbshare/tvm/libffi-3.3/install
         * 重新配置 python：
         
         修改 configure 增加 FFI include路径
         
         
            #if test "$with_system_ffi" = "yes" && test -n "$PKG_CONFIG"; then  
            #    LIBFFI_INCLUDEDIR="`"$PKG_CONFIG" libffi --cflags-only-I 2>/dev/null | sed -e 's/^-I//;s/ *$//'`"  
            #else  
            #    LIBFFI_INCLUDEDIR=""  
            #fi  
            LIBFFI_INCLUDEDIR="/home/wanyouwen/smbshare/tvm/libffi-3.3/install/include"  
            
            
            * 重新配置：
              ./configure --host=aarch64-himix100-linux --build=arm --with-openssl=/home/wanyouwen/smbshare/tvm/openssl-1.1.1g/install --with-system-ffi=/home/wanyouwen/smbshare/tvm/libffi-3.3/install/lib --disable-ipv6 ac_cv_file__dev_ptmx=no ac_cv_file__dev_ptc=no --prefix=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install
              * 修改 生成 的 Makefile -lpthread 替换为 -pthread
              * 233行附近：修改为 LIBS=		-lcrypt -pthread -ldl  -pthread -lutil
            * 编译：
              * make & make install
            
            * 编译好的 python环境在 /home/wanyouwen/smbshare/tvm/Python-3.7.7/install
              * 还需要将 libffi-3.3/install/lib/libffi.so.7 拷贝到 Python-3.7.7/install/lib
   * 问题3: 方向键使用不正常
     * 需要安装 readline 和  termcap
       * 下载 [readline](http://ftp.gnu.org/gnu/readline/readline-7.0.tar.gz)
         * 16地址： /home/wanyouwen/smbshare/tvm/readline-7.0
       * 下载 [termcap](http://ftp.gnu.org/gnu/termcap/termcap-1.3.1.tar.gz)
         * 16地址： /home/wanyouwen/smbshare/tvm/termcap-1.3.1
       * 编译 readline
         * ./configure --host=$CROSS_COMPILE --prefix=/home/wanyouwen/smbshare/tvm/readline-7.0/install
         * make -j4 & make install
       * 编译 termcap 
         * ./configure --host=$CROSS_COMPILE --prefix=/home/wanyouwen/smbshare/tvm/termcap-1.3.1/install
         * 修改 makefile：
           * 34行 CFLAGS = -g -fPIC  添加-fPIC 选项 
         * make -j4 & make install
     * 编译python时 Moudles/Setup.dists & Setup 里面需要打开对应的模块
       * 168行     打开 readline readline.c -lreadline -ltermcap
       * 195-198行 打开 fnctl
       * 201行     打开 mmap
       * 229行     打开 _posixsubprocess
       * 175行     打开 math , subprocess 依赖于 math 
       * 177行     打开 _struct
     * 重新编译python  
       * 配置
              ./configure --host=aarch64-himix100-linux --build=arm \
                   --with-openssl=/home/wanyouwen/smbshare/tvm/openssl-1.1.1g/install \
                   --with-system-ffi=/home/wanyouwen/smbshare/tvm/libffi-3.3/install/lib \
                   --disable-ipv6 ac_cv_file__dev_ptmx=no ac_cv_file__dev_ptc=no \
                   LDFLAGS="-L/home/wanyouwen/smbshare/tvm/readline-7.0/install/lib -L/home/wanyouwen/smbshare/tvm/readline-7.0/install/lib -L/home/wanyouwen/smbshare/tvm/termcap-1.3.1/install/lib"  \
                   CPPFLAGS="-I/home/wanyouwen/smbshare/tvm/readline-7.0/install/include -I/home/wanyouwen/smbshare/tvm/termcap-1.3.1/install/include "  \
                   CFLAGS="-I/home/wanyouwen/smbshare/tvm/readline-7.0/install/include -I/home/wanyouwen/smbshare/tvm/termcap-1.3.1/install/include " \
                   --prefix=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install 
      * 修改 makefile 
        * 修改 生成 的 Makefile -lpthread 替换为 -pthread
        * 233行附近：修改为 LIBS=		-lcrypt -pthread -ldl  -pthread -lutil
      * 编译：
      
      
            make Parser/pgen --silent
            export PYTHONPATH=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install/lib/python3.7/site-packages
            make -j4 & make install
            $cross_compiler/aarch64-himix100-linux-strip -s install/bin/python3
            
        
   * 问题4: ModuleNotFoundError: No module named 'numpy' 还需要交叉编译numpy模块
       * 在编译完arm版python以后，再安装第三方包
       * 在第三方源代码目录下，运行PC版本python3 安装，安装时要用--prefix指定arm版本的python的路径
       * python3 setup.py install --prefix=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install
       
       * numpy 安装
         * 源码下载 ： https://files.pythonhosted.org/packages/2c/2f/7b4d0b639a42636362827e611cfeba67975ec875ae036dd846d459d52652/numpy-1.19.1.zip
           * 1.17 版本： https://pypi.org/project/numpy/1.17.0/#files
           * 16位置: /home/wanyouwen/smbshare/tvm/numpy-1.19.1
         * 设置环境变量
       
                export cross_compiler=/home/jiangxincong/smbshare/compiler/toolchain/aarch64-himix100-linux/bin \
                export ARCH=arm  \
                export PATH=$cross_compiler:$PATH  \
                export CROSS_COMPILE=aarch64-himix100-linux     \
                export CC=$cross_compiler/aarch64-himix100-linux-gcc  \
                export CXX=$cross_compiler/aarch64-himix100-linux-g++  \
                export LD=$cross_compiler/aarch64-himix100-linux-ld  \
                export AR=$cross_compiler/aarch64-himix100-linux-ar  \
                export AS=$cross_compiler/aarch64-himix100-linux-as  \
                export RANLIB=$cross_compiler/aarch64-himix100-linux-ranlib  \
                export STRIP=$cross_compiler/aarch64-himix100-linux-strip  \
                export LDFLAGS="-L/home/wanyouwen/smbshare/tvm/Python-3.7.7/install/lib -L/home/jiangxincong/smbshare/compiler/toolchain/aarch64-himix100-linux/target"
                export CFLAGS="-I/home/jiangxincong/smbshare/compiler/toolchain/aarch64-himix100-linux/target -I/home/wanyouwen/smbshare/tvm/Python-3.7.7/install/include/python3.7m  "
                //export LDSHARED="${CC} -shared"
       * 设置编译numpy 默认的 工具链
         * numpy在编译的时候会直接使用 gcc、x86_64-linux-gnu-gcc 和 gfrotran 进行编译。
         * 新建一个目录，构建工具链 软连接到 交叉编译工具
         * mkdir /home/wanyouwen/smbshare/tvm/Python-3.7.7/cros_comp
         * cd /home/wanyouwen/smbshare/tvm/Python-3.7.7/cros_comp
         * ln -s $AR  x86_64-linux-gnu-ar
         * ln -s $CXX x86_64-linux-gnu-g++
         * ln -s $CC  x86_64-linux-gnu-gcc
         * ln -s $LD  ld
         * ln -s $AR  ar
         * ln -s $CXX g++
         * ln -s $CC  gcc
         * ln -s $AS  as
         * export PATH=/home/wanyouwen/smbshare/tvm/Python-3.7.7/cros_comp:$PATH
       * 编译安装numpy
         * cd /home/wanyouwen/smbshare/tvm/numpy-1.19.1
         * 运行安装命令，要把 --prefix 指定为ARM版本的python所在的目录
         * CC=$CC CXX=$CXX LD=$LD CROSS_COMPILE=$CROSS_COMPILE python3 setup.py install --prefix=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install
         
         * 可能会报 TEST FAILED:  python3.7/site-packages does NOT support .pth files bad install directory or PYTHONPATH
           * 需要修改 /etc/sudoers
           * 注释 #Defaults	env_reset
           * 增加 Defaults env_keep += "PYTHONPATH"
           * export PYTHONPATH=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install/lib/python3.7/site-packages
         
         * 可能会报错 系统头文件 连接的不是 交叉编译工具链的 库
         * 需要修改配置文件
         * cp site.cfg.example site.cfg
         * 修改 site.cfg
         * 89-91行 修改为
         
            [DEFAULT]  
            library_dirs = /home/jiangxincong/smbshare/compiler/toolchain/aarch64-himix100-linux/target/usr/lib
            include_dirs = /home/jiangxincong/smbshare/compiler/toolchain/aarch64-himix100-linux/target/usr/include
            
       * 编译成功后  numpy 会被安装到 site-packages/numpy-1.19.1-py3.7-linux-x86_64.egg  
         移动库 mv /home/wanyouwen/smbshare/tvm/Python-3.7.7/install/lib/python3.7/site-packages/numpy-1.19.1-py3.7-linux-x86_64.egg/* ../
       
       * 打包 /home/wanyouwen/smbshare/tvm/Python-3.7.7/install  这个就是可以在嵌入式端运行的python包
       
       * 错误 缺少  pkg_resources    No module named 'pkg_resources'
         * 需要安装 setuptool
         * 下载源码 https://pypi.org/project/setuptools/#files
         * 16位置 /home/wanyouwen/smbshare/tvm/setuptools-49.6.0
         * 安装
         * cd /home/wanyouwen/smbshare/tvm/setuptools-49.6.0
         * CC=$CC CXX=$CXX LD=$LD CROSS_COMPILE=$CROSS_COMPILE  python3 setup.py install --prefix=/home/wanyouwen/smbshare/tvm/Python-3.7.7/install
         * 安装程序会在 python3.7.7/site-packages目录下生成setuptools-46.1.1-py3.7.egg
         * unzip ./setuptools-46.1.1-py3.7.egg
       * 1.19 版本 import numpy 报错 AttributeError: module 'numpy.random.bit_generator' has no attribute 'BitGenerator'
       * 尝试安装 1.17.0 版本
         * 修改 numpy-1.17.0\numpy\random\setup.py
         * 56行 EXTRA_COMPILE_ARGS += ['-msse2'] 修改为  EXTRA_COMPILE_ARGS += [ ] 去除 '-msse2'编译选项
         * 还是报错 AttributeError: module 'numpy.random.common' has no attribute '__pyx_capi__'
       * 尝试安装 1.18.3 版本
         * https://pypi.org/project/numpy/1.18.3/#files
       
   * 嵌入式端安装 python环境
     * 登录嵌入式端，远程挂载 服务器地址（包含编译好的 python安装包install 和 tvm runtime ）
     * export PATH=/mnt/wanyouwen/nfs/AIcompile/tvm/install/bin:$PATH
     * export LD_LIBRARY_PATH=/mnt/wanyouwen/nfs/AIcompile/tvm/install/lib:$LD_LIBRARY_PATH
     
## 3. 嵌入式linux端运行 tvm runtime 
   * 拷贝生成好的 tvm软件包 到嵌入式linux端(或者远程挂载)
   * 设置tvm环境变量
     * export TVM_HOME=/mnt/wanyouwen/nfs/AIcompile/tvm/tvm_0821_target
     * export PYTHONPATH=$TVM_HOME/python:$TVM_HOME/topi/python:$TVM_HOME/nnvm/python:${PYTHONPATH}
     
   * 运行RPC server
     * python3 -m tvm.exec.rpc_server --host 0.0.0.0 --port=9090
     * 返回正确信息
       * INFO:root:RPCServer: bind to 0.0.0.0:9090

## 4. 服务器端运行rpc_tracker 端侧注册
   * 服务器端运行： python3 -m tvm.exec.rpc_tracker --host=0.0.0.0 --port=9190 
   * 端侧连接服务器 python -m tvm.exec.rpc_server --tracker=[HOST_IP]:9190 --key=hisi3559
   * HOST_IP填写服务器端ip地址，key用于服务器端登录设备时的验证码钥匙
     
