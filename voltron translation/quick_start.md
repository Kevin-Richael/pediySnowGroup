速成指南

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
所以每次你step，或者触发断点，视图都会更新。
