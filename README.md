# TVM 神经网络编译器
TVM可以称为许多工具集的集合，其中这些工具可以组合起来使用，来实现我们的一些神经网络的加速和部署功能。这也是为什么叫做TVM Stack了。TVM的使用途径很广，几乎可以支持市面上大部分的神经网络权重框架(ONNX、TF、Caffe2等)，也几乎可以部署在任何的平台，例如Windows、Linux、Mac、ARM等

[一步一步解读神经网络编译器TVM(一)——一个简单的例子](https://oldpan.me/archives/the-first-step-towards-tvm-1)


<img src=https://raw.githubusercontent.com/tqchen/tvm.ai/master/images/logo/tvm-logo-small.png width=128/> Open Deep Learning Compiler Stack
==============================================

[![GitHub license](https://dmlc.github.io/img/apache2.svg)](./LICENSE)
[![Build Status](http://ci.tvm.ai:8080/buildStatus/icon?job=tvm/master)](http://ci.tvm.ai:8080/job/tvm/job/master/)

[Documentation](https://docs.tvm.ai) |
[Contributors](CONTRIBUTORS.md) |
[Community](https://tvm.ai/community.html) |
[Release Notes](NEWS.md)

TVM is a compiler stack for deep learning systems. It is designed to close the gap between the
productivity-focused deep learning frameworks, and the performance- and efficiency-focused hardware backends.
TVM works with deep learning frameworks to provide end to end compilation to different backends.
Checkout the [tvm stack homepage](https://tvm.ai/)  for more information.


# 编译安装


Windows 安装：

 编译器版本: Visual Studio Community 2015 Update 3 +
            CMake 3.5 +
            
 需要支持cuda的话，请装cuda 10.0以上的。
 
1.	下载 安装 LLVM

https://github.com/llvm/llvm-project/releases/tag/llvmorg-10.0.0/ 

LLVM-10.0.0-win64.exe  预编译好的

直接运行这个安装包，按照他的步骤来，中途会有一个问你是否添加到系统路径，你填第二个，即添加到系统路径并所有人都可以使用。安装完成后，把LLVM的路径，比如我的是C:\Program Files\LLVM\bin  （我把这个预编译的LLVM装在C盘），再添加到系统路径下（跟之前的不同）。

然后打开CMD（命令行），输入clang -v，出现下图就表示这个预编译LLVM安装成功。


LLVM 源码安装

下载源码： https://github.com/llvm/llvm-project/tree/llvmorg-10.0.0

cd llvm & make build & cd build 

运行 cmake -G "Visual Studio 15 2017 Win64" .. -Thost=x64 -DLLVM_ENABLE_PROJECTS=clang

如果没有意外，打开build文件夹下的LLVM.sln，确认编译的平台和版本release x64，然后右击生成 Libraries和Tools下面的所有项目。

这个编译时间很长大约30+分钟，最后如果显示生成成功，错误那里显示0个就表示编译OK了。

在build\lib\cmake\llvm有tvm需要的cmake文件（注1）。

在build\Release\bin下面有各种可能用到的工具，可以加到系统PATH。



2.	下载安装 TVM

git 下载源代码

    git clone --recursive https://github.com/apache/incubator-tvm
    cd incubator-tvm
    git submodule init
    git submodule update

如果你直接从GitHub上把整个安装包下下来，编译的时候会有问题，需要通过git下载（git下载方法请自行百度哈）才行，而且上边3句话全部要在命令行中运行才行.

修改tvm源码下面的CMakeLists.txt，把USE_LLVM 设置成 ON，也可以根据需要打开其他功能。   	USE_CUDA等可以也打开 

在tvm下新建build文件夹，通过cmd进入build文件

运行cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_BUILD_TYPE=Release -DCMAKE_CONFIGURATION_TYPES=Release .. -DLLVM_DIR=E:/my_tvm/llvm/build/lib/cmake/llvm

其中-DLLVM_DIR是由上一步生成的，查看注1。

打开tvm.sln, 确认编译的平台和版本release x64，选择ALL_BUILD，右击生成

 
没有意外的话，编译成功，然后cmd分别到tvm/python,tvm/topi/python,运行python setup.py install，这两步是真正的安装，之前的只是编译。

安装其他package（可选）

pip install numpy decorator attrs tornado psutil xgboost mypy orderedset antlr4-python3-runtime

运行python

import tvm





License
-------
© Contributors Licensed under an [Apache-2.0](https://github.com/dmlc/tvm/blob/master/LICENSE) license.

Contribute to TVM
-----------------
TVM adopts apache committer model, we aim to create an open source project that is maintained and owned by the community.
Checkout the [Contributor Guide](https://docs.tvm.ai/contribute/)

Acknowledgement
---------------
We learnt a lot from the following projects when building TVM.
- [Halide](https://github.com/halide/Halide): TVM uses [HalideIR](https://github.com/dmlc/HalideIR) as data structure for
  arithmetic simplification and low level lowering. We also learnt and adapted some part of lowering pipeline from Halide.
- [Loopy](https://github.com/inducer/loopy): use of integer set analysis and its loop transformation primitives.
- [Theano](https://github.com/Theano/Theano): the design inspiration of symbolic scan operator for recurrence.
