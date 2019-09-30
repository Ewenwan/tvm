# TVM 神经网络编译器
TVM可以称为许多工具集的集合，其中这些工具可以组合起来使用，来实现我们的一些神经网络的加速和部署功能。这也是为什么叫做TVM Stack了。TVM的使用途径很广，几乎可以支持市面上大部分的神经网络权重框架(ONNX、TF、Caffe2等)，也几乎可以部署在任何的平台，例如Windows、Linux、Mac、ARM等

[一步一步解读神经网络编译器TVM(一)——一个简单的例子](https://oldpan.me/archives/the-first-step-towards-tvm-1)


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
