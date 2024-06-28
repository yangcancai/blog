---
title: x64通用寄存器
date: 2024-06-28 15:31:00
tags: asm
categories:
- asm
- x64汇编
- 逆向开发
---

## 16个通用寄存器

在 x86-64 架构中，通用寄存器（General Purpose Registers, GPRs）是处理器中用于执行各种任务的寄存器。这些寄存器可以用于存储操作数、地址、指针以及各种临时数据。x86-64 架构扩展了原有的 x86 架构，使寄存器数量增加，并且每个寄存器的宽度从 32 位扩展到 64 位。

通用寄存器列表
x86-64 有 16 个 64 位的通用寄存器：

RAX: 累加器寄存器，常用于算术运算和函数返回值。
RBX: 基址寄存器，常用作指针。
RCX: 计数器寄存器，常用于循环和字符串操作。
RDX: 数据寄存器，常用于算术运算和 I/O 操作。
RSI: 源索引寄存器，常用于字符串和数组操作。
RDI: 目的索引寄存器，常用于字符串和数组操作。
RBP: 基址指针寄存器，常用作栈帧指针。
RSP: 堆栈指针寄存器，指向当前的栈顶。
R8: 通用寄存器，常用于函数参数传递。
R9: 通用寄存器，常用于函数参数传递。
R10: 通用寄存器。
R11: 通用寄存器。
R12: 通用寄存器。
R13: 通用寄存器。
R14: 通用寄存器。
R15: 通用寄存器。
各寄存器的细分部分
每个 64 位寄存器可以按需访问其较低的部分，以便与 32 位、16 位和 8 位数据进行操作。下面是一些例子：

RAX: 64 位寄存器。

EAX: 低 32 位。
AX: 低 16 位。
AH: 高 8 位。
AL: 低 8 位。
RBX: 64 位寄存器。

EBX: 低 32 位。
BX: 低 16 位。
BH: 高 8 位。
BL: 低 8 位。
R8: 64 位寄存器。

R8D: 低 32 位。
R8W: 低 16 位。
R8B: 低 8 位。

## 常见用途

1. 函数调用约定
在 x86-64 的函数调用中，通用寄存器经常用于传递参数和返回值。以下是两个主要的调用约定：

Windows x64 调用约定：

参数传递顺序：RCX、RDX、R8、R9。
返回值：使用 RAX。
System V AMD64 ABI（Unix/Linux 系统）：

参数传递顺序：RDI、RSI、RDX、RCX、R8、R9。
返回值：使用 RAX。
2. 算术运算
许多算术运算指令会用到这些寄存器，例如加法、减法、乘法、除法等：

assembly
复制代码
mov rax, 5      ; 将 5 存入 RAX
add rax, 3      ; RAX = RAX + 3, 此时 RAX 为 8
3. 地址计算和指针操作
这些寄存器也常用于地址计算和指针操作：

assembly
复制代码
mov rsi, array  ; 将数组的基址存入 RSI
mov rax, [rsi]  ; 将数组第一个元素存入 RAX
4. 循环和字符串操作
RCX 经常作为循环计数器或字符串操作的计数器：

assembly
复制代码
mov rcx, 10     ; 设置循环计数器为 10
loop_start:
    ; 循环体代码
    dec rcx     ; 循环计数器减 1
    jnz loop_start  ; 如果 RCX 不为 0，则跳转回 loop_start
示例代码
以下是一个简单的汇编程序示例，展示了如何使用通用寄存器：

assembly
复制代码
section .data
    message db 'Hello, world!', 0

section .text
    global _start

_start:
    ; 使用 RSI 和 RDI 进行字符串复制
    mov rsi, message    ; 源地址
    mov rdi, buffer     ; 目标地址
    mov rcx, 13         ; 字符串长度
    rep movsb           ; 复制字符串

    ; 使用 RAX 和 RBX 进行算术运算
    mov rax, 5
    mov rbx, 10
    add rax, rbx        ; RAX = RAX + RBX

    ; 调用系统调用退出程序
    mov eax, 60         ; syscall number (sys_exit)
    xor edi, edi        ; status (0)
    syscall             ; 调用系统调用
这个程序将“Hello, world!”字符串复制到一个缓冲区，然后执行一些简单的算术运算，最后退出程序。

通过理解和有效使用这些通用寄存器，你可以编写高效的汇编代码，实现复杂的功能。