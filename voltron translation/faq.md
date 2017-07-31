文档：
见github上的wiki

FAQ
Q.为什么我加载Voltron的时候遇到了ImportError
A.您可能安装了多个版本的Python，并使用了错误的版本安装了Voltron。 请参阅更详细的安装说明。

Q.GET?PEDA?PwdDbg?fG's gdbinit?
A.像Voltron这种所有牛逼的GDB扩展都提供了附加命令集，还提供了寄存器，堆栈，代码等视图。这些工具都会在调试器中断时显示这些视图。Voltron采用不同的方法，将RPC服务器嵌入到调试器中，并允许从其他终端查看附加视图（或甚至Web浏览器或现在与二进制），从而就能让用户构建更清爽的多窗口界面。Voltron与所有这些工具配合很好。 您只需禁用GDB扩展选项中的上下文显示，并连接一些Voltron视图，同时仍然可以获得这些工具添加的有用命令的所有好处。