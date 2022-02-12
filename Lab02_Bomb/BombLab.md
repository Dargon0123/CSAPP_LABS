[TOC]

# 01 Bomb Lab（Boom！）

## phase_5

```shell
(gdb) disas phase_5
```



```cpp
Dump of assembler code for function phase_5:
   0x0000000000401062 <+0>:	push   %rbx // 保存寄存器在栈帧中
   0x0000000000401063 <+1>:	sub    $0x20,%rsp // 对 stack frame 开辟空间
   0x0000000000401067 <+5>:	mov    %rdi,%rbx // to use %rbx  =%rdi
   0x000000000040106a <+8>:	mov    %fs:0x28,%rax // 金丝雀 防止栈溢出
   0x0000000000401073 <+17>:	mov    %rax,0x18(%rsp) // 保存到栈顶,就相当于最高的8位
   0x0000000000401078 <+22>:	xor    %eax,%eax // 异或 clear zero
   0x000000000040107a <+24>:	callq  0x40131b <string_length>
   0x000000000040107f <+29>:	cmp    $0x6,%eax // 比较 input strings 长度必须是6
   0x0000000000401082 <+32>:	je     0x4010d2 <phase_5+112>
   0x0000000000401084 <+34>:	callq  0x40143a <explode_bomb>
   0x0000000000401089 <+39>:	jmp    0x4010d2 <phase_5+112> // start jump mark_1
       // come here ! mark_2
   0x000000000040108b <+41>:	movzbl (%rbx,%rax,1),%ecx // mark_3
   0x000000000040108f <+45>:	mov    %cl,(%rsp)
   0x0000000000401092 <+48>:	mov    (%rsp),%rdx
   0x0000000000401096 <+52>:	and    $0xf,%edx // 保留低四位的值，对应着16的余数
   0x0000000000401099 <+55>:	movzbl 0x4024b0(%rdx),%edx 
   0x00000000004010a0 <+62>:	mov    %dl,0x10(%rsp,%rax,1) //将字符stored at stack frame
   0x00000000004010a4 <+66>:	add    $0x1,%rax//依次将字符存储在字节(rsp+ 0x10)~(rsp+ 0x15)
   0x00000000004010a8 <+70>:	cmp    $0x6,%rax 
       // 将ans.txt里面的，六个字符，依次读取出来%16，并进行映射到“maduiersnfotvbyl”
   0x00000000004010ac <+74>:	jne    0x40108b <phase_5+41> // start for mark_3
   0x00000000004010ae <+76>:	movb   $0x0,0x16(%rsp)
   0x00000000004010b3 <+81>:	mov    $0x40245e,%esi // 此地址处存放"flyers"
--Type <RET> for more, q to quit, c to continue without paging--y
   0x00000000004010b8 <+86>:	lea    0x10(%rsp),%rdi  
   0x00000000004010bd <+91>:	callq  0x401338 <strings_not_equal>
   0x00000000004010c2 <+96>:	test   %eax,%eax // 判断 strings 是否相等
   0x00000000004010c4 <+98>:	je     0x4010d9 <phase_5+119> // start jump mark_4
   0x00000000004010c6 <+100>:	callq  0x40143a <explode_bomb>
   0x00000000004010cb <+105>:	nopl   0x0(%rax,%rax,1)
   0x00000000004010d0 <+110>:	jmp    0x4010d9 <phase_5+119>
       // come here ! mark_1
   0x00000000004010d2 <+112>:	mov    $0x0,%eax // clear %eax
   0x00000000004010d7 <+117>:	jmp    0x40108b <phase_5+41> // start jump mark_2
       // come here ! mark_4
   0x00000000004010d9 <+119>:	mov    0x18(%rsp),%rax
   0x00000000004010de <+124>:	xor    %fs:0x28,%rax // 防止栈溢出
   0x00000000004010e7 <+133>:	je     0x4010ee <phase_5+140>
   0x00000000004010e9 <+135>:	callq  0x400b30 <__stack_chk_fail@plt>
   0x00000000004010ee <+140>:	add    $0x20,%rsp
   0x00000000004010f2 <+144>:	pop    %rbx
   0x00000000004010f3 <+145>:	retq   
End of assembler dump.

```



![](E:\Codefield\Code_C\CSAPP\Linux命令手册\Graph\800px-ASCII-Table-wide.svg.png)

