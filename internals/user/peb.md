# System wide list of module list

`dt poi(nt!PsLoadedModuleList) nt!_LDR_DATA_TABLE_ENTRY -l InLoadOrderLinks.Flink DllBase FullDllName`

Instead of poi(nt!PsLoadedModuleList) we can use 0x77b61c6c as well found later.

# PEB

```
0: kd> r @$peb
$peb=03059000

0: kd> dt ntdll!_PEB @$peb -y ldr
   +0x00c Ldr : 0x77b61c60 _PEB_LDR_DATA

0: kd> dx -id 0,0,ad327040 -r1 ((ntdll!_PEB_LDR_DATA *)0x77b61c60)
((ntdll!_PEB_LDR_DATA *)0x77b61c60)                 : 0x77b61c60 [Type: _PEB_LDR_DATA *]
    [+0x000] Length           : 0x30 [Type: unsigned long]
    [+0x004] Initialized      : 0x1 [Type: unsigned char]
    [+0x008] SsHandle         : 0x0 [Type: void *]
    [+0x00c] InLoadOrderModuleList [Type: _LIST_ENTRY]
    [+0x014] InMemoryOrderModuleList [Type: _LIST_ENTRY]
    [+0x01c] InInitializationOrderModuleList [Type: _LIST_ENTRY]
    [+0x024] EntryInProgress  : 0x0 [Type: void *]
    [+0x028] ShutdownInProgress : 0x0 [Type: unsigned char]
    [+0x02c] ShutdownThreadId : 0x0 [Type: void *]
```

# Grab head of InLoadOrderModuleList

```
0: kd> dx -id 0,0,ad327040 -r1 (*((ntdll!_LIST_ENTRY *)0x77b61c6c)) <---- Head of InLoadOrderModuleList
(*((ntdll!_LIST_ENTRY *)0x77b61c6c))                 [Type: _LIST_ENTRY]
    [+0x000] Flink            : 0x32d1d30 [Type: _LIST_ENTRY *] <--- Points inside a strcuture of type _LDR_DATA_TABLE_ENTRY
    [+0x004] Blink            : 0x3315b58 [Type: _LIST_ENTRY *]

```

# Next grab pointer to structure named _LDR_DATA_TABLE_ENTRY

```
0: kd> dt ntdll!_LDR_DATA_TABLE_ENTRY
   +0x000 InLoadOrderLinks : _LIST_ENTRY
   +0x008 InMemoryOrderLinks : _LIST_ENTRY
   +0x010 InInitializationOrderLinks : _LIST_ENTRY
   +0x018 DllBase          : Ptr32 Void
   +0x01c EntryPoint       : Ptr32 Void
   +0x020 SizeOfImage      : Uint4B
   +0x024 FullDllName      : _UNICODE_STRING
   +0x02c BaseDllName      : _UNICODE_STRING
(truncated)

0: kd> dx -id 0,0,ad327040 -r1 ((ntdll!_PEB_LDR_DATA *)0x77b61c60)
((ntdll!_PEB_LDR_DATA *)0x77b61c60)                 : 0x77b61c60 [Type: _PEB_LDR_DATA *]
    [+0x000] Length           : 0x30 [Type: unsigned long]
    [+0x004] Initialized      : 0x1 [Type: unsigned char]
    [+0x008] SsHandle         : 0x0 [Type: void *]
    [+0x00c] InLoadOrderModuleList [Type: _LIST_ENTRY]
    [+0x014] InMemoryOrderModuleList [Type: _LIST_ENTRY]
    [+0x01c] InInitializationOrderModuleList [Type: _LIST_ENTRY]
    [+0x024] EntryInProgress  : 0x0 [Type: void *]
    [+0x028] ShutdownInProgress : 0x0 [Type: unsigned char]
    [+0x02c] ShutdownThreadId : 0x0 [Type: void *]

0: kd> dx -id 0,0,ad327040 -r1 (*((ntdll!_LIST_ENTRY *)0x77b61c6c))
(*((ntdll!_LIST_ENTRY *)0x77b61c6c))                 [Type: _LIST_ENTRY]
    [+0x000] Flink            : 0x32d1d30 [Type: _LIST_ENTRY *]
    [+0x004] Blink            : 0x3315b58 [Type: _LIST_ENTRY *]

0: kd> dx -id 0,0,ad327040 -r1 (*((ntdll!_LIST_ENTRY *)0x77b61c6c))
(*((ntdll!_LIST_ENTRY *)0x77b61c6c))                 [Type: _LIST_ENTRY]
    [+0x000] Flink            : 0x32d1d30 [Type: _LIST_ENTRY *]
    [+0x004] Blink            : 0x3315b58 [Type: _LIST_ENTRY *]

```

