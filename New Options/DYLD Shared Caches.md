
## DYLD共享缓存（DYLD Shared Caches）


苹果将​​大多数dylibs和插件预先链接到“Shared Library Cache”中。 SLC位于 /var/db/dyld（OS X）和 /System/Library/Caches/com.apple.dyld（iOS）中。 OS X 缓存也有一个“map”，但是iOS没有。

在iOS中，没有单独的dylib，它们都在缓存中。这使得缓存非常重要，并且有许多的“提取”工具。 JTool不提供全面的支持，只提供我觉得有用的东西。尤其是：

```
# Identifying a shared cache file
Phontifex-Magnus:~ root# jtool  /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
File is a shared cache containing 1007 images (use -l to list)
# Finding the map of a shared cache file (formerly, --map which isn't needed anymore
Phontifex-Magnus:~ root# jtool -l /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
File is a shared cache containing 1007 images
   0:        180028000 /System/Library/AccessibilityBundles/AXSpeechImplementation.bundle/AXSpeechImplementation
   1:        180030000 /System/Library/AccessibilityBundles/AccessibilitySettingsLoader.bundle/AccessibilitySettingsLoader
   2:        18003c000 /System/Library/AccessibilityBundles/AccountsUI.axbundle/AccountsUI
   3:        180040000 /System/Library/AccessibilityBundles/AddressBookUIFramework.axbundle/AddressBookUIFramework
   4:        180048000 /System/Library/AccessibilityBundles/CameraKit.axbundle/CameraKit
   5:        180058000 /System/Library/AccessibilityBundles/CameraUI.axbundle/CameraUI
   6:        180064000 /System/Library/AccessibilityBundles/HearingAidUIServer.axuiservice/HearingAidUIServer
   7:        180074000 /System/Library/AccessibilityBundles/MapKitFramework.axbundle/MapKitFramework
   8:        180080000 /System/Library/AccessibilityBundles/MediaPlayerFramework.axbundle/MediaPlayerFramework
...
# And the -h (header) option, I added after Pangu 9. See anyhing different, boys and girls? *hint* *hint*
Phontifex-Magnus:~ root# jtool -h /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64           
File is a shared cache containing 1007 images (use -l to list)
6 mappings starting from 152, Mapping Count: 1007. 344 Images starting from 0
DYLD base address: 0, Code Signature Address: 25a4c000 (2f0f02 bytes)
Slide info: 1ca18000 (1a4000 bytes)
Local Symbols: 204c8000 (5584000 bytes)
mapping r--/r--    0MB        180000000 -> 180028000         (25d40000-25d68000)
mapping r-x/r-x  384MB        180028000 -> 1980a4000         (28000-180a4000)
mapping rw-/rw-   73MB        19a0a4000 -> 19ea18000         (180a4000-1ca18000)
mapping r--/r--   12MB        1a0a18000 -> 1a16b0000         (1ca18000-1d6b0000)
mapping r--/r--    0MB        1a16b0000 -> 1a16b4000         (25d68000-25d6c000)
mapping r--/r--   46MB        1a16b4000 -> 1a44c8000         (1d6b4000-204c8000)
```

Jtool可以轻松地解压cache文件，只需使用 -e dylib 选项即可提取需要的dylib。请注意，提取的dylib将大于50MB，因为JTool不会解构被合并的__LINKEDIT段。

但是为什么需要提取dylib呢??? Jtool的一个关键功能是其他decachers没有的，就是它对于cache中的dylib也是有效的！这不仅可以节省您的磁盘空间，而且还可以让您看到缓存中的dylibs与其他dylib的关系（例如跨dylib调用）。要使用此功能，只需将":"作为cache和dylib名称之间的分隔符。所有的功能（例如 -l, -S等）都能正常工作，但真正有用的功能是-d。例如：

```
root@phontifexMagnus (~)# jtool -d /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64:libMobileGestalt | more
Found but not extracting - setting File Start to 16a10000
Disassembling from file offset 0x11a4, Address 0x196a111a4 
Processing cached file from offset 16a38000  size: 25d44000
_MGSetLogHandler:
   196a111a4    ADRP   x8, 32640                ; ->R8 = 0x19e991000 
   196a111a8    STR    X0, [X8, #280]           ; *0x19e991118 = X0 ARG0
   196a111ac    RET                     
_func_196a111b0:
   196a111b0    ORR    W1, WZR, #0x1            ; ->R1 = 0x1 
   196a111b4    B      _func_196a111b8  ; 196a111b8
..
```