## phase_6

```shell
disas phase_6
```

```cpp
dump of assembler code for function phase_6: 
   0x00000000004010f4 <+0>:	push   %r14
   0x00000000004010f6 <+2>:	push   %r13
   0x00000000004010f8 <+4>:	push   %r12
   0x00000000004010fa <+6>:	push   %rbp
   0x00000000004010fb <+7>:	push   %rbx
       // 以上 调用者保存的寄存器 保存过程
   0x00000000004010fc <+8>:	sub    $0x50,%rsp
   0x0000000000401100 <+12>:	mov    %rsp,%r13 //%r13 =%rsp
   0x0000000000401103 <+15>:	mov    %rsp,%rsi //%rsi =%rsp
   0x0000000000401106 <+18>:	callq  0x40145c <read_six_numbers>
       // 以上 将ans.txt里面的六个数字读入到 [%rsp] -[%rsp +0x14]里面
   0x000000000040110b <+23>:	mov    %rsp,%r14 // %r14 =%rsp
   0x000000000040110e <+26>:	mov    $0x0,%r12d // %12d =0 
       // recrusive 开始
   0x0000000000401114 <+32>:	mov    %r13,%rbp // %rbp =%rsp
   0x0000000000401117 <+35>:	mov    0x0(%r13),%eax // eax =a1
   0x000000000040111b <+39>:	sub    $0x1,%eax
   0x000000000040111e <+42>:	cmp    $0x5,%eax //比较输入的数字需要 <=6
   0x0000000000401121 <+45>:	jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:	callq  0x40143a <explode_bomb>
   0x0000000000401128 <+52>:	add    $0x1,%r12d
   0x000000000040112c <+56>:	cmp    $0x6,%r12d 
       //%r12 =0x6才能跳出，tedious,针对每一个参数ai,确定<=6后，还需要和别的a[i+1]不同（尽管这只需要比较一遍就行了）
   0x0000000000401130 <+60>:	je     0x401153 <phase_6+95>
   0x0000000000401132 <+62>:	mov    %r12d,%ebx // %ebx =%r12d =1
   0x0000000000401135 <+65>:	movslq %ebx,%rax // rax =ebx =1
--Type <RET> for more, q to quit, c to continue without paging--y
   0x0000000000401138 <+68>:	mov    (%rsp,%rax,4),%eax //eax =[rsp +rax*4] =a2
   0x000000000040113b <+71>:	cmp    %eax,0x0(%rbp) // compare a[i] !=a[i+1], not equal
   0x000000000040113e <+74>:	jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:	callq  0x40143a <explode_bomb> // a1 !=a2 else bomb
   0x0000000000401145 <+81>:	add    $0x1,%ebx //ebx +1 =2 
   0x0000000000401148 <+84>:	cmp    $0x5,%ebx //
   0x000000000040114b <+87>:	jle    0x401135 <phase_6+65>
       //programe 走到这里，就是可以判断a1-a6,都不相等
   0x000000000040114d <+89>:	add    $0x4,%r13 //r13 =rsp +0x4 =a2
   0x0000000000401151 <+93>:	jmp    0x401114 <phase_6+32>
       // recrusive 调用结束
   0x0000000000401153 <+95>:	lea    0x18(%rsp),%rsi // rsi =rsp +0x18
   0x0000000000401158 <+100>:	mov    %r14,%rax // rax =r14 =rsp =a1
   0x000000000040115b <+103>:	mov    $0x7,%ecx // ecx =0x7
       
   0x0000000000401160 <+108>:	mov    %ecx,%edx // edx =ecx =0x7
   0x0000000000401162 <+110>:	sub    (%rax),%edx //edx -=a1
   0x0000000000401164 <+112>:	mov    %edx,(%rax) // a1 =7-a1
   0x0000000000401166 <+114>:	add    $0x4,%rax // rax =rsp +0x4
   0x000000000040116a <+118>:	cmp    %rsi,%rax // rsp +0x18 和 rsp +0x4 compare
   0x000000000040116d <+121>:	jne    0x401160 <phase_6+108>
       // 以上实现 ai =7-ai (i =1,...,6)
   0x000000000040116f <+123>:	mov    $0x0,%esi
   0x0000000000401174 <+128>:	jmp    0x401197 <phase_6+163> 
       // clear register %rsi, jump directly address 0x401197 
   0x0000000000401176 <+130>:	mov    0x8(%rdx),%rdx //rdx =[0x6032d0 +0x8]
   0x000000000040117a <+134>:	add    $0x1,%eax // eax =2
   0x000000000040117d <+137>:	cmp    %ecx,%eax // compare [7-ai] and %eax
   0x000000000040117f <+139>:	jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:	jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:	mov    $0x6032d0,%edx // 7-ai <=1
   0x0000000000401188 <+148>:	mov    %rdx,0x20(%rsp,%rsi,2) // [rsp+0x20] =rdx
   0x000000000040118d <+153>:	add    $0x4,%rsi // calculate offset
   0x0000000000401191 <+157>:	cmp    $0x18,%rsi // rsi =0x4 until rsi =0x18 
   0x0000000000401195 <+161>:	je     0x4011ab <phase_6+183>
       // come here!
   0x0000000000401197 <+163>:	mov    (%rsp,%rsi,1),%ecx //ecx =m[rsp +rsi] = 7-ai
   0x000000000040119a <+166>:	cmp    $0x1,%ecx
   0x000000000040119d <+169>:	jle    0x401183 <phase_6+143> //7-ai <=1 jump 0x401183
       // 7-a1 >1 continue !
   0x000000000040119f <+171>:	mov    $0x1,%eax // eax =1
   0x00000000004011a4 <+176>:	mov    $0x6032d0,%edx
   0x00000000004011a9 <+181>:	jmp    0x401176 <phase_6+130> // jump above
       // 以上利用 7-ai 的值,将0x6032d0 +i*0x8, 放入rsp +0x20 +2*rsi
   0x00000000004011ab <+183>:	mov    0x20(%rsp),%rbx
--Type <RET> for more, q to quit, c to continue without paging--c
   0x00000000004011b0 <+188>:	lea    0x28(%rsp),%rax
   0x00000000004011b5 <+193>:	lea    0x50(%rsp),%rsi
   0x00000000004011ba <+198>:	mov    %rbx,%rcx
   0x00000000004011bd <+201>:	mov    (%rax),%rdx // 提取node2 的 address
   0x00000000004011c0 <+204>:	mov    %rdx,0x8(%rcx) // node1.next =node2,
       // 将node2的地址，放在node1的地址+0x8的位置上
   0x00000000004011c4 <+208>:	add    $0x8,%rax
   0x00000000004011c8 <+212>:	cmp    %rsi,%rax
   0x00000000004011cb <+215>:	je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:	mov    %rdx,%rcx // 重新更新节点的位置
   0x00000000004011d0 <+220>:	jmp    0x4011bd <phase_6+201>
       // 以上 是连接重新排列之后的链表
   0x00000000004011d2 <+222>:	movq   $0x0,0x8(%rdx)
   0x00000000004011da <+230>:	mov    $0x5,%ebp // 6个节点，来控制比较的次数
   0x00000000004011df <+235>:	mov    0x8(%rbx),%rax
   0x00000000004011e3 <+239>:	mov    (%rax),%eax
   0x00000000004011e5 <+241>:	cmp    %eax,(%rbx) //p[i] >p[i+1], otherwise explode!
   0x00000000004011e7 <+243>:	jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:	callq  0x40143a <explode_bomb>
   0x00000000004011ee <+250>:	mov    0x8(%rbx),%rbx
   0x00000000004011f2 <+254>:	sub    $0x1,%ebp
   0x00000000004011f5 <+257>:	jne    0x4011df <phase_6+235>
       // 以上 是判断链表是否是递减的顺序排列，否则就boom！
   0x00000000004011f7 <+259>:	add    $0x50,%rsp
   0x00000000004011fb <+263>:	pop    %rbx
   0x00000000004011fc <+264>:	pop    %rbp
   0x00000000004011fd <+265>:	pop    %r12
   0x00000000004011ff <+267>:	pop    %r13
   0x0000000000401201 <+269>:	pop    %r14
   0x0000000000401203 <+271>:	retq   
End of assembler dump.

       
```





