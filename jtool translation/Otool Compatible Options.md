## Otool 可用的选项

当我使用jtool时，我认为我可以使jtool替代otool,所以一些选项实际上是一样的。这些参数包括-L(显示依赖的库，和ldd相似)，-f(获取FAT头)。其他参数兼容，但有些区别。例如：-h Mach-O 头，会打印出很多详细信息，不会只是一行：

	Zephyr:~ morpheus$ otool -h /bin/ls
	/bin/ls:
	Mach header
	      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
	 0xfeedfacf 16777223          3  0x80           2    19       1816 0x00200085
	
	Zephyr:~ morpheus$ jtool -h  /bin/ls
	Magic:	64-bit Mach-O
	Type:	executable
	CPU:	x86_64
	Cmds:	19
	size:	1816 bytes
	Flags:	0x200085
	


我更偏爱jtool的原因就是它对grep的支持更友好，所以你能分离出包含你想要属性的行。在最常见的选项-l，然而，他只是将otool的多行输出变得行数更少。为了使他更友好：


	$ otool -l /bin/ls                $ jtool -l /bin/ls
	/bin/ls:
	Load command 0                    LC 00: LC_SEGMENT_64          Mem: 0x000000000-0x100000000      __PAGEZERO
	      cmd LC_SEGMENT_64           LC 01: LC_SEGMENT_64          Mem: 0x100000000-0x100005000      __TEXT
	  cmdsize 72                             Mem: 0x100004430-0x100004604            __TEXT.__stubs  (Symbol Stubs)
	  segname __PAGEZERO                     Mem: 0x100004430-0x100004604            __TEXT.__stubs  (Symbol Stubs)
	   vmaddr 0x0000000000000000             Mem: 0x100004920-0x100004b10            __TEXT.__const  
	   vmsize 0x0000000100000000             Mem: 0x100004b10-0x100004f66            __TEXT.__cstring        (C-String Literals)               
	  fileoff 0                              Mem: 0x100004430-0x100004604            __TEXT.__stubs  (Symbol Stubs)
	 filesize 0                        LC 02: LC_SEGMENT_64          Mem: 0x100005000-0x100006000      __DATA
	  maxprot 0x00000000                         ....
	 initprot 0x00000000
	   nsects 0
	    flags 0x0
	Load command 1
	      cmd LC_SEGMENT_64
	  cmdsize 552
	  segname __TEXT
	   vmaddr 0x0000000100000000
	   vmsize 0x0000000000005000
	  fileoff 0
	 filesize 20480
	  maxprot 0x00000007
	 initprot 0x00000005
	   nsects 6
	    flags 0x0



otools的输出简单的可怕，因为它是稀疏地分散到多个行。jtool聚集这些信息使你更清楚的看到那些重要信息，并减少了滚屏的页面。另一个好处是使用grep：


	# Finding details about a segment with otool(1):
	Zephyr:~ morpheus$ otool -l /bin/ls | grep LC_SEGM
	      cmd LC_SEGMENT_64
	      cmd LC_SEGMENT_64
	      cmd LC_SEGMENT_64
	      cmd LC_SEGMENT_64
	# Really helpful, right? Yes, you could do that with grep -A, but I find this simpler: 
	Zephyr:~ morpheus$ jtool -l -v /bin/ls | grep LC_SEGM
	LC 00: LC_SEGMENT_64          Mem: 0x000000000-0x100000000	File: Not Mapped	---/---	__PAGEZERO
	LC 01: LC_SEGMENT_64          Mem: 0x100000000-0x100005000	File: 0x0-0x5000	r-x/rwx	__TEXT
	LC 02: LC_SEGMENT_64          Mem: 0x100005000-0x100006000	File: 0x5000-0x6000	rw-/rwx	__DATA
	LC 03: LC_SEGMENT_64          Mem: 0x100006000-0x100009000	File: 0x6000-0x8750	r--/rwx	__LINKEDIT
	

注意上述使用了那个冗长的（LC_SEGMENT 命令）-v。

一般的（FAT）文件，JTool提供相同的-arch 开关来选择详细的架构。有几种不同otool：


- jtool拒绝接触一个fat二进制文件，除非你指定架构
- 你可以用 -arch 指定架构，也可以使用环境变量  ARCH= 。所以设置默认值非常有用。
- 可以用数字指定架构

毕竟最后一个选项貌似不常见，FAT能有多少种架构？嗯，好吧，我也思考过这个问题，直到太极的8.4越狱出现，不仅出现了26-27的架构，而且同时使用，因为otool只能匹配第一个，导致otool会混乱。但jtool可以指定一个架构，避免这种错误。（我的文章里有举例）