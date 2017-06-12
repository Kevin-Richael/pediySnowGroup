## 新的选项（New Options）

#### --pages
您可能用过pagestuff命令，该命令能导出Mach-O二进制文件的内存页布局。语法略奇怪，将文件名作为第一个参数，有点粗糙。

```
$ pagestuff ~/Documents/iOS/JB/Pangu9/pguntether -a | more
File Page 0 contains fat file headers
File Page 1 contains empty space in the file between:
    fat file headers and
    Mach-O file for armv7
...
File Page 8 contains empty space in the Mach-O file for armv7 between:
    Mach-O headers and
    contents of section (__TEXT,__text)
File Page 9 contains contents of section (__TEXT,__text) (armv7)
Symbols on file page 9 virtual address 0x91f8 to 0xa000
... this little piggy went to the river...
```

jtool  --pages 选项能够提供更清晰的输出

```
# Cut to the chase:
Phontifex-Magnus:~ root# ARCH=arm64 jtool --pages /pguntether
0x0-0x38000	__TEXT
0x53e8-0x34f50	__TEXT.__text
0x34f50-0x35490	__TEXT.__stubs
0x35490-0x359e8	__TEXT.__stub_helper
0x359e8-0x36a11	__TEXT.__const
0x36a11-0x379cf	__TEXT.__cstring
0x379cf-0x37e30	__TEXT.__objc_methname
0x37e30-0x37e64	__TEXT.__gcc_except_tab
0x37e64-0x37ffc	__TEXT.__unwind_info
0x38000-0x3c000	__DATA
0x38000-0x38048	__DATA.__got
0x38048-0x383c8	__DATA.__la_symbol_ptr
0x383d0-0x38530	__DATA.__const
0x38530-0x389d0	__DATA.__cfstring
0x389d0-0x389d8	__DATA.__objc_imageinfo
0x389d8-0x38b80	__DATA.__objc_selrefs
0x38b80-0x38bd8	__DATA.__objc_classrefs
0x38bd8-0x3b99c	__DATA.__data
0x3c000-0x3f120	__LINKEDIT
0x3c000-0x3c030	Rebase Info     (opcodes)
0x3c030-0x3c2d8	Binding Info    (opcodes)
0x3c2d8-0x3cb98	Lazy Bind Info  (opcodes)
0x3cb98-0x3ce70	Exports                  
0x3ce70-0x3d008	Function Starts
0x3d008-0x3d080	Code Signature DRS
0x3d008-0x3d008	Data In Code
0x3d080-0x3dcd0	Symbol Table
0x3dcd0-0x3e074	Indirect Symbol Table
0x3e074-0x3e974	String Table
0x3e980-0x3f120	Code signature
```

#### -a


有时，您只需要快速查找某个地址在段中的偏移量，而不必在pagestuff或者jtool --pages的输出中筛选。为此jtool有一个-a选项

```
Phontifex-Magnus:~ root# ARCH=arm64 jtool -a 0x38b81 /pguntether 
Offset 38b81 in file will be loaded at 100038b81 (__DATA.__objc_classrefs)
Phontifex-Magnus:~ root# ARCH=arm64 jtool -a 0x81 /pguntether 
Requested offset 129 falls in Load Command 1 (LC_SEGMENT_64)
```

#### -S


-S选项与nm命令做同样的事情。该选项有两种形式，简单的 -S（S大写）和完整信息的-V（包括每个符号所在的dylib）。-v等同于nm中的-m。不像dyldinfo，选项不兼容。

这个选项真正有用的是快速的编写脚本，例如在多个二进制文件中寻找特定符号：


```
# If something calls SecTaskCopyValueForEntitlement, then it checks entitlements:
# (though there are other ways, like calling csops[_audittoken] directly..)
Phontifex-Magnus:/usr/libexec root# for d in /usr/libexec/* ; do  \
     if jtool -S $d 2>/dev/null | grep SecTaskCopy > /dev/null; then \
        echo $d checks entitlements...; \
     fi;  \
     done
/usr/libexec/OTATaskingAgent checks entitlements...
/usr/libexec/adid checks entitlements...
/usr/libexec/configd checks entitlements...
/usr/libexec/crash_mover checks entitlements...
/usr/libexec/demod checks entitlements...
/usr/libexec/demod_helper checks entitlements...
/usr/libexec/keybagd checks entitlements...
/usr/libexec/locationd checks entitlements...
/usr/libexec/lockbot checks entitlements...
/usr/libexec/lskdd checks entitlements...
/usr/libexec/lskdmsed checks entitlements...
/usr/libexec/mobile_obliterator checks entitlements...
/usr/libexec/mobileassetd checks entitlements...
/usr/libexec/nlcd checks entitlements...
/usr/libexec/pfd checks entitlements...
/usr/libexec/pkd checks entitlements...
/usr/libexec/rolld checks entitlements...
/usr/libexec/securityd checks entitlements...
/usr/libexec/timed checks entitlements...
/usr/libexec/transitd checks entitlements...
/usr/libexec/webinspectord checks entitlements...
```