# Finally use LDR_LDR_DATA_TABLE_ENTRY's member variable InLoadOrderLinks

>InLoadOrderLinks is of type LIST_ENTRY to  traverse the chain/linklist of such entries. 
> Dumping relevent information like dllname, entrypoint etc.

```
0: kd> dt 0x77b61c6c ntdll!_LDR_DATA_TABLE_ENTRY -l InLoadOrderLinks.Flink DllBase EntryPoint SizeOfImage FullDllName LoadReason
InLoadOrderLinks.Flink at 0x77b61c6c
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d1d30 - 0x3315b58 ]
   +0x018 DllBase     : (null)
   +0x01c EntryPoint  : (null)
   +0x020 SizeOfImage : 0
   +0x024 FullDllName : _UNICODE_STRING ""
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d1d30
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d1c48 - 0x77b61c6c ]
   +0x018 DllBase     : 0x00310000 Void
   +0x01c EntryPoint  : 0x00333510 Void
   +0x020 SizeOfImage : 0x30000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\system32\notepad.exe"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x032d1c48
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d2150 - 0x32d1d30 ]
   +0x018 DllBase     : 0x77a40000 Void
   +0x01c EntryPoint  : (null)
   +0x020 SizeOfImage : 0x19e000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\SYSTEM32\ntdll.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d2150
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d24c0 - 0x32d1c48 ]
   +0x018 DllBase     : 0x77470000 Void
   +0x01c EntryPoint  : 0x7748d030 Void
   +0x020 SizeOfImage : 0x9b000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\KERNEL32.DLL"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x032d24c0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d2e38 - 0x32d2150 ]
   +0x018 DllBase     : 0x75eb0000 Void
   +0x01c EntryPoint  : 0x75fa42f0 Void
   +0x020 SizeOfImage : 0x212000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\KERNELBASE.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d2e38
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d3048 - 0x32d24c0 ]
   +0x018 DllBase     : 0x76150000 Void
   +0x01c EntryPoint  : 0x76155d30 Void
   +0x020 SizeOfImage : 0x23000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\GDI32.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d3048
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d3248 - 0x32d2e38 ]
   +0x018 DllBase     : 0x75b60000 Void
   +0x01c EntryPoint  : (null)
   +0x020 SizeOfImage : 0x1d000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\win32u.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d3248
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d34d8 - 0x32d3048 ]
   +0x018 DllBase     : 0x75dd0000 Void
   +0x01c EntryPoint  : 0x75e38f10 Void
   +0x020 SizeOfImage : 0xdb000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\gdi32full.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d34d8
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d3750 - 0x32d3248 ]
   +0x018 DllBase     : 0x75ac0000 Void
   +0x01c EntryPoint  : 0x75ad7800 Void
   +0x020 SizeOfImage : 0x7b000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\msvcp_win.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d3750
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d39d0 - 0x32d34d8 ]
   +0x018 DllBase     : 0x75b80000 Void
   +0x01c EntryPoint  : 0x75baba30 Void
   +0x020 SizeOfImage : 0x120000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\ucrtbase.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d39d0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d6b58 - 0x32d3750 ]
   +0x018 DllBase     : 0x76180000 Void
   +0x01c EntryPoint  : 0x7618c4d0 Void
   +0x020 SizeOfImage : 0x17a000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\USER32.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d6b58
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d73c8 - 0x32d39d0 ]
   +0x018 DllBase     : 0x76910000 Void
   +0x01c EntryPoint  : 0x76a4c090 Void
   +0x020 SizeOfImage : 0x281000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\combase.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d73c8
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d7630 - 0x32d6b58 ]
   +0x018 DllBase     : 0x764f0000 Void
   +0x01c EntryPoint  : 0x7653ff70 Void
   +0x020 SizeOfImage : 0xc6000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\RPCRT4.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d7630
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d78b0 - 0x32d73c8 ]
   +0x018 DllBase     : 0x763f0000 Void
   +0x01c EntryPoint  : 0x76432b30 Void
   +0x020 SizeOfImage : 0x87000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\shcore.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d78b0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32d7cd8 - 0x32d7630 ]
   +0x018 DllBase     : 0x76cf0000 Void
   +0x01c EntryPoint  : 0x76d25ac0 Void
   +0x020 SizeOfImage : 0xbf000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\msvcrt.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032d7cd8
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dcb10 - 0x32d78b0 ]
   +0x018 DllBase     : 0x6cd70000 Void
   +0x01c EntryPoint  : 0x6cdf54e0 Void
   +0x020 SizeOfImage : 0x210000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\WinSxS\x86_microsoft.windows.common-controls_6595b64144ccf1df_6.0.19041.1110_none_a8625c1886757984\COMCTL32.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032dcb10
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc170 - 0x32d7cd8 ]
   +0x018 DllBase     : 0x76db0000 Void
   +0x01c EntryPoint  : 0x76db21a0 Void
   +0x020 SizeOfImage : 0x26000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\IMM32.DLL"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x032dc170
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dcbc0 - 0x32dcb10 ]
   +0x018 DllBase     : 0x760d0000 Void
   +0x01c EntryPoint  : 0x760e1e00 Void
   +0x020 SizeOfImage : 0x7a000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\ADVAPI32.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dcbc0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc640 - 0x32dc170 ]
   +0x018 DllBase     : 0x76370000 Void
   +0x01c EntryPoint  : 0x76390070 Void
   +0x020 SizeOfImage : 0x75000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\sechost.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032dc640
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc4e0 - 0x32dcbc0 ]
   +0x018 DllBase     : 0x75ca0000 Void
   +0x01c EntryPoint  : 0x75cd3650 Void
   +0x020 SizeOfImage : 0x5f000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\bcryptPrimitives.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dc4e0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc850 - 0x32dc640 ]
   +0x018 DllBase     : 0x73f10000 Void
   +0x01c EntryPoint  : 0x73f147e0 Void
   +0x020 SizeOfImage : 0xf000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\SYSTEM32\kernel.appcore.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dc850
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc900 - 0x32dc4e0 ]
   +0x018 DllBase     : 0x73b20000 Void
   +0x01c EntryPoint  : 0x73b61000 Void
   +0x020 SizeOfImage : 0x7e000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\system32\uxtheme.dll"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x032dc900
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc220 - 0x32dc850 ]
   +0x018 DllBase     : 0x76790000 Void
   +0x01c EntryPoint  : 0x767fbd50 Void
   +0x020 SizeOfImage : 0x7e000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\clbcatq.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dc220
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc590 - 0x32dc900 ]
   +0x018 DllBase     : 0x71100000 Void
   +0x01c EntryPoint  : 0x71150cc0 Void
   +0x020 SizeOfImage : 0xc0000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\MrmCoreR.dll"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x032dc590
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dcc70 - 0x32dc220 ]
   +0x018 DllBase     : 0x76e00000 Void
   +0x01c EntryPoint  : 0x76f7be30 Void
   +0x020 SizeOfImage : 0x5b3000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\SHELL32.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dcc70
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dbd50 - 0x32dc590 ]
   +0x018 DllBase     : 0x740b0000 Void
   +0x01c EntryPoint  : 0x7428b2d0 Void
   +0x020 SizeOfImage : 0x608000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\SYSTEM32\windows.storage.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dbd50
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc6f0 - 0x32dcc70 ]
   +0x018 DllBase     : 0x75480000 Void
   +0x01c EntryPoint  : 0x754885a0 Void
   +0x020 SizeOfImage : 0x24000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\system32\Wldp.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032dc6f0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc7a0 - 0x32dbd50 ]
   +0x018 DllBase     : 0x76ba0000 Void
   +0x01c EntryPoint  : 0x76bb7860 Void
   +0x020 SizeOfImage : 0x45000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\shlwapi.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dc7a0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc2d0 - 0x32dc6f0 ]
   +0x018 DllBase     : 0x76830000 Void
   +0x01c EntryPoint  : 0x7687dfc0 Void
   +0x020 SizeOfImage : 0xd3000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\MSCTF.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dc2d0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dbe00 - 0x32dc7a0 ]
   +0x018 DllBase     : 0x76c50000 Void
   +0x01c EntryPoint  : 0x76c85cd0 Void
   +0x020 SizeOfImage : 0x96000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\OLEAUT32.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032dbe00
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc380 - 0x32dc2d0 ]
   +0x018 DllBase     : 0x69b60000 Void
   +0x01c EntryPoint  : 0x69bef2b0 Void
   +0x020 SizeOfImage : 0x94000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\system32\TextShaping.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x032dc380
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dc430 - 0x32dbe00 ]
   +0x018 DllBase     : 0x5cb80000 Void
   +0x01c EntryPoint  : 0x5cbfc700 Void
   +0x020 SizeOfImage : 0x9b000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\efswrt.dll"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x032dc430
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dbeb0 - 0x32dc380 ]
   +0x018 DllBase     : 0x72790000 Void
   +0x01c EntryPoint  : 0x72808590 Void
   +0x020 SizeOfImage : 0xdb000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\SYSTEM32\wintypes.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032dbeb0
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x32dbf60 - 0x32dc430 ]
   +0x018 DllBase     : 0x6b900000 Void
   +0x01c EntryPoint  : 0x6b903540 Void
   +0x020 SizeOfImage : 0x19000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\MPR.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x032dbf60
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x3314e48 - 0x32dbeb0 ]
   +0x018 DllBase     : 0x703c0000 Void
   +0x01c EntryPoint  : 0x70443930 Void
   +0x020 SizeOfImage : 0x18f000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\twinapi.appcore.dll"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x03314e48
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x3315898 - 0x32dbf60 ]
   +0x018 DllBase     : 0x6bd50000 Void
   +0x01c EntryPoint  : 0x6bd6c2d0 Void
   +0x020 SizeOfImage : 0x53000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\oleacc.dll"
   +0x094 LoadReason  : 4 ( LoadReasonDynamicLoad )

InLoadOrderLinks.Flink at 0x03315898
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x3314d98 - 0x3314e48 ]
   +0x018 DllBase     : 0x70ad0000 Void
   +0x01c EntryPoint  : 0x70b10690 Void
   +0x020 SizeOfImage : 0xb9000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\SYSTEM32\textinputframework.dll"
   +0x094 LoadReason  : 3 ( LoadReasonDelayloadDependency )

InLoadOrderLinks.Flink at 0x03314d98
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x33157e8 - 0x3315898 ]
   +0x018 DllBase     : 0x72fe0000 Void
   +0x01c EntryPoint  : 0x7303e960 Void
   +0x020 SizeOfImage : 0x27e000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\CoreUIComponents.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x033157e8
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x3314a28 - 0x3314d98 ]
   +0x018 DllBase     : 0x72f20000 Void
   +0x01c EntryPoint  : 0x72f57f50 Void
   +0x020 SizeOfImage : 0xb2000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\CoreMessaging.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )

InLoadOrderLinks.Flink at 0x03314a28
---------------------------------------------
   +0x000 InLoadOrderLinks :  [ 0x3315b58 - 0x33157e8 ]
   +0x018 DllBase     : 0x76480000 Void
   +0x01c EntryPoint  : 0x76484b40 Void
   +0x020 SizeOfImage : 0x63000
   +0x024 FullDllName : _UNICODE_STRING "C:\Windows\System32\WS2_32.dll"
   +0x094 LoadReason  : 0 ( LoadReasonStaticDependency )
   ```