```shell
disas read_six_numbers
```

```cpp
Dump of assembler code for function read_six_numbers:
   0x000000000040145c <+0>:	sub    $0x18,%rsp
       // 接下来被scanf 调用，开始初始化参数寄存器
       // %rdi %rsi前两个参数被占用 ,此时的 rsi 是进入函数之前的 rsp
   0x0000000000401460 <+4>:	mov    %rsi,%rdx // 第三个参数rsi =rdx <-> a1
   0x0000000000401463 <+7>:	lea    0x4(%rsi),%rcx // 第四个参数 (%rsi +0x4)<->%rcx <-> a2 
   0x0000000000401467 <+11>:	lea    0x14(%rsi),%rax // %rsi +0x14 ->%rax
   0x000000000040146b <+15>:	mov    %rax,0x8(%rsp) //多出的参数(%rsi +0x14)-->(%rsp +0x8) <->a5
   0x0000000000401470 <+20>:	lea    0x10(%rsi),%rax
   0x0000000000401474 <+24>:	mov    %rax,(%rsp) //多出的参数(%rsi +0x10)-->%rsp <->a6
   0x0000000000401478 <+28>:	lea    0xc(%rsi),%r9 // 第六个参数(%rsi +0xc)<->%r9 <-> a4
   0x000000000040147c <+32>:	lea    0x8(%rsi),%r8 // 第五个参数(%rsi +0x8)<->%r8 <-> a3
   0x0000000000401480 <+36>:	mov    $0x4025c3,%esi
   0x0000000000401485 <+41>:	mov    $0x0,%eax
       // 调用该函数之后，按%rsi +0x0 --> %rsi +0x14，查找六个对应的参数
   0x000000000040148a <+46>:	callq  0x400bf0 <__isoc99_sscanf@plt>
   0x000000000040148f <+51>:	cmp    $0x5,%eax
   0x0000000000401492 <+54>:	jg     0x401499 <read_six_numbers+61>
   0x0000000000401494 <+56>:	callq  0x40143a <explode_bomb>
   0x0000000000401499 <+61>:	add    $0x18,%rsp
   0x000000000040149d <+65>:	retq   
End of assembler dump.


```

