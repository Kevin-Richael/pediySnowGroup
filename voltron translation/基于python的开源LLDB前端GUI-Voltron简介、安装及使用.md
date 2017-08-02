# 基于python的开源LLDB前端GUI-Voltron简介、安装及使用

## Voltron
Voltron是一个可拓展的调试器UI工具包。用Python编写，它的目的是改善各种调试器（-iob、GDS、VDB和WinDbg）的用户体验，通过启用能够从调试器主机中检索和显示数据的使用程序视图。通过在其它TTYS中运行这些视图，您可以构建一个定制的调试器用户界面来满足您的需求。

Voltron并不打算成为所有人的一切，它不是对调试器的CLI的批量替换。相反，它的目的是对现有的设置进行补充，并允许您尽可能多地拓展CLI调试器。如果您只是想在调试器旁边的窗口中看到一个窗口，那么您可以这样做。如果你想把所有的东西都拿出来，让它看起来更像DllyDbg，你也可以这样做。

## 内置视图展示：

 * 寄存器
 * 反汇编
 * 堆栈
 * 内存
 * 断点
 * 调用堆栈回溯

作者的设置看起来像这样：

![](https://camo.githubusercontent.com/f364ec6565b14e266e33005c9cb80e5c7f7b367d/687474703a2f2f692e696d6775722e636f6d2f396e756b7a74412e706e67)



任何调试器命令可以分为视图和高亮指定的pygments lexer：
![](https://camo.githubusercontent.com/2d3d1fc22454a754a356ac1e8b0b2a098ec55587/687474703a2f2f692e696d6775722e636f6d2f526259515958702e706e67)

更多截图<a href="https://github.com/snare/voltron/wiki/Screenshots">戳这里</a>

## 支持
Voltron支持LLDB, GDB, VDB及WinDbg/CDB(via [PyKD](https://pykd.codeplex.com/ "PyKD")) 等调试器，并且可以运行在macOS、Linux及Windows上。
对WinDbg的支持刚开始，所以如果使用过程中遇到什么问题，请[开个issue](https://github.com/snare/voltron/issues)。

Voltron支持如下的处理器架构：

| |lldb|gdb|vdb|windbg|
|:-|:-:|:-:|:-:|:-:|
|x86|✓|✓|✓|✓|
|x86_64|✓|✓|✓|✓|
|arm|✓|✓|✓|✗|
|arm64|✓|✗|✗|✗|
|powerpc|✗|✓|✗|✗|

## 安装
记住：只有macOS系统和Debian衍生系统是完全支持安装脚本。它应该不会在其他Linux发行版上失败，但是它不尝试安装软件包的依赖关系。如果你在用另一个发行版，去看看install.sh，在运行前你需要安装哪些依赖。

下载源代码并安装脚本：
>$ git clone https://github.com/snare/voltron
>
>$ cd voltron
>
>$ ./install.sh

默认情况下，安装脚本将安装在用户的site—packages文件夹。如果你想安装到系统site—packages下，使用-s 标识:
>$ ./install.sh -s

你也可以安装到虚拟环境（仅支持LLDB）像这样：
>$ ./install.sh -v /path/to/venv -b lldb

如果你在Windows上没有shell，安装有问题，或更希望手动安装，请参考[手动安装文档](https://github.com/snare/voltron/wiki/Installation).

## 速成指南

1. 如果你的调试器支持初始化脚本(lldb是.lldbinit，gdb是.gdbinit)， 把它设置成在里面执行entry.py脚本， 来保证每次启动载入Vultron。
完整的路径在voltron包里。 举个例子， 在macOS里它可能是/Library/Python/2.7/site-packages/voltron/entry.py。
install.sh脚本执行时如果在当前路径里探测到了GDB或者LLDB会自动把它加进去。

 LLDB:
 >command script import /path/to/voltron/entry.py

 GDB:
 >source /path/to/voltron/entry.py

2. 必要的话，手动启动你的调试器，然后初始化Voltron。

  比较新的LLDB版本里不需要手动初始化Voltron:

  >$ lldb target_binary

  老点的版本里你就得在载入调试对象后手动调用voltron init了:

  >$ lldb target_binary
  >(lldb) voltron init

 VDB:

  >$ ./vdbbin target_binary

  >script /path/to/voltron/entry.py

WinDbg/CDB仅通过Bash On Windows支持. 作者已通过 Git Bash 和 ConEmu进行了测试. PyKD可以在启动调试器时一并启动Voltron:

 >$ cdb -c '.load C:\path\to\pykd.pyd ; !py --global C:\path\to\voltron\entry.py' target_binary

3.在另一个终端（我用iTerm panes）里启动任一UI视图. 在LLDB, WinDbg和GDB这几个调试器下这几个视图会立刻更新.
在VDB下它们需要等到下个更新点 (跑到断点, 走完一步, 等等)才会更新。

  >$ voltron view register

  >$ voltron view stack

  >$ voltron view disasm

  >$ voltron view backtrace

4.设个断点，开动.
 >(*db) b main
 >(*db) run

5.断点被触发时， 视图会被更新到寄存器/堆栈/内存/等等当前的状态。 调试器命令行里的每个指令都会利用调试器的「暂停挂钩」更新视图。
所以每次你步进，或者触发断点，视图都会更新。

## 文档：
见github上的wiki

## FAQ
Q.为什么我加载Voltron的时候遇到了ImportError
A.您可能安装了多个版本的Python，并使用了错误的版本安装了Voltron。 请参阅更详细的安装说明。

Q.GET?PEDA?PwdDbg?fG's gdbinit?
A.像Voltron这种所有牛逼的GDB扩展都提供了附加命令集，还提供了寄存器，堆栈，代码等视图。这些工具都会在调试器中断时显示这些视图。Voltron采用不同的方法，将RPC服务器嵌入到调试器中，并允许从其他终端查看附加视图（或甚至Web浏览器或现在与二进制），从而就能让用户构建更清爽的多窗口界面。Voltron与所有这些工具配合很好。 您只需禁用GDB扩展选项中的上下文显示，并连接一些Voltron视图，同时仍然可以获得这些工具添加的有用命令的所有好处。

## BUG以及勘误表

在Github上查看问题跟踪[issue tracker](https://github.com/snare/voltron/issues)，或者提交问题 .

如果你在运行Voltron时遇到加载错误，首先请确保你按照正确的平台[安装说明](https://github.com/snare/voltron/wiki/Installation)做了
## LLDB

在旧版本的LLDB，Voltron init命令必须在加载调试目标后手动运行。就好比目标程序必须在Voltron Hook安装前载入。Voltron 将会尝试自动注册它的事件处理程序，并在需要voltron init时通知用户。
## WinDbg

更多关于 WinDbg/CDB 的支持信息，请[戳我](https://github.com/snare/voltron/wiki/Installation#windbg)
## Misc

作者主要在MacOS最新版本的LLDB上使用Voltron。在发布之前，我们将尽可能在更多的平台和架构上测试，但LLDB / MacOS / x64将是迄今为止最常用的组合。希望Voltron不会让你失望！。
## License

查看许可文件[License](https://github.com/snare/voltron/blob/master/LICENSE).

如果你在使用Voltron，并且你不讨厌的话，请支持我吧，该许可证还延伸到其他贡献者--[richo](https://github.com/richo)，他值得我们的支持。
## Credits

感谢我以前的雇主 Assurance and Azimuth Security 让我有时间在这方面工作。

支持[richo](https://github.com/richo)为Voltron的所有贡献。

最初的项目灵感[fG](https://github.com/gdbinit)!

感谢[Willi](https://github.com/williballenthin)提供VDB的支持。

Voltron现在使用[Capstone](http://www.capstone-engine.org/)进行分析，就好像调试程序内部分析工具。Capstone是一个功能强大、开源、多结构反汇编程序的逆向工程和下一代的调试工具。

感谢[grazfather](https://github.com/grazfather)一直以来的帮助。
