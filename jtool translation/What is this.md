## 这是什么？

jtool命令旨在满足和超越XCode的otool（1）的功能，沿着额外的Mach-O命令，如atos（1），dyldinfo（1），nm（1），segedit（1）， pagestuff（1），string（1），甚至codesign（1）和非正式的ldid。 jtool还提供了新颖的功能，如二进制搜索功能，符号注入和反汇编功能（有限但不断改进）的仿真功能。 它还提供颜色输出。 最重要的是，它可以在各种平台上运行 - *OS X，iOS甚至Linux* 。 使用任何类型都是免费的，最新版本可以在[这里](http://www.newosxbook.com/tools/jtool.tar)找到。

虽然jtool是围绕我自己的用例构建的，但其他人也发现它很有用。 它支持的选项增加了许多。 随着MOXiI 2的发布（即将推出），我终于发布jtool（几乎）1.0版本，并且以示例的形式记录了它的无数功能，其中一些可能比较难理解。 运行没有参数的jtool将显示所有的选项，我意识到这是一个恐怖的事情：

```
This is jtool v(almost 1.0) (Moscow) - with partial ARMv7k disassembly and SLC symbol resolution, compiled on Jan  8 2017 20:33:01

Usage: jtool [options] _filename_

OTool Compatible Options:
   -arch ..            	For fat (universal) binaries: i386,x86_64[h], arm[v[67]]
   -h                  	print header (ELF or Mach-O)
   -f                  	print fat header
   -l                  	List sections/commands in binary
   -L                  	List shared libraries used (like LDD)
   -v[v]               	Verbose output. -vv is even more verbose. -vvv might be more than you can handle :-)

New Options:
   --pages             	Show file page map (similar to pagestuff(1))
   -a _offset_         	Find virtual address corresponding to file offset _offset
   -e[xtract] _name_   	Extract section/segment _name_ in binary, or file _name_ from shared cache
   -e[xtract] arch     	Extract selected architecture from FAT file. Specify arch with -arch .. or ARCH=
   -e[xtract] signature	Extract code signature from binary
   -F [_string_]       	find all occurrences of _string_ in binary
      -Fs _string_     	also show search results (experimental)
   -S                  	List Symbols (like NM)
      -Sa _address_ _symbolname_	Add symbolname manually for address (to .jtool)
      -Sd _symbolname_ (not yet)	Remove symbolname for address (to .jtool)
   -p _addr_[,_size_]  	Peek at _size_ bytes in virtual _address_ in binary (like OD, but on memory)

dyldinfo Compatible Options:
   -bind               	print addresses dyld will set based on symbolic lookups
   -lazy_bind          	print symbols which dyld will lazily set on first use
   -weak_bind          	print symbols which dyld must coalesce
   -export             	print addresses of all symbols this file exports
   -opcodes            	print opcodes used to generate the rebase and binding information
   -function_starts    	print table of function start addresses
   -data_in_code       	print any data-in-code inforamtion

Destructive Options (will write output to /tmp):
   -m                  	Modify
     __SEGMENT[.__section],[_offset][,size]	(null)
   -r                  	Remove/Resize (Experimental)
     -rL _dylib/soname_      	Library
     -rC _Load_Command_#_    	Load Command
   -/+pie              	Toggle Position Independent Executable (ASLR)
   -/+lcmain           	Toggle pre-Mountain-Lion/iOS6 compatibility (LC_UNIXTHREAD/LC_MAIN)
   -/+enc              	Mark as decrypted/encrypted (toggles cryptid of LC_ENCRYPTION_INFO[64])

Disassembly/Dump Options:
   -d[_arg_[,size]]    	disassemble/dump (experimental) -  _arg_ may specify address/section/symbol/Obj-C class. size is optional
     -dA [_arg_[,size]]	Disassemble as ARM code (32-bit instructions)
     -dT [_arg_[,size]]	Disassemble as Thumb/Thumb-2 code (16/32-bit instructions)
     -dD [_arg_[,size]]	Dump (even on a text segment)
     -do [_arg_[,size]]	Dump/Disassemble from offset, rather than address
     -d objc           	Dump objective-C classes in binary, if any
   -D : As -d, but attempts to decompile only (i.e. shows only C-level code, no disassembly	(null)
   -opcodes            	Also dump opcode bytes
   --jtooldir _path_   	path to search for companion jtool files (default: $PWD).
			Use this to force create a file, if one does not exist

Code Signing Options:
   --sig               	Show code signature in binary (if any)
   --sign [adhoc]      	self-sign with no certificate
   --ident _ident_     	provides identity (fake, of course)
   --appdir            	Set App Path (for code signing and/or verification)
   --ent               	Show entitlements in binary (if any)

Advanced Options:
   --pcrelative        	show addresses as PC relative offset
   --slide  _slide_    	slide text by _slide_ bytes (may be negative)
   --rebase _address_  	rebase text to this address (destructive)
   --inplace           	Perform destructive operations in place (instead of out.bin) for the brave
   --version           	Show tool version and compilation date

Output Options:
   --html              	Output as HTML (implies color)
   --curses            	Output as Color using ncurses (can also set JCOLOR=1)

Environment variables: JDEBUG=1, NOPSUP=1 (suppress NOPs in disassembly), NOOBJC=1 (avoid Obj-C crashes)

Note: Experimental features may not be available in public version of this tool
```

所以让我们逐个去说明每一个选项。


重要提示：Jtool已经过全面的测试，但我仍然依靠你的错误报告来解决Mach-O menagerie中更多的bug。 如果你有一个JTool很难解析的二进制，或者如果它引起了JTool的崩溃，请告知我。 我不能修复我不知道的错误..遇到问题，你可以尝试使用NOOBJC = 1再次运行jtool，来禁用OBJC支持（可能还有bug）。

jtool有很多有用的功能，我不能把它们全部放在man页面中。 真的，没有人读这个人（因为你必须将jtool.1移动到/usr/share/man/man1）。 所以我最后发布了一个像样的HTML文档。 这个文档正在逐渐丰富，但是目前还没有覆盖所有功能。

## 文档的目录

- Otool兼容选项
- dyldinfo兼容选项
- 高级选项：
   - pages（获取布局）
   - -a（查找地址）
   - -S
- 代码签名选项
   - --sig
   - --ent
   - --sign
- Objective-C 支持
- 补充的一些很酷的选项