构造整体的答案`ansV2.txt`

```shell
Border relations with Canada have never been better.
1 2 4 8 16 32
1 311
7 0 
9?>567
4 3 2 1 6 5 
```

测试运行 PASS

```shell
(gdb) r ansV2.txt 
Starting program: /home/dargon/桌面/CSAPP/lab2_bomb/bomb/bomb ansV2.txt
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Congratulations! You've defused the bomb!
[Inferior 1 (process 4988) exited normally]

```



## Bonus 彩蛋环节

phase_5 发现 secret_phase, interesting 

```shell
0x4024b0 <array.3449>:	"maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"
(gdb) 
0x4024f8:	"Curses, you've found the secret phase!"
(gdb) 
0x40251f:	""
(gdb) 
0x402520:	"But finding it and solving it are quite different..."
(gdb) 
0x402555:	""
(gdb) 
0x402556:	""
(gdb) 
0x402557:	""
(gdb) 
0x402558:	"Congratulations! You've defused the bomb!"
(gdb) 
0x402582:	"Well..."
(gdb) 
0x40258a:	"OK. :-)"
(gdb) 
0x402592:	"Invalid phase%s\n"
(gdb) 
0x4025a3:	"\nBOOM!!!"
(gdb) 
0x4025ac:	"The bomb has blown up."
(gdb) 
0x4025c3:	"%d %d %d %d %d %d"
(gdb) 
0x4025d5:	"Error: Premature EOF on stdin"
(gdb) 
0x4025f3:	"GRADE_BOMB"
(gdb) 
0x4025fe:	"Error: Input line too long"
(gdb) 
0x402619:	"%d %d %s"
(gdb) 
0x402622:	"DrEvil"
(gdb) 
0x402629:	"greatwhite.ics.cs.cmu.edu"
(gdb) 

```



```shell
(gdb) disas phase_defused 
```

