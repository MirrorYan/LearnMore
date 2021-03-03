# WebAssembly

WebAssembly 的目标：

- 快速、高效、可移植 —— 通过利用常见的硬件能力，WebAssembly 代码在不同平台上能够以接近本地速度运行。
-  可读、可调式 —— WebAssembly 是一门低阶语言，但它有一种人类可读的文本格式（其标准即将得到最终版本），这允许通过手工来写代码，看代码以及调试代码。
- 保持安全——WebAssembly被限制运行在一个安全的沙箱执行环境中。像其他网络代码一样，它遵循浏览器的同源策略和授权策略。
- 不破坏网络——WebAssembly的设计原则是与其他网络技术和谐共处并保持向后兼容。

> WebAssembly 也用在网络和 JS 环境之外（参考非网络嵌入）。

## 如何适应网络平台？

网络平台可以想象成拥有两个部分：

- 一个运行网络程序代码（如JS）的虚拟机；
- 一系列网络诚信能够调用从而控制网络浏览器/设备功能，并且能够让事物发生变化的网络API（DOM等）。

过去，虚拟机只能加载 JS，当试图把 JS 应用到一些大量要求原生性能的其他领域的时候，就会遇到性能问题，下载、解析、编译 JS 应用程序的成本是很高的，移动平台和其他资源受限平台进一步放大了这些性能瓶颈。WebAssembly 被设计为和 JS 一起协同工作，从而使得网络开发者能够利用两种语言的优势：

- JS 是一门高级语言，对于写网络应用程序而言，它足够灵活且富有表达力；
- WebAssembly 是一门低级的类汇编语言。它有一种紧凑的二进制格式，使其能够以接近原生性能的速度运行，并且为诸如 C++ 和 Rust 等拥有低级的内存模型语言提供了一个编译目标以便它们能够在网络上运行。

不同类型的代码能够按照需要进行相互调用——WebAssembly 的 JavaScript API 使用能够被正常调用的JS 函数封装了导出的 WebAssembly 代码，并且 WebAssembly 代码能够导入和同步调用常规的 JS 函数。

### WebAssembly 关键概念

- **模块**：表示一个已经被浏览器编译为可执行机器码的 WebAssembly 二进制代码。一个模块是无状态的，并且像一个二进制大对象一样能够被缓存到IndexedDB中或者在Windows和workers之间进行共享（通过postMessage()函数）。一个模块能够像一个ES2015的模块一样声明导入和导出。
- **内存**：ArrayBuffer，大小可变。本质上是连续的字节数组，WebAssembly的低级内存存取指令可以对它进行读写操作。
- **表格**：带类型数组，大小可变。表格中的项存储了**不能作为原始字节存储在内存里的对象**的引用（为了安全和可移植性的原因）。
- **实例**：一个模块及其在运行时使用的所有状态，包括内存、表格和一系列导入值。一个实例就像一个已经被加载到一个拥有一组特定导入的特定的全局变量的ES2015模块。

WebAssembly 向网络平台增加的基本要素：

- 代码的二进制格式；
- 加载运行该二进制代码的API。

## 如何在Web APP中使用？

WebAssembly 有两个主要的着手点：

- 使用`Emscripten`移植一个C/C++应用程序。
- 直接在汇编层，编写或生成WebAssembly代码。
- 编写Rust程序，将WebAssembly作为它的输出。