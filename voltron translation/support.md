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