```cpp
Dump of assembler code for function phase_defused:
   0x00000000004015c4 <+0>:	sub    $0x78,%rsp
   0x00000000004015c8 <+4>:	mov    %fs:0x28,%rax
   0x00000000004015d1 <+13>:	mov    %rax,0x68(%rsp)
   0x00000000004015d6 <+18>:	xor    %eax,%eax
   0x00000000004015d8 <+20>:	cmpl   $0x6,0x202181(%rip)        # 0x603760 <num_input_strings>
   0x00000000004015df <+27>:	jne    0x40163f <phase_defused+123> 
       //the above procedeers  the length of string isn't equal 6, start jump ! mark_1
   0x00000000004015e1 <+29>:	lea    0x10(%rsp),%r8
   0x00000000004015e6 <+34>:	lea    0xc(%rsp),%rcx
   0x00000000004015eb <+39>:	lea    0x8(%rsp),%rdx // 加载地址到 参数寄存器
   0x00000000004015f0 <+44>:	mov    $0x402619,%esi // check "%d %d %s"
   0x00000000004015f5 <+49>:	mov    $0x603870,%edi // check "0 0"
   0x00000000004015fa <+54>:	callq  0x400bf0 <__isoc99_sscanf@plt>
   0x00000000004015ff <+59>:	cmp    $0x3,%eax
   0x0000000000401602 <+62>:	jne    0x401635 <phase_defused+113> 
       // 检查 sscanf 函数的返回值的个数， eax ==3 jump +113 的位置m ark_2 
       // 普通的操作 在第3/4 phase 输入的是两个%d %d，所以后面需要跟着一个字符串%s，
       // 使得函数 scanf 返回值 eax ==3 向下走（不直接跳转）
   0x0000000000401604 <+64>:	mov    $0x402622,%esi // check "DrEvil"
   0x0000000000401609 <+69>:	lea    0x10(%rsp),%rdi
       // 移动 rsp两步，判断strings %rdi 和%esi 是否相等
   0x000000000040160e <+74>:	callq  0x401338 <strings_not_equal>
   0x0000000000401613 <+79>:	test   %eax,%eax
   0x0000000000401615 <+81>:	jne    0x401635 <phase_defused+113>
   0x0000000000401617 <+83>:	mov    $0x4024f8,%edi
       // check 0x4024f8 "Curses, you've found the secret phase!"
   0x000000000040161c <+88>:	callq  0x400b10 <puts@plt>
   0x0000000000401621 <+93>:	mov    $0x402520,%edi
       // check 0x402520finding  and solving it are quite different..."
   0x0000000000401626 <+98>:	callq  0x400b10 <puts@plt>
   0x000000000040162b <+103>:	mov    $0x0,%eax
   0x0000000000401630 <+108>:	callq  0x401242 <secret_phase> //secret_phase address
       // come here mark_2
   0x0000000000401635 <+113>:	mov    $0x402558,%edi 
       // check this address --> "Congratulations! You've defused the bomb!"
   0x000000000040163a <+118>:	callq  0x400b10 <puts@plt>
       // come here mark_1, over directly.
   0x000000000040163f <+123>:	mov    0x68(%rsp),%rax
   0x0000000000401644 <+128>:	xor    %fs:0x28,%rax
   0x000000000040164d <+137>:	je     0x401654 <phase_defused+144>
   0x000000000040164f <+139>:	callq  0x400b30 <__stack_chk_fail@plt>
   0x0000000000401654 <+144>:	add    $0x78,%rsp
   0x0000000000401658 <+148>:	retq   
End of assembler dump.
```

接下来直接关注 `secret_phase and functions`

查看一下汇编代码`Dump of the secret_phase`

```shell
(gdb) disas secret_phase
```

