---
id: Wasm_Operation_Principle
title: Operating principle of wasm virtual machine
sidebar_label: Operating principle of wasm virtual machine
---

## 前言

WASM（WebAssembly）不是一种语言，而是一种新的字节码格式，是一种全新的底层二进制语法，经编译器编译后的指令代码。其体积小、可移植、加载快并兼容WEB的全新格式，是一种以安全有效的方式运行可移植程序的新技术。本文详细解析了PlatON WASM合约的引入、优势、运行机理与使用。

## WASM的引入背景

早期WASM主要的应用领域在web应用，但随着其不断地发展，越来越多的项目将其作为智能合约的最终格式，使用对应的VM加载运行，如wavm、wagon，字节码的具体解析与运行则在核心虚拟机模块中进行。

虚拟机（VM Virtual Machine）指的是，通过软件模拟的、具备完整硬件系统功能并运行在隔离环境下的完整计算机系统，比如操作系统虚拟化的Vmware虚拟机，软件运行环境的JVM虚拟机等。而区块链中的虚拟机是建立在区块链上的代码运行环境，其主要作用是处理区块链的智能合约。

**PlatON为何要在引用EVM虚拟机的同时引入WASM虚拟机?**

随着区块链应用对虚拟机和智能合约的需求增多，区块链虚拟机技术也在逐渐完善。目前，基于WASM的虚拟机在速度和性能方面都有了显著提升，支持C、C++、Rust、Golang等多种高级开发语言，开发门槛更低。

为了让区块链应用开发更便捷，PlatON推出了双虚拟机引擎，同时支持WASM虚拟机和EVM虚拟机。原本在Ethereum等支持EVM的公链上运行的区块链应用，如果由于性能等原因需要使用PlatON，可以轻松实现无缝迁移。

EVM在本质上是脚本程序，是基于栈的虚拟机，需要由编译程序翻译成指令后执行，这样的执行方式在效率上是非常低的。同时EVM在架构和使用上也存在一些问题，例如：

- *强耦合，难扩展*：链本身与EVM内部之间存在相互依赖的关系；

- *缺少标准库*：solidity中基本没有标准的库，必须通过自己编码才能实现相应功能，如：字符串比较。

与EVM相比，WASM作为一种中间代码，拥有更高更快的智能合约运行速度，可以使用任何熟悉的编程语言进行开发（C/C++/Rust/Golang），同时存在诸多特点：

- *性能高效*：采用二进制编码，在程序执行过程中性能优越；

- *多语言支持*：可使用C/C++/Rust/Golang等多种语言编写合约后编译出字节码；

- *丰富的标准库*：不同语言有丰富的标准库可供使用。

同时，越来越多的项目使用WASM，包括引入EVM虚拟机的Ethereum也在开始转向WASM。WASM越发流行，其社区生态体系也越来越完善。

为让已经存在的一些WASM应用或者一些熟悉WASM的开发者更好地切换到PlatON生态中来，PlatON经综合考虑后决定，在已存在EVM虚拟机的同时引入WASM虚拟机，实现双虚拟机支持。

## PlatON引入WASM的优势

PlatON在决定引入虚拟机之初就考虑到了EVM的一些弊端，在初步引入EVM虚拟机之后马上就开始了对于WASM虚拟机的相关方案设计及开发，在拓展性、稳定性、运行效率等方面不断地创新和改进。主要功能点包含：

- 提供一整套完备的合约开发套件[PlatON-CDT](https://github.com/PlatONnetwork/PlatON-CDT);

- 更小的合约字节码，使用CDT套件编译出来的WASM二进制码，大小相比其他项目减少 1/2；

- 提供可用于进行快速开发的智能合约框架[PlatON-Truffle](https://github.com/PlatONnetwork/platon-truffle)；

- 扩展对string, address, hash, u128等数据类型的支持；

- 增加了Gas机制。每执行一条合约指令，都会扣除相应的Gas，确保合约指令在执行有限次运算后，一定可以终止执行，有效防止代码无限循环攻击；

- 增加了合约嵌套调用功能，可以立即获取到调用另外一个合约方法的结果，使得合约之间的调用像函数调用一样方便；

- 支持自动扩展线性内存，缓存wasm module，优化合约的加载性能，从而极大提升了虚拟机运行性能。

基于对WASM的应用及优化改造，PlatON的智能合约模块具备更高的兼容性与性能，同时拥有更强的安全性与灵活性。在成功集成WASM，实现双虚拟机支持之后，PlatON在同等环境下进行了EVM和WASM性能对比测试。

测试结果表明，基于WASM虚拟机的执行效率远远高于EVM虚拟机。EVM使用了256位的机器码在很大程度上影响了执行的性能，同时增大了内存的占用率。

PlatON在引用WASM虚拟机实现双虚机支持后，主要具备如下优势：

- 主流的WASM智能合约可以方便快捷地切入到PlatON网络；

- 降低合约开发者的门槛，学习成本低；

- 使用其他基于WASM的网络的开发者可以无缝切换，上手快；

- 智能合约的执行效率更高效；

- 丰富的开发库，提升智能合约开发效率。

## WASM的运行机理

WASM虚拟机是一个完全独立的沙盒，合约代码对外完全隔离并在虚拟机内部运行，运行在虚拟机内部的代码不能接触到网络、文件系统或其它进程。同时，为了防止虚拟机执行过多的计算指令，陷入死循环等情况，引入了燃料（Gas）机制+超时机制双重控制。

**双虚拟机工作原理：**

<img src="/docs/img/en/wasm-dual.png" />

如图所示：使用solidity/c/c++等语言编写的智能合约，经过编译器编译成为字节码，同时不同类型的合约包含了不同的字节码特征，PlatON双虚拟机引擎通过对特征的识别，可以判断该段字节码需要使用哪种虚拟机进行执行，选择对应虚拟机后加载字节码，解析字节码，然后根据指令去执行对应的函数功能。

**智能合约使用流程：**

<img src="/docs/img/en/wasm-contract.png" />  

如图所示，可以使用c++编写基于WASM的智能合约，也可以使用solidity编写属于EVM的智能合约，然后通过工具套件编译生成字节码等信息，再通过SDK开发工具集通过交易的方式发送到PlatON网络节点中。这样一个与合约相关的交易流程就完成了。

PlatON提供了完备的WASM合约开发套件PlatON-CDT，使用该套件，可以快速地编写出WASM虚拟机的智能合约，该套件提供了多种数据类型的封装，简洁的API接口，同时配套详细的接口文档。同时，还可以使用智能合约测试框架PlatON-Truffle进行开发、编译、部署、合约接口测试等功能，方便开发者开发与测试合约。

PlatON提供了[合约SDK](https://github.com/PlatONnetwork/client-sdk-java)自动生成工具，能自动生成后端系统调用合约接口的代码，可以屏蔽调用合约时的参数编解码细节等，帮助开发者快速集成合约业务到系统中，从而可以投入更多的精力到智能合约业务逻辑的开发中。


  ## 总结  

本文主要探讨了PlatON支持WASM虚拟机的背景及初衷，同时介绍了关于引入WASM后PlatON的独有优势，还对WASM虚拟机的基本原理及运行机制做了简单说明。希望能对参与PlatON的社区用户、开发者、相关的区块链从业者提供一些可借鉴的经验。WASM从最初的不被看好到现在被广泛使用，我们可以肯定未来它将作为分布式应用开发的基础层被运用到越来越多的项目之中。