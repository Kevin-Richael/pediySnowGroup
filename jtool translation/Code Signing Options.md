##<font color=green>代码签名选项(Code Signing Options)</font>
代码签名选项是JTool的第二大功能。对于iOS通过代码签名和授权文件的安全方案来说，重要的是要有一种方法来快速确定二进制文件拥有的授权以及签名的方式。OS X已经有了这样的工具，但是我发现它粗糙的多，而且还没有什么端口供iOS使用，因此jtool这样的工具就很有必要了。


#### --sig

--sig选项允许您验证或显示代码签名。无参数的情况下，它会显示签名相关的信息，并执行无提示验证：
```
Phontifex-Magnus:~ root# jtool --sig /System/Library/CoreServices/SpringBoard.app/SpringBoard
Blob at offset: 7453808 (47168 bytes) is an embedded signature
	Code Directory (36570 bytes)
		Version:     20100 					# 20100 or the older 20001
		Flags:       adhoc					# adhoc (iOS System) or none
		CodeLimit:   0x71bc70					# Code signature maximum reach
		Identifier:  com.apple.springboard (0x30)		# App/Bundle identifier
		CDHash:	     8dcf2b2e1839d86068672ac51386d6f6692eb7c4	# Code Directory Hash (calculated)
		# of Hashes: 1820 code + 5 special			# Code and special slots
		Hashes @170 size: 20 Type: SHA-1			# Offset of Hash slot 0
 Empty requirement set (12 bytes)					# Blob 1
Entitlements (10515 bytes) (use --ent to view)				# Blob 2
Blob Wrapper (8 bytes) (0x10000 is CMS (RFC3852) signature)		# Blob 3
```

和其他工具一样，jtool会更详细地输出关于子区块（特别是偏移）的更多细节以及代码段( CodeDirectory )中的每个页哈希（如果有的话，还会显示特殊插槽(slots)信息)：

```
Phontifex-Magnus:~ root# jtool -v --sig /System/Library/CoreServices/SpringBoard.app/SpringBoard 
Blob at offset: 7453808 (47168 bytes) is an embedded signature of 47149 bytes, and 4 blobs
	Blob 0: Type: 0 @44: Code Directory (36570 bytes)
		Version:     20100
		Flags:       adhoc (0x2)
		CodeLimit:   0x71bc70
		Identifier:  com.apple.springboard (0x30)
		CDHash:	     8dcf2b2e1839d86068672ac51386d6f6692eb7c4
		# of Hashes: 1820 code + 5 special
		Hashes @170 size: 20 Type: SHA-1
			Entitlements blob:	52a9a55ab0c40a12e3a915bcea29d602419a21f8 (OK)
			Application Specific:	Not Bound
			Resource Directory:	c0c8a907e2663ee1dc33a137fbfb55f253bcf464 (OK)
			Requirements blob:	3a75f6db058529148e14dd7ea1b4729cc09ec973 (OK)
			Bound Info.plist:	f852d93c0a643f9d774e7d2057664d769d96d528 (OK)
			Slot   0 (File page @0x0000):	34f05f5bc893e54b69c7a3f87137f86f49a09e5d (OK)
			Slot   1 (File page @0x1000):	e515677a23b37411199258e68da4a03cd184ffae (OK)
			....
			Slot 1818 (File page @0x71a000):	e563b1335082da705a949c7c62c15c6da47876b2 (OK)
			Slot 1819 (File page @0x71b000):	496567ae0e21a025eccb859c9d2e20bacb88075d (OK)
	Blob 1: Type: 2 @36614:  Empty requirement set (12 bytes)
	Blob 2: Type: 5 @36626: Entitlements (10515 bytes) (use --ent to view)
	Blob 3: Type: 10000 @47141: Blob Wrapper (8 bytes) (0x10000 is CMS (RFC3852) signature)
```

jtool通常会自动找到特殊的槽位资源，但是如果没有，你可能需要手工指定 --appdir


#### --ent

如果你想检查一个二进制文件所声明的权限，那么你需要这个工具。和其他工具相同但更准确的是，jtool也会解析区块(blob)头：
```
Phontifex-Magnus:~ root# jtool  --ent /System/Library/CoreServices/SpringBoard.app/SpringBoard 
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>allow-obliterate-device</key>
	<true/>
	# ....
	<key>keychain-access-groups</key>
	<array>
		<string>apple</string>
		<string>com.apple.preferences</string>
	</array>
	<key>vm-pressure-level</key>
	<true/>
</dict>
</plist>
```
再次提醒下，你需要记住，jtool是用shell脚本设计的，所以你也可以做一个或者其他工具来获得所有那些苹果在使用的但是没有文档化的权限(entitlements)。（作者正在整理这些到数据库中，完成后会公开分享出来 --译者注）

####--sign
该选项是我在内部已经使用很长时间的功能，最终才发布出来。jtool是用和Saurik's相同的方式来生成自签名二进制文件。因此，您可以指定要插入或者使用的权限文件。该工具可能导致未知的结果，一旦你使用它，就代表了你同意这个临时警告/免责声明：
```
morpheus@Zephyr (~)$  jtool --sign binary
  **************************************************************************************
  * Warning: Code signatures are still defined as Beta. Lots of minutiae to deal with, *
  * and it isn't as easy as you might think to get things right with all these hashes. *
  *                                                                                    *
  * I suggest you use --sig -v to validate your pseudo-signed binaries.                *
  * Try JDEBUG=1 if you want to follow along.                                          *
  **************************************************************************************
Warning: Destructive option. Output (397920 bytes) written to out.bin
morpheus@Zephyr ()$  ldid -S binary
morpheus@Zephyr ()$  ls -l binary out.bin
-rwxr-xr-x  1 root  staff  397920 Oct 18 03:30 binary
-rwxr-xr-x  1 root  staff  397920 Oct 18 03:30 out.bin
morpheus@Zephyr ()$  diff out.bin binary

morpheus@Zephyr ()$  echo $?
0
```
为什么会有这个警告，并且默认输出的是out.bin文件而不是替换原来的文件,原因很简单: 代码签名是有风险的，如果丢失原来的文件，很难甚至是不可恢复的。
你如果不使用内置的命令codesign(1) 或者codesign_allocate(1)，而是使用```JDEBUG=1```命令,jtool就会显示整个流程一步一步地执行的过程，你就可以看到代码签名是如何实现的：

```
morpheus@Zephyr (~)$ JDEBUG=1 ARCH=armv7 jtool  --sign --ent ent.xml  /tmp/a
Very last section ends at 0xc11c, so that's where the code signature will be
Aligning to 16 byte offset - 0xc120
Allocating Load Command
First section offset is 7ea4; Mach header size is 580
Patching header to reflect inserted command @580
Patching __LINKEDIT to reflect new size of file
Setting LC fields
Allocating code signature superblob of 669 bytes, with 3 sub-blobs..
Setting LC_CODE_SIGNATURE's blob size to match CS Blob size..
Creating Code Directory with 13 code slots and 5 special slots
Calculating Hashes to fill code slots..
Need to pad 288 bytes to page size in last page (because code signature is also in this page)
Padding to page size with 3808 bytes
Calculating (modified) last page hash
Adding empty requirements set to 447
Filling the special slot (-2) for requirements blob...
 Copying entitlements blob to 459
Filling the special slot (-5) for entitlement blob...
 Crafting New Mach-O
Inserting 669 bytes Blob at 49440, bringing new file size to 50109
Warning: Destructive option. Output (50109 bytes) written to out.bin
```