```cpp

Dump of assembler code for function secret_phase:
   0x0000000000401242 <+0>:	push   %rbx
   0x0000000000401243 <+1>:	callq  0x40149e <read_line>
   0x0000000000401248 <+6>:	mov    $0xa,%edx // 第3个参数
   0x000000000040124d <+11>:	mov    $0x0,%esi // 第2个参数
   0x0000000000401252 <+16>:	mov    %rax,%rdi // 第1个参数 对应调用函数之前的返回值
   0x0000000000401255 <+19>:	callq  0x400bd0 <strtol@plt> // string too long
   0x000000000040125a <+24>:	mov    %rax,%rbx // rbx =rax
   0x000000000040125d <+27>:	lea    -0x1(%rax),%eax // 内容-1 
   0x0000000000401260 <+30>:	cmp    $0x3e8,%eax
   0x0000000000401265 <+35>:	jbe    0x40126c <secret_phase+42> // <= Ok, otherwise
   0x0000000000401267 <+37>:	callq  0x40143a <explode_bomb>
   0x000000000040126c <+42>:	mov    %ebx,%esi // rsi = rbx =odd rax 第2参数
   0x000000000040126e <+44>:	mov    $0x6030f0,%edi // 第一参数
   0x0000000000401273 <+49>:	callq  0x401204 <fun7> // 调用function 7
   0x0000000000401278 <+54>:	cmp    $0x2,%eax
   0x000000000040127b <+57>:	je     0x401282 <secret_phase+64> // fun7的返回值==2 
   0x000000000040127d <+59>:	callq  0x40143a <explode_bomb>
   0x0000000000401282 <+64>:	mov    $0x402438,%edi // 第1参数
       // check 0x402438 "Wow! You've defused the secret stage!"
   0x0000000000401287 <+69>:	callq  0x400b10 <puts@plt>
   0x000000000040128c <+74>:	callq  0x4015c4 <phase_defused>
   0x0000000000401291 <+79>:	pop    %rbx
   0x0000000000401292 <+80>:	retq   
End of assembler dump.

```



关于`fun7`，整体的是关于一个`binary search 的traversal `

```shell
the usage of test: 
0x0000000000401208 <+4>:	test   %rdi,%rdi 
0x000000000040120b <+7>:	je     0x401238 <fun7+52>
1, test sets the zero flag--[ZF]when the result of the AND operation is zero. If two operands are equal, their bitwise AND is zero only when both are zero.
2, test also sets the sign flag--[SF] when the most significant bit is set in the result.
3, test sets the parity flag--[PF](奇偶数flag) when the number of set bits is even.(偶数)/ odd(奇数)

je [jump if equals] tests the zero flag and jumps if the flag is set. 
je is an alias of jz[jump if zero]
```



```shell
(gdb) disas fun7
```



```cpp
Dump of assembler code for function fun7:
   0x0000000000401204 <+0>:	sub    $0x8,%rsp // recursive traversal  
   0x0000000000401208 <+4>:	test   %rdi,%rdi 
   0x000000000040120b <+7>:	je     0x401238 <fun7+52> // the content of %rdi is zero, jump 
   0x000000000040120d <+9>:	mov    (%rdi),%edx // edx =m[0x6030f0], 将其打印输出即是tree
   0x000000000040120f <+11>:	cmp    %esi,%edx
   0x0000000000401211 <+13>:	jle    0x401220 <fun7+28> // edi <=esi (input的值) mark_1
   0x0000000000401213 <+15>:	mov    0x8(%rdi),%rdi // 向树的左边，遍历，找小值
   0x0000000000401217 <+19>:	callq  0x401204 <fun7>
   0x000000000040121c <+24>:	add    %eax,%eax // 对应的返回值
   0x000000000040121e <+26>:	jmp    0x40123d <fun7+57>
       // come here ！ mark_2
   0x0000000000401220 <+28>:	mov    $0x0,%eax
   0x0000000000401225 <+33>:	cmp    %esi,%edx
   0x0000000000401227 <+35>:	je     0x40123d <fun7+57> // 遍历整个树，找到相等的值，直接结束
   0x0000000000401229 <+37>:	mov    0x10(%rdi),%rdi // 向树的右边，遍历，找大值
   0x000000000040122d <+41>:	callq  0x401204 <fun7>
   0x0000000000401232 <+46>:	lea    0x1(%rax,%rax,1),%eax
   0x0000000000401236 <+50>:	jmp    0x40123d <fun7+57>
   0x0000000000401238 <+52>:	mov    $0xffffffff,%eax // 对应的返回值
   0x000000000040123d <+57>:	add    $0x8,%rsp
   0x0000000000401241 <+61>:	retq   
End of assembler dump.

```

构建`ans.txt`

```shell
Border relations with Canada have never been better.
1 2 4 8 16 32
1 311
7 0 DrEvil
9?>567
4 3 2 1 6 5 
```

测试运行结果如下 `defused all`

```shell
Starting program: /home/dargon/桌面/CSAPP/lab2_bomb/bomb/bomb ans.txt
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Curses, you've found the secret phase!
But finding it and solving it are quite different...
22
Wow! You've defused the secret stage!
Congratulations! You've defused the bomb!
[Inferior 1 (process 4959) exited normally]
```



终于梳理完了，结束！哈哈哈哈

