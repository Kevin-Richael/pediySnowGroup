##安装
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
