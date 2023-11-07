---
layout: post
title: Win32 Kernel INT3 Hook
categories: [windows]
---

## INT3 中断 

INT3 中断是调试器用来实现断点的一种方式，当程序执行到 INT3 中断时，会触发调试器的中断处理函数，从而实现断点的功能。

INT3 中断的中断向量号为 0x03 , 内核会在 IDT 表中查找 0x03 的索引的描述符，然后通过其中的 GDT 的段选择符和偏移量找到中断处理函数的地址，然后执行中断处理函数。

1. 通过中断号 0x03 在 IDT 表中查找中断描述符
2. 通过中断描述符中的段选择子在 GDT 表中查找段描述符
3. 通过段描述符中的段基址和中断描述符中的偏移量计算中断处理函数的地址

![interrupt call](/pics/win32_kernel_int3_hook/interrupt_call.jpg)


## INT3 中断的处理函数的HOOK思路

有了上面的知识，我们就利用修改 IDT 表中的中断描述符的段选择子，将中断处理函数的地址指向我们自己的中断处理函数，从而实现 INT3 中断的HOOK。

1. 新建一个段描述符，将段基址指向我们自己的中断处理函数的Base 
2. 修改 IDT 表中的中断描述符的段选择子，将其指向我们新建的段描述符
3. 在我们自己的中断处理函数中，执行原来的中断处理函数，从而实现 INT3 中断的HOOK

这里通过使用 windbg 调试 windows7 32位系统的内核来理一下思路。

### 计算中断处理函数的基址

windbg 查看 INT3 的中断描述符和中断处理函数

```bash
kd> r idtr
idtr=80b98000

kd> dq 80b98000
80b98000  83e88e00`00085210 83e88e00`0008557c
80b98010  00008500`00580000 83e8ee00`00085fc0
......

kd> !idt 3

Dumping IDT: 80b98000

785b07be00000003:	83e85fc0 nt!KiTrap03
```

通过上面的信息，我们可以得到中断处理函数的基址为  `0x83e85fc0`

假设我们的中断处理函数的地址为 `0x12345678`，那么我们的中断处理函数的基址为 `0x12345678 - 0xx83e85fc0 = 0x8E4BF6B8` 

### 新建段描述符

![segment descriptor](/pics/win32_kernel_int3_hook/segment_descriptor.jpg)


根据上图，我们可以得到我们的段描述符的结构体如下：

```
8E CF9B 4BF6B8 FFFF

C => G(1), D/B(1), 0, AVL(0)
F => Segment Limit
9 => P(1), DPL(00), S(1)
B => Type(1011)
```

使用 windbg 写入

```bash
dq 80B98848 8ECF9B4B`F6B8FFFF
```

### 修改 IDT 表中的中断描述符

将原来的 `0008` 修改成 `0048`

```bash
dq 80B98018 83E8EE00`0048BFC0
```


## 代码实现

KernelUtil 代码
```c++
DWORD KernelUtil::GetGdtAddr(USHORT gdtIndex)
{
	GDT gdt = { 0 };

	__asm 
	{
		sgdt gdt;
	}
	return gdt.addr + gdtIndex * 8;
}

DWORD KernelUtil::GetIdtAddr(USHORT idtIndex)
{
	IDT idt = { 0 };

	__asm 
	{
		sidt idt;
	}

	return idt.addr + idtIndex * 8;
}

USHORT KernelUtil::GetSegmentSelector(USHORT gdtIndex, BYTE rpl /*= 0*/)
{
	return (gdtIndex << 3) | rpl;
}

```

Hook 测试程序: 

```c++
#include "stdafx.h"
#include "../Base/KernelUtil.h"

DWORD realInt3CallbackOffset = 0;
DWORD NtDbgPrint = 0;
const char* tips = "hook int3 success\n";

void _declspec(naked) MyInt3Callback() 
{
	__asm 
	{
		// jmp cs:offset;
		// 从我们伪造的段跳转真实的内核代码段
		push 0x8;
		push UseKernelCsSegment;
		jmp fword ptr [esp];

UseKernelCsSegment:
		add esp, 0x8;

		pushad;
		pushfd;

		// 这里设置成内核线程中执行
		push 0x30;
		pop fs;
		push tips;
		call NtDbgPrint;
		add esp, 0x4;

		popfd;
		popad;

		// 跳转到真实的Int3
		mov eax, realInt3CallbackOffset
		jmp eax;
	}
}


void PrintNewSegmentDescriptorCommand(DWORD funcBase, USHORT gdtIndex) 
{

	// 计算gdt中的地址
	DWORD sdAddr = KernelUtil::GetGdtAddr(gdtIndex);
	
	/*
	C => G(1), D/B(1), 0, AVL(0)
	F => Segment Limit
	9 => P(1), DPL(00), S(1)
	B => Type(1011)
	*/
	printf("eq %p %02XCF9B%02X`%04XFFFF\n", sdAddr, funcBase >> 24, (funcBase >> 16) & 0x00FF, funcBase & 0x0000FFFF);
}

void PrintModifyInt3DecriptorCommand(DWORD realInt3CallbackOffset, USHORT newSegmentSelectorIndex) 
{
	DWORD sdAddr = KernelUtil::GetIdtAddr(3);

	USHORT selecotr = KernelUtil::GetSegmentSelector(newSegmentSelectorIndex);

	// 83e8 ee00`0008 5fc0
	/* 
	E => P（1）DPL(11) S(0)
	E => 1110
	*/
	printf("eq %p %04XEE00`%04X%04X\n", sdAddr, realInt3CallbackOffset >> 16, selecotr, realInt3CallbackOffset & 0x0000FFFF);
}


int _tmain(int argc, _TCHAR* argv[])
{
	// 获取int3真实的中断处理程序的地址
	printf("使用 `!idt 3` 获取地址后输入: ");
	scanf_s("%x", &realInt3CallbackOffset);

	printf("使用 `u nt!DbgPrint` 获取地址后输入: ");
	scanf_s("%x", &NtDbgPrint);

	printf("将以下指令在windbg中执行:\n");
	USHORT overrideGdtIndex = 9;
	PrintNewSegmentDescriptorCommand((DWORD)MyInt3Callback - realInt3CallbackOffset, overrideGdtIndex);
	PrintModifyInt3DecriptorCommand(realInt3CallbackOffset, overrideGdtIndex);
	printf("之后在windbg执行恢复指令:\n");
	PrintModifyInt3DecriptorCommand(realInt3CallbackOffset, 1);
	system("pause");

	__asm {
		int 3;

		// 恢复用户线程
		push 0x3B;
		pop fs;
	}


	printf("Success!!!");
	system("pause");

	return 0;
}
```

### 测试结果

从 windbg 中获取参数:
![test result](/pics/win32_kernel_int3_hook/get_params.jpg)

程序运行截图：
![test result](/pics/win32_kernel_int3_hook/demo.jpg)

windbg 调试截图：
![test result](/pics/win32_kernel_int3_hook/windbg_demo.jpg)
