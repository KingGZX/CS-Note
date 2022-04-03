## 1. IP Protocol



## 2. Lab0

一开始是带你熟悉了一下Linux底下的一些常用网络命令。

用到了telnet、netcat之类的，按照提示在终端键入命令，运行，并不会有什么问题。

[lab0 instructions](https://cs144.github.io/assignments/lab0.pdf)

代码部分的内容是写一个可以获取网页的小程序，语言是C++，需要对Linux操作系统有一定的了解。首先根据提示搭好环境，预编译一些源代码：

![1643572569676](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643572569676.png)

然后作业有一些比较重要的要求，尽量去避免C风格的代码等，仔细看完其实就是**effective C++**笔记中的一些重要条款。除了使用C++，要学习使用Git，可以上B站学或者在pdf给出的网站中学。

因为我们现在不当API调用者了，要自己逐步实现lab最后搭建一套自己的TCP协议栈，我们要通读或者说必须熟悉每个lab必要的那些头文件。要求我们的是 文件描述符.hh、socket.hh、地址.hh。

![1643573021176](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643573021176.png)