# Similarly for InMemoryOrderLinks we grab address of InMemoryOrderModuleList

```
0: kd> dt 0x77b61c60 _PEB_LDR_DATA -b
ntdll!_PEB_LDR_DATA
   +0x000 Length           : 0x30
   +0x004 Initialized      : 0x1 ''
   +0x008 SsHandle         : (null)
   +0x00c InLoadOrderModuleList : _LIST_ENTRY [ 0x32d1d30 - 0x3315b58 ]
      +0x000 Flink            : 0x032d1d30
      +0x004 Blink            : 0x03315b58
   +0x014 InMemoryOrderModuleList : _LIST_ENTRY [ 0x32d1d38 - 0x3315b60 ]
      +0x000 Flink            : 0x032d1d38
      +0x004 Blink            : 0x03315b60
   +0x01c InInitializationOrderModuleList : _LIST_ENTRY [ 0x32d1c58 - 0x33158a8 ]
      +0x000 Flink            : 0x032d1c58
      +0x004 Blink            : 0x033158a8
   +0x024 EntryInProgress  : (null)
   +0x028 ShutdownInProgress : 0 ''
   +0x02c ShutdownThreadId : (null)

0: kd> ? 0x77b61c60 + 0x014
Evaluate expression: 2008423540 = 77b61c74

dt 0x77b61c74 ntdll!_LDR_DATA_TABLE_ENTRY -l InMemoryOrderLinks.Flink DllBase FullDllName Loadreason
(truncated)
```

# Dump the same using windbg extension directly

`!dlls`


# Reference
https://docs.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-peb

https://docs.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-peb_ldr_data

https://www.codemachine.com/articles/kernel_structures.html#LIST_ENTRY

https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-dlls

https://www.ired.team/offensive-security/code-injection-process-injection/finding-kernel32-base-and-function-addresses-in-shellcode
