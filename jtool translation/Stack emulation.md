## 堆栈模拟 

函数的堆栈模拟我已经推迟了很长时间，但我觉得现在是时候了。按照STR/STP 我研究了相当长一阶段：jtool现在可以动态检测blocks，让你知道哪个函数包含它。
例如，dispatch_[a]sync or xpc_connection_set_event_handler:


![Alt text](./jtoolblocks.png)

更妙的是，它可以动态跟踪mach_msg构造。这真的非常有用，我的“MOXiI Vol.1”的@TODO之一是记录所有的SpringBoard，Backboard和其他进程的MIG。你可以看到这个节省了多少时间呢？

```
# Note that:
# A) you need to disable color for the regexp to work (because of curses sequences)
# B) the regexp is then "mach_msg(" (decompiled function) or "begins with _" (function label)
# C) jtool does everything in the cache, no need to extract!
#
morpheus@Zephyr (~/Documents/Work/JTool) % JCOLOR=0 jtool -d dyld_shared_cache_arm64:BackBoardServices |
|                                           egrep "(_mach_msg\(|^_)" | less
_BKSRestartActionOptionsDescription: # No mach_msg here 
_BKSTouchDeliveryPolicyServerGetProxyWithErrorHandler: # No mach_msg here either..
__BKSHIDGetBacklightFactor:
; _mach_msg(6000000)
__BKSHIDSetBacklightFactorPending:
; _mach_msg(6000001)
__BKSHIDSetBacklightFactorWithFadeDuration:
; _mach_msg(6000002)
__BKSHIDSetBacklightFactorWithFadeDurationAsync:
; _mach_msg(6000003)
..
morpheus@Zephyr (~/Documents/Work/JTool) % jtool -d dyld_shared_cache_arm64:BackBoardServices |
                                            egrep "(_mach_msg\(|^_)"       
_BKSRestartActionOptionsDescription: 
_BKSTouchDeliveryPolicyServerGetProxyWithErrorHandler: 
__BKSHIDGetBacklightFactor:
; _mach_msg(6000000)
__BKSHIDSetBacklightFactorPending:
; _mach_msg(6000001)
__BKSHIDSetBacklightFactorWithFadeDuration:
; _mach_msg(6000002)
...
__BKSHIDSetHardwareKeyboardLayout:
; _mach_msg(6000056)
__BKSHIDGetHardwareKeyboardLanguage:
; _mach_msg(6000057)
__BKSHIDSetEventRouters:
; _mach_msg(0)  # OK, so it's not perfect -- I don't follow FP operations (yet)!
__BKSHIDSetKeyCommands:
; _mach_msg(6000059)
__BKSHIDSetStackshotCombos:
; _mach_msg(6000059)
__BKSHIDSetTouchHand:
; _mach_msg(6000061)
__BKSDisplayStart:
; _mach_msg(6001000)
__BKSDisplayIsDisabled:
; _mach_msg(6001001)
..
```
## 反编译优化 ##

在我可以支持任意函数定义之前，我提供了在关联文件中定义一个后缀的选项:如下所示

![Alt text](./jtooldecomp1.png)

这样，jtool不仅会跟随参数，而且会在调用函数时将其显示出来。当前支持的类型是CFString，cstring(它会得到错误消息、反馈和日志功能）

还有很多，不过我以后再来解释。如果您使用jtool并发现bug，请不要只是默默地抱怨——让我知道，我很乐意修复。同样，如果你有任何改进的建议。记住-我在我自己的用例构建了这个功能- 如果你有其他的，我会很高兴的调整。
我不介意你使用IDA或IDA可以这样做(如果他们不能，他们会在他们的下一个版本中采用这个)或者是IDAPython，或者别的什么。这是我使用的工具，我发现它很有用，并且我鼓励其他人使用它(免费地，不像ida/hopper和他们的同类)，并给出改进建议。

即将涉及更多：
- 汇编
-  关联文件
- jtool脚本
-  定制与machlib接口的代码
-  反编译功能
