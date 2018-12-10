---
title: csapp-bomb实验
date: 2018-9-21
comments: true
tags:
- csapp
- 逆向
categories:
- csapp
---

## 实验材料
   这是这学期上系统级编程课的实验之一，是从CMU引入的，源代码和资料可以[CMU课程网站](http://csapp.cs.cmu.edu/3e/labs.html)获得，直接选择第二个实验的Self-Study Handout下载即可。
  做这个实验需要反汇编和与调试，建议使用gdb和objdump，如果还不会gdb 可以看看这个[简易gdb使用指南](http://csapp.cs.cmu.edu/3e/docs/gdbnotes-x86-64.pdf)，关于objdump[简单看看这个就行了](http://man.linuxde.net/objdump)。
## 准备工作
下载的解压包里面就三个文件，有用的也就是那个可执行文件bomb,还有一个bomb.c可以让你看清楚整个程序执行流程
![bomp.c.png](https://upload-images.jianshu.io/upload_images/8804020-1ad1ccf788e2b1e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这是main函数主要的部分，可以看到程序分为6个phase，每一个都需要你输入一行字符串，然后对应调用phase_n()函数进行判断是否触发炸弹
先用objdump -d bomb > bomb.asm 反汇编保存到bomb.asm,然后用tmux开分屏,左边是gdb调试bomb
![2.png](https://upload-images.jianshu.io/upload_images/8804020-d8a8ac6b4253ee05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
首先定位到main函数如下:
```
   00000000000400da0 <main>:
  400da0:	53                   	push   %rbx
  400da1:	83 ff 01             	cmp    $0x1,%edi
  400da4:	75 10                	jne    400db6 <main+0x16>
  400da6:	48 8b 05 9b 29 20 00 	mov    0x20299b(%rip),%rax        # 603748 <stdin@@GLIBC_2.2.5>
  400dad:	48 89 05 b4 29 20 00 	mov    %rax,0x2029b4(%rip)        # 603768 <infile>
  400db4:	eb 63                	jmp    400e19 <main+0x79>
  400db6:	48 89 f3             	mov    %rsi,%rbx
  400db9:	83 ff 02             	cmp    $0x2,%edi
  400dbc:	75 3a                	jne    400df8 <main+0x58>
  400dbe:	48 8b 7e 08          	mov    0x8(%rsi),%rdi
  400dc2:	be b4 22 40 00       	mov    $0x4022b4,%esi
  400dc7:	e8 44 fe ff ff       	callq  400c10 <fopen@plt>
  400dcc:	48 89 05 95 29 20 00 	mov    %rax,0x202995(%rip)        # 603768 <infile>
  400dd3:	48 85 c0             	test   %rax,%rax
  400dd6:	75 41                	jne    400e19 <main+0x79>
  400dd8:	48 8b 4b 08          	mov    0x8(%rbx),%rcx
  400ddc:	48 8b 13             	mov    (%rbx),%rdx
  400ddf:	be b6 22 40 00       	mov    $0x4022b6,%esi
  400de4:	bf 01 00 00 00       	mov    $0x1,%edi
  400de9:	e8 12 fe ff ff       	callq  400c00 <__printf_chk@plt>
  400dee:	bf 08 00 00 00       	mov    $0x8,%edi
  400df3:	e8 28 fe ff ff       	callq  400c20 <exit@plt>
  400df8:	48 8b 16             	mov    (%rsi),%rdx
  400dfb:	be d3 22 40 00       	mov    $0x4022d3,%esi
  400e00:	bf 01 00 00 00       	mov    $0x1,%edi
  400e05:	b8 00 00 00 00       	mov    $0x0,%eax
  400e0a:	e8 f1 fd ff ff       	callq  400c00 <__printf_chk@plt>
  400e0f:	bf 08 00 00 00       	mov    $0x8,%edi
  400e14:	e8 07 fe ff ff       	callq  400c20 <exit@plt>
  400e19:	e8 84 05 00 00       	callq  4013a2 <initialize_bomb>
  400e1e:	bf 38 23 40 00       	mov    $0x402338,%edi
  400e23:	e8 e8 fc ff ff       	callq  400b10 <puts@plt>
  400e28:	bf 78 23 40 00       	mov    $0x402378,%edi
  400e2d:	e8 de fc ff ff       	callq  400b10 <puts@plt>
  400e32:	e8 67 06 00 00       	callq  40149e <read_line>
  400e37:	48 89 c7             	mov    %rax,%rdi
  400e3a:	e8 a1 00 00 00       	callq  400ee0 <phase_1>
  400e3f:	e8 80 07 00 00       	callq  4015c4 <phase_defused>
  400e44:	bf a8 23 40 00       	mov    $0x4023a8,%edi
  400e49:	e8 c2 fc ff ff       	callq  400b10 <puts@plt>
  400e4e:	e8 4b 06 00 00       	callq  40149e <read_line>
  400e53:	48 89 c7             	mov    %rax,%rdi
  400e56:	e8 a1 00 00 00       	callq  400efc <phase_2>
  400e5b:	e8 64 07 00 00       	callq  4015c4 <phase_defused>
  400e60:	bf ed 22 40 00       	mov    $0x4022ed,%edi
  400e65:	e8 a6 fc ff ff       	callq  400b10 <puts@plt>
  400e6a:	e8 2f 06 00 00       	callq  40149e <read_line>
  400e6f:	48 89 c7             	mov    %rax,%rdi
  400e72:	e8 cc 00 00 00       	callq  400f43 <phase_3>      
  400e77:	e8 48 07 00 00       	callq  4015c4 <phase_defused>
  400e7c:	bf 0b 23 40 00       	mov    $0x40230b,%edi
  400e81:	e8 8a fc ff ff       	callq  400b10 <puts@plt>
  400e86:	e8 13 06 00 00       	callq  40149e <read_line>
  400e8b:	48 89 c7             	mov    %rax,%rdi
  400e8e:	e8 79 01 00 00       	callq  40100c <phase_4>
  400e93:	e8 2c 07 00 00       	callq  4015c4 <phase_defused>
  400e98:	bf d8 23 40 00       	mov    $0x4023d8,%edi
  400e9d:	e8 6e fc ff ff       	callq  400b10 <puts@plt>
  400ea2:	e8 f7 05 00 00       	callq  40149e <read_line>
  400ea7:	48 89 c7             	mov    %rax,%rdi
  400eaa:	e8 b3 01 00 00       	callq  401062 <phase_5>
  400eaf:	e8 10 07 00 00       	callq  4015c4 <phase_defused>
  400eb4:	bf 1a 23 40 00       	mov    $0x40231a,%edi
  400eb9:	e8 52 fc ff ff       	callq  400b10 <puts@plt>
  400ebe:	e8 db 05 00 00       	callq  40149e <read_line>
  400ec3:	48 89 c7             	mov    %rax,%rdi
  400ec6:	e8 29 02 00 00       	callq  4010f4 <phase_6>
  400ecb:	e8 f4 06 00 00       	callq  4015c4 <phase_defused>
  400ed0:	b8 00 00 00 00       	mov    $0x0,%eax
  400ed5:	5b                   	pop    %rbx
```
和我们在bomb.c中看到的是一样的，main函数内每次先调用read_line,然后将返回的地址传递给phase_n函数，如果输入的不正确那么就会执行爆炸函数。所以当然就顺着main函数执行轨迹一个个来排雷~

##  Phase_1
先查看phase_1反汇编代码:
```
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq   

```
phase_1汇编代码非常简洁,　在这之前首先说明一下
 > read_line函数会将读入字符串地址存放在rdi 和rsi中，strings_not_equal函数会使用edi和esi中的值当做两个字符址，并且判断他们是否相等，相等返回0

再看phase_1函数首先将0x402400这个赋值给esi，然后调用strings_not_equal,　刚才分析了，在每次调用phase_n之前都会先调用read_line读入一行并且放在edi和esi。显然这里是调用字符串比较函数比较我们输入的字符串和存放在0x402400地址的字符串是否相等，紧接着调用test指令，如果eax为0也就是两个字符串相等就跳转到函数结尾，否则调用explode_bomb函数，这个就是引爆炸弹的函数。到这里答案也就出来了，我们需要输入的就是存放在0x402400处的字符串。接下来用gdb开始调试

```
(gdb) b  phase_1               ;打断点
(gdb) run                           ;执行到下一个断点
(gdb) info r　                    ;查看寄存器值
(gdb) print (char*)(0x402400) ;查看内存中字符串
```
![3.png](https://upload-images.jianshu.io/upload_images/8804020-fe204296391d5a63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过上面调试窗口可以看到($edi)处存放的正是我输入的hello ,而地址0x402400处的"Border relations with Canada have never been better."正是答案。接着重新打开调试窗口输入这个字符串，通过phase_1。
> 可以把之前解出来的答案写到一个文件里，每个答案一行，然后开始调试时设置下命令行参数
set args xixi(这里是你的答案文件名)即可后续直接输入已经解出的答案

## Phase_2
还是先看看汇编代码，这个函数要长不少，而且中间多了很多条件跳转指令，很不利于理解代码作用，我一般喜欢在分支处标明
```
0000000000400efc <phase_2>:
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>　　　　;读入六个数，第一个存在($rsp)处
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)　　　　　　　　　　  ;第一个数和1比较
  400f0e:	74 20                	je     400f30 <phase_2+0x34>                        ；等于１跳转
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>                      ;否则爆炸
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax                     ;取出rbx-4处的值赋给eax
  400f1a:	01 c0                	add    %eax,%eax                               ; eax = eax *2
  400f1c:	39 03                	cmp    %eax,(%rbx)                                        
　　　　　　　　　;比较eax*2和rbx处的值,注意:eax是ebx-4处的值，即将rbx和前一个数的两倍比较
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
　　　　　　                                                ；如果相等就跳转，而跳转处的代码是将rbx+4
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>    ;否则爆炸
  400f25:	48 83 c3 04          	add    $0x4,%rbx         ; 将rbx+4
  400f29:	48 39 eb             	cmp    %rbp,%rbx      
                  ;将加4后的值和rbp比较，注意rbp是rsp+24,而rsp是第一个数，一个数四个字节。那么rbp就应该是
                  后那个数后面那个地址，即rbp是个循环哨兵
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>   ；不等就继续跳转去循环
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>  ;　相等就结束跳转到函数结尾
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx                                       ；将rsp+4存到rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp                                       ;将rsp +24 存到rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>                         ;跳转
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq   
```
可以很明显的看到调用了read_six_numbers，这个函数作用名字已经告诉我们了，只是有一点需要去看看它的代码才知道，它会把第一个数存在地址($rsp),以后依次递增。这段代码注释已经很清楚了，主体就是一个循环，而每一轮循环要做的就是判断当前数和前一个数的两倍是否相等，一旦不相等就爆炸。加上要求第一个数必须为1，那么输入的六个数就应该是　1 2 4 8 16 32,用gdb调试验证
![4.png](https://upload-images.jianshu.io/upload_images/8804020-59b52dd40989b5ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## phase_3
还是先放第三行的代码：
```
0000000000400f43 <phase_3>:
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)    
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>     #将第一个数和７比较，大于跳转到炸弹
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
  400f75:	ff 24 c5 70 24 40 00 	jmpq   (,*0x402470%rax,8)
  400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax
  400f81:	eb 3b                	jmp    400fbe <phase_3+0x7b>
  400f83:	b8 c3 02 00 00       	mov    $0x2c3,%eax
  400f88:	eb 34                	jmp    400fbe <phase_3+0x7b>
  400f8a:	b8 00 01 00 00       	mov    $0x100,%eax
  400f8f:	eb 2d                	jmp    400fbe <phase_3+0x7b>
  400f91:	b8 85 01 00 00       	mov    $0x185,%eax
  400f96:	eb 26                	jmp    400fbe <phase_3+0x7b>
  400f98:	b8 ce 00 00 00       	mov    $0xce,%eax
  400f9d:	eb 1f                	jmp    400fbe <phase_3+0x7b>
  400f9f:	b8 aa 02 00 00       	mov    $0x2aa,%eax
  400fa4:	eb 18                	jmp    400fbe <phase_3+0x7b>
  400fa6:	b8 47 01 00 00       	mov    $0x147,%eax
  400fab:	eb 11                	jmp    400fbe <phase_3+0x7b>
  400fad:	e8 88 04 00 00       	callq  40143a <explode_bomb>
  400fb2:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb7:	eb 05                	jmp    400fbe <phase_3+0x7b>
  400fb9:	b8 37 01 00 00       	mov    $0x137,%eax
  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq   
```
首先看到了，sscanf，所以这个函数前面一定会有一个字符串常量存储需要读取的数据格式，所以字符串常量一定是$0x4025cf，
用gdb打印出来确认格式
![image.png](https://upload-images.jianshu.io/upload_images/8804020-33d85fefa404514a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们看到格式是"%d %d",所以我们需要输入两个整数。往后看汇编，这段代码的后面有很多的jmp语句，而且极其的有规律，估计是个跳转表即switch语句，要跳转过去的地址是0x402470+%rax+8,而eax就是我们输入的第一个数，然后每一个jmp可以看做是一个case语句，每一个case语句我们看到都是在将一个参数赋值给eax,比如0xcf、0x2c3等，然后所有case统一跳转到0x400fbe，而在这个地方则是将我们输入的第二个数和eax中的值比较，相等就跳过炸弹否则爆炸，而刚才分析了eax的值是根据第一个值跳转到不同的case得到的。那么有多少个case就应该有多少个解题的答案，我们只需要确定第一个数然后顺着挑战到其中一个case，然后看这个case中的常量值是多少即为我们输入的第二个值。要注意输入的第一个值必须小于７，这在汇编中有注释，可见应该有７个case.
我选择第一个数输入３，顺着找到了第二个数为0x100即十进制256。

        所以此题的其中一个解为3 256

## phase_4
反汇编代码:
```
000000000040100c <phase_4>:
  40100c:	48 83 ec 18          	sub    $0x18,%rsp
  401010:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  401015:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  40101a:	be cf 25 40 00       	mov    $0x4025cf,%esi
  40101f:	b8 00 00 00 00       	mov    $0x0,%eax
  401024:	e8 c7 fb ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  401029:	83 f8 02             	cmp    $0x2,%eax
  40102c:	75 07                	jne    401035 <phase_4+0x29>
  40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
  401033:	76 05                	jbe    40103a <phase_4+0x2e>　#第一个数小与等于0xe跳转
  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
  40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
  40103f:	be 00 00 00 00       	mov    $0x0,%esi
  401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
  401048:	e8 81 ff ff ff       	callq  400fce <func4>
  40104d:	85 c0                	test   %eax,%eax　　　　　　#测试返回值是否为０，否就爆炸
  40104f:	75 07                	jne    401058 <phase_4+0x4c>
  401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  401056:	74 05                	je     40105d <phase_4+0x51>
  401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
  40105d:	48 83 c4 18          	add    $0x18,%rsp
  401061:	c3                   	retq   
```
还是出现了sscan,这次直接先看输入的格式，0x4025cf不正是上一题的格式字符串"%d %d"吗，看来这题还是需要输入两个整数
，phase_4汇编中还会调用func4函数，这个func４函数是关键，反汇编如下：
```
0000000000400fce <func4>: 
  400fce:	sub    $0x8,%rsp                      ;; 分配栈帧
  400fd2:	mov    %edx,%eax                      ;; C                  eax
  400fd4:	sub    %esi,%eax                      ;; C - B         更新 eax
  400fd6:	mov    %eax,%ecx                      ;; C - B              ecx 
  400fd8:	shr    $0x1f,%ecx                     ;; 右移 31 位， ecx 长为 32 位（也就是之前的最高位变为最低位，其余 31 位填充补 0），可以认为 ecx = 0
  400fdb:	add    %ecx,%eax                      ;; C - B              eax
  400fdd:	sar    %eax                           ;; 这里是一个缩写 sar $1,%eax (对应的机器码为 D1F8)  eax = (C-B)/2
  400fdf:	lea    (%rax,%rsi,1),%ecx             ;; (C+B)/2               ecx        
  400fe2:	cmp    %edi,%ecx                      ;; ecx 与 A 进行比较               (1)
  400fe4:	jle    400ff2 <func4+0x24>            ;; ecx 小于等于 A 则跳转
  400fe6:	lea    -0x1(%rcx),%edx                ;; C = (C+B)/2 - 1
  400fe9:	callq  400fce <func4>                 ;; 递归调用
  400fee:	add    %eax,%eax                      ;; 递归返回值加倍
  400ff0:	jmp    401007 <func4+0x39>            ;; 跳转到 func 函数的出口处 
  400ff2:	mov    $0x0,%eax                      ;; eax = 0                      (2)
  400ff7:	cmp    %edi,%ecx                      ;; ecx 与 A 进行比较
  400ff9:	jge    401007 <func4+0x39>            ;; eax 大于等于 A 则跳转
  400ffb:	lea    0x1(%rcx),%esi                 ;; B = ecx + 1
  400ffe:	callq  400fce <func4>                 ;; 递归调用
  401003:	lea    0x1(%rax,%rax,1),%eax          ;; 递归返回值加倍并再加上 1
  401007:	add    $0x8,%rsp                      ;; 释放栈帧
  40100b:	retq                                  ;; 函数返回

```
在这个函数中我们很明确的看到了func4内部在调用func4，这不就是递归的汇编。尝试着写出对应的c语言代码如下：
```
int func4(int target, int step, int limit) {
  /* edi = target; esi = step; edx = limit */
  int temp = (limit - step) * 0.5;
  int mid = temp + step;
  if (mid > target) {
    limit = mid - 1;
    int ret1 = func4(target, step, limit);
    return 2 * ret1;
  } else {
    if (mid >= target) {
      return 0;
    } else {
      step = mid + 1;
      int ret2 = func4(target, step, limit);
      return (2 * ret2 + 1);
    }
  }
}
```
最后根据c语言代码推出一个答案(7,0),但是此题还有其它的解。

## phase_5
```
0000000000401062 <phase_5>:
  401062:	53                   	push   %rbx
  401063:	48 83 ec 20          	sub    $0x20,%rsp
  401067:	48 89 fb             	mov    %rdi,%rbx
  40106a:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  401071:	00 00 
  401073:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
  401078:	31 c0                	xor    %eax,%eax
  40107a:	e8 9c 02 00 00       	callq  40131b <string_length>
  40107f:	83 f8 06             	cmp    $0x6,%eax　　　#要求输入的字符串长度为6
  401082:	74 4e                	je     4010d2 <phase_5+0x70>
  401084:	e8 b1 03 00 00       	callq  40143a <explode_bomb>
  401089:	eb 47                	jmp    4010d2 <phase_5+0x70>
  40108b:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx
  40108f:	88 0c 24             	mov    %cl,(%rsp)
  401092:	48 8b 14 24          	mov    (%rsp),%rdx
  401096:	83 e2 0f             	and    $0xf,%edx     #  取edx后四位
  401099:	0f b6 92 b0 24 40 00 	movzbl 0x4024b0(%rdx),%edx　　#将edx后四位作为0x4024b0字符数组的索引值
  4010a0:	88 54 04 10          	mov    %dl,0x10(%rsp,%rax,1)　　　# 依次拷贝字符数组到0x10((%rsp,%rax,1))
  4010a4:	48 83 c0 01          	add    $0x1,%rax             #循环计数+1
  4010a8:	48 83 f8 06          	cmp    $0x6,%rax            #循环计数和6比较，即循环6次
  4010ac:	75 dd                	jne    40108b <phase_5+0x29>
  4010ae:	c6 44 24 16 00       	movb   $0x0,0x16(%rsp)    #字符串末尾添加"\0"
  4010b3:	be 5e 24 40 00       	mov    $0x40245e,%esi　　# 字符串常量
  4010b8:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  4010bd:	e8 76 02 00 00       	callq  401338 <strings_not_equal>　# 和字符串常量比较
  4010c2:	85 c0                	test   %eax,%eax
  4010c4:	74 13                	je     4010d9 <phase_5+0x77>
  4010c6:	e8 6f 03 00 00       	callq  40143a <explode_bomb>
  4010cb:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
  4010d0:	eb 07                	jmp    4010d9 <phase_5+0x77>
  4010d2:	b8 00 00 00 00       	mov    $0x0,%eax
  4010d7:	eb b2                	jmp    40108b <phase_5+0x29>
  4010d9:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
  4010de:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  4010e5:	00 00 
  4010e7:	74 05                	je     4010ee <phase_5+0x8c>
  4010e9:	e8 42 fa ff ff       	callq  400b30 <__stack_chk_fail@plt>
  4010ee:	48 83 c4 20          	add    $0x20,%rsp
  4010f2:	5b                   	pop    %rbx
  4010f3:	c3                   	retq   
```
这里后面会有一个和字符串常量比较的地方，我们先看看这个字符串常量是什么：
![image.png](https://upload-images.jianshu.io/upload_images/8804020-7b22d2906fddf1d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
"flyers"
这段汇编还有一个字符串常量0x4024b0:
![](https://upload-images.jianshu.io/upload_images/8804020-59dabfa06a34b71d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
"maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"

由于汇编代码比较长，我就直接说明这一段到底在干什么：
1.要求输入6个字符，然后依次循环这个输入的字符数组
2.每一轮循环取一个字符，然后取这个字符的后四位作为索引，在第二个字符常量处取一个字符
依次存放到0x10(%rsp)处
3.最后将新0x10(%rsp)处的字符串和"flyers"比较，相同则通过，否则爆炸
所以我们需要根据结果倒推，比如flyers中的f字符是由我们输入的第一个字符的后四位作为索引在
"maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"取得，但是我们知道四位二进制最多索引16
个位置，所以这一长串的字符只有前16个可以来取我们需要的字符。所以f的索引为9,即二进制1001，只需要查询ascii表后四位为1001的字符均可，我取的Ｙ。以此类推得到6个字符的一个组合：YONEFw


## phase_6
这一关的汇编真的太难看懂了，我只是读懂了局部一些，还没能串起来，所以这里就不贴反汇编了。
我得到的信息大概也是需要输入6个数字且小于等于6。而且在循环过程中还会翻转每个数(a = 7 -a)。
在网上查阅别人的答案
4 3 2 1 6 5


## Secret_phase
这个不看反汇编代码根本不知道有这个雷存在，现在我们就来看看这个秘密炸弹
老规矩还是看反汇编
```
0000000000401242 <secret_phase>:
  401242:	53                   	push   %rbx
  401243:	e8 56 02 00 00       	callq  40149e <read_line>
  401248:	ba 0a 00 00 00       	mov    $0xa,%edx
  40124d:	be 00 00 00 00       	mov    $0x0,%esi
  401252:	48 89 c7             	mov    %rax,%rdi
  401255:	e8 76 f9 ff ff       	callq  400bd0 <strtol@plt>
  40125a:	48 89 c3             	mov    %rax,%rbx
  40125d:	8d 40 ff             	lea    -0x1(%rax),%eax
  401260:	3d e8 03 00 00       	cmp    $0x3e8,%eax
  401265:	76 05                	jbe    40126c <secret_phase+0x2a>
  401267:	e8 ce 01 00 00       	callq  40143a <explode_bomb>
  40126c:	89 de                	mov    %ebx,%esi
  40126e:	bf f0 30 60 00       	mov    $0x6030f0,%edi
  401273:	e8 8c ff ff ff       	callq  401204 <fun7>
  401278:	83 f8 02             	cmp    $0x2,%eax
  40127b:	74 05                	je     401282 <secret_phase+0x40>
  40127d:	e8 b8 01 00 00       	callq  40143a <explode_bomb>
  401282:	bf 38 24 40 00       	mov    $0x402438,%edi
  401287:	e8 84 f8 ff ff       	callq  400b10 <puts@plt>
  40128c:	e8 33 03 00 00       	callq  4015c4 <phase_defused>
  401291:	5b                   	pop    %rbx
  401292:	c3                   	retq   
```
但是有个问题，main函数里我们没有看到显示调用secret_phase函数的指令啊，那么是哪里被调用的呢，在全局搜索关键字可以发现在phase_defused这个函数里调用了，而phase_defused是在每次通过一个phase时都会被执行的，那么接下来就是分析在什么情况下会触发调用secret_phase
### 进入前的戏
```
00000000004015c4 <phase_defused>:
  4015c4:	48 83 ec 78          	sub    $0x78,%rsp
  4015c8:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  4015cf:	00 00 
  4015d1:	48 89 44 24 68       	mov    %rax,0x68(%rsp)
  4015d6:	31 c0                	xor    %eax,%eax
      比较输入的字符串数目是否等于6，不等于则跳转至程序结束
  4015d8:	83 3d 81 21 20 00 06 	cmpl   $0x6,0x202181(%rip)        # 603760 <num_input_strings>
  4015df:	75 5e                	jne    40163f <phase_defused+0x7b>
  4015e1:	4c 8d 44 24 10       	lea    0x10(%rsp),%r8
  4015e6:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  4015eb:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  4015f0:	be 19 26 40 00       	mov    $0x402619,%esi     
  4015f5:	bf 70 38 60 00       	mov    $0x603870,%edi
  4015fa:	e8 f1 f5 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  4015ff:	83 f8 03             	cmp    $0x3,%eax
  401602:	75 31                	jne    401635 <phase_defused+0x71>
  401604:	be 22 26 40 00       	mov    $0x402622,%esi
  401609:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  40160e:	e8 25 fd ff ff       	callq  401338 <strings_not_equal>
  401613:	85 c0                	test   %eax,%eax
  401615:	75 1e                	jne    401635 <phase_defused+0x71>
  401617:	bf f8 24 40 00       	mov    $0x4024f8,%edi
  40161c:	e8 ef f4 ff ff       	callq  400b10 <puts@plt>
  401621:	bf 20 25 40 00       	mov    $0x402520,%edi
  401626:	e8 e5 f4 ff ff       	callq  400b10 <puts@plt>
  40162b:	b8 00 00 00 00       	mov    $0x0,%eax
  401630:	e8 0d fc ff ff       	callq  401242 <secret_phase>　　　　；调用secret_phase
  401635:	bf 58 25 40 00       	mov    $0x402558,%edi
  40163a:	e8 d1 f4 ff ff       	callq  400b10 <puts@plt>
  40163f:	48 8b 44 24 68       	mov    0x68(%rsp),%rax
  401644:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  40164b:	00 00 
  40164d:	74 05                	je     401654 <phase_defused+0x90>
  40164f:	e8 dc f4 ff ff       	callq  400b30 <__stack_chk_fail@plt>
  401654:	48 83 c4 78          	add    $0x78,%rsp
  401658:	c3                   	retq   
```
我们来一段一段分析上面的代码
首先是
```
4015d6:	31 c0                	xor    %eax,%eax
      比较输入的字符串数目是否等于6，不等于则跳转至程序结束
  4015d8:	83 3d 81 21 20 00 06 	cmpl   $0x6,0x202181(%rip)        # 603760 <num_input_strings>
  4015df:	75 5e                	jne    40163f <phase_defused+0x7b>
```
然后如果输入的是六个字符串，也就是说你通过了前六个phase而且没有触发爆炸就能进入接下来的代码
```
 4015f0:	be 19 26 40 00       	mov    $0x402619,%esi     
 4015f5:	bf 70 38 60 00       	mov    $0x603870,%edi
 4015fa:	e8 f1 f5 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
 4015ff:	83 f8 03             	cmp    $0x3,%eax
 401602:	75 31                	jne    401635 <phase_defused+0x71>
  ```
这里的esi 和edi显然是两个字符串的地址，接下来会调用sscanf，所以有一个必然是我们输入的字符串，另外一个是scanf("formate",&,&)中的formate，我们接下来用gdb看看这两个字符串到底是什么
![5.png](https://upload-images.jianshu.io/upload_images/8804020-886e1df4d2d8382e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可见esi里放的是"%d %d %s" 而edi则是我们做phase_4输入的答案"7 0"但是这肯定不配啊，%s没法匹配。我们继续看
```
 4015fa:	e8 f1 f5 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  4015ff:	83 f8 03             	cmp    $0x3,%eax
  401602:	75 31                	jne    401635 <phase_defused+0x71>
 ```
在调用sscanf后，判断返回值eax(即正确匹配的通配符个数)是否为3，不等于的话就跳转到函数末尾打印这句话
```
 401635:	bf 58 25 40 00       	mov    $0x402558,%edi
  40163a:	e8 d1 f4 ff ff       	callq  400b10 <puts@plt>
```
我们看看0x402558这里放的是什么
![](https://upload-images.jianshu.io/upload_images/8804020-08d158026e4c6efc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
正是顺利通过前六个phase提示语，但是我们没有进入secret_phase
所以现在我们假设我们输入的匹配3个也就是在第四个题解后面加一个字符串会执行到哪
```
  401604:	be 22 26 40 00       	mov    $0x402622,%esi
  401609:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  40160e:	e8 25 fd ff ff       	callq  401338 <strings_not_equal>
  401613:	85 c0                	test   %eax,%eax
  401615:	75 1e                	jne    401635 <phase_defused+0x71>
  401617:	bf f8 24 40 00       	mov    $0x4024f8,%edi
  40161c:	e8 ef f4 ff ff       	callq  400b10 <puts@plt>
  401621:	bf 20 25 40 00       	mov    $0x402520,%edi
  401626:	e8 e5 f4 ff ff       	callq  400b10 <puts@plt>
  40162b:	b8 00 00 00 00       	mov    $0x0,%eax
  401630:	e8 0d fc ff ff       	callq  401242 <secret_phase>　　　　；调用secret_phase
```
这里又是将两个字符串地址传到esi和edi然后调用字符串比较函数，不等还是会跳转到函数结束然后打印那句祝贺，如果相等则会先打印出0x4024f8和0x402520处的字符串然后调用secret_phase，看来想进入秘密关卡关键就是让edi和esi中的字符串相等。我们先来看看这两个地方到底是什么。为了能够执行到这一步我们先在第四题的题解后面加一个字符串也就是"7 0"变"7 0 xixi"(xixi是随便加的)，下面放gdb查看字符串截图
![7.png](https://upload-images.jianshu.io/upload_images/8804020-9b732e2ea1900bfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
！！！！！！！这正是想的那样，rdi里放的是%s匹配的那一个字符串，而rsi里放的就是一个提前设定好的。只要这两个相等我们就能进入秘密关卡，好，我们拿着钥匙"DrEvil"去替换"xixi"，开始正式进入secret_phase(搞这么大半天才进入。。。。
### 前戏后的主角
按照惯例还是先放反汇编代码，虽然前面放过了，但是隔太远了...
```
0000000000401242 <secret_phase>:
  401242:	53                   	push   %rbx
  401243:	e8 56 02 00 00       	callq  40149e <read_line>
  401248:	ba 0a 00 00 00       	mov    $0xa,%edx
  40124d:	be 00 00 00 00       	mov    $0x0,%esi
  401252:	48 89 c7             	mov    %rax,%rdi
  401255:	e8 76 f9 ff ff       	callq  400bd0 <strtol@plt>
  40125a:	48 89 c3             	mov    %rax,%rbx
  40125d:	8d 40 ff             	lea    -0x1(%rax),%eax
  401260:	3d e8 03 00 00       	cmp    $0x3e8,%eax
  401265:	76 05                	jbe    40126c <secret_phase+0x2a>
  401267:	e8 ce 01 00 00       	callq  40143a <explode_bomb>
  40126c:	89 de                	mov    %ebx,%esi
  40126e:	bf f0 30 60 00       	mov    $0x6030f0,%edi
  401273:	e8 8c ff ff ff       	callq  401204 <fun7>
  401278:	83 f8 02             	cmp    $0x2,%eax
  40127b:	74 05                	je     401282 <secret_phase+0x40>
  40127d:	e8 b8 01 00 00       	callq  40143a <explode_bomb>
  401282:	bf 38 24 40 00       	mov    $0x402438,%edi
  401287:	e8 84 f8 ff ff       	callq  400b10 <puts@plt>
  40128c:	e8 33 03 00 00       	callq  4015c4 <phase_defused>
  401291:	5b                   	pop    %rbx
  401292:	c3                   	retq   
```
还是一段一段分析
```
  401243:	e8 56 02 00 00       	callq  40149e <read_line>
  401248:	ba 0a 00 00 00       	mov    $0xa,%edx
  40124d:	be 00 00 00 00       	mov    $0x0,%esi
  401252:	48 89 c7             	mov    %rax,%rdi
  401255:	e8 76 f9 ff ff       	callq  400bd0 <strtol@plt>
  40125a:	48 89 c3             	mov    %rax,%rbx
  ```
这里很明显是先读入一行然后调用strtol函数，这个是c语言中的用于字符串转long的，函数原型如下:
> 描述:  C 库函数 *long int strtol(const char str, char endptr, int base) 把参数 str 所指向的字符串根据给定的 base     
       转换为一个长整数（类型为 long int 型），base 必须介于 2 和 36（包含）之间，或者是特殊值 0。

>声明: long int strtol(const char *str, char **endptr, int base)

那么大概可以猜出rdi中存放的read_line返回值rax是str参数，而edx中的0xa应该是代表十进制，esi应该是特殊值０
接着分析strtol返回后的
```
  40125a:	48 89 c3             	mov    %rax,%rbx　　　          ;将rax保存到rbx中　
  40125d:	8d 40 ff             	lea    -0x1(%rax),%eax               ; eax =eax -1
  401260:	3d e8 03 00 00       	cmp    $0x3e8,%eax                    ;cmp 1000, eax
  401265:	76 05                	jbe    40126c <secret_phase+0x2a>     ;if  eax < = 1000 then 跳过炸弹
  401267:	e8 ce 01 00 00       	callq  40143a <explode_bomb>           ;炸弹
  40126c:	89 de                	mov    %ebx,%esi　　　　                ;　　传参
  40126e:	bf f0 30 60 00       	mov    $0x6030f0,%edi　　              ;     　传参
  401273:	e8 8c ff ff ff       	callq  401204 <fun7>　　　            ;　　调用fun7
  401278:	83 f8 02             	cmp    $0x2,%eax　　　　　　　;比较返回值和2
  40127b:	74 05                	je     401282 <secret_phase+0x40>   ;相等就跳转输出0x402438处的字符串并返回
  40127d:	e8 b8 01 00 00       	callq  40143a <explode_bomb>　;不等就爆炸
  401282:	bf 38 24 40 00       	mov    $0x402438,%edi
  401287:	e8 84 f8 ff ff       	callq  400b10 <puts@plt>
 ```
看了来secret_phase整体就是要输入一个字符串，然后把字符串转为long类型，转换出错或者转换后的数>1000都会爆炸，然后用转换来的数传入fun7函数，如果返回值为2则顺利通这一关，否则就爆炸。
那么现在关键就是fun7到底是个什么函数，我们进去一探究竟:
fun7:
```
0000000000401204 <fun7>:
  401204:	48 83 ec 08          	sub    $0x8,%rsp
  401208:	48 85 ff             	test   %rdi,%rdi
  40120b:	74 2b                	je     401238 <fun7+0x34>
  40120d:	8b 17                	mov    (%rdi),%edx
  40120f:	39 f2                	cmp    %esi,%edx
  401211:	7e 0d                	jle    401220 <fun7+0x1c>
  401213:	48 8b 7f 08          	mov    0x8(%rdi),%rdi　　;rdi = (rdi+8)
  401217:	e8 e8 ff ff ff       	callq  401204 <fun7>　　;递归1
  40121c:	01 c0                	add    %eax,%eax
  40121e:	eb 1d                	jmp    40123d <fun7+0x39>
  401220:	b8 00 00 00 00       	mov    $0x0,%eax
  401225:	39 f2                	cmp    %esi,%edx
  401227:	74 14                	je     40123d <fun7+0x39>
  401229:	48 8b 7f 10          	mov    0x10(%rdi),%rdi
  40122d:	e8 d2 ff ff ff       	callq  401204 <fun7>　  ;递归2
  401232:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  401236:	eb 05                	jmp    40123d <fun7+0x39>
  401238:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  40123d:	48 83 c4 08          	add    $0x8,%rsp
  401241:	c3                   	retq   

```
其实这个函数我一眼看过去的就是有两个递归调用，那么我们去找出口在哪，还是一段一段来
```
  401208:	48 85 ff             	test   %rdi,%rdi　　　;edi如果为0则跳转并返回-1
  40120b:	74 2b                	je     401238 <fun7+0x34>
  
```
测试传入的edi是否为０，是就跳转至结束并返回0xffffffff即0
```
  40120d:	8b 17                	mov    (%rdi),%edx　　;取出rdi地址的值赋给edx
  40120f:	39 f2                	cmp    %esi,%edx　　　;比较edx和esi的值
  401211:	7e 0d                	jle    401220 <fun7+0x1c>　；if edx <= esi(这就是strtol转换来的数字)，跳转
  401213:	48 8b 7f 08          	mov    0x8(%rdi),%rdi　　;否则执行递归  rdi = (rdi+8)
  401217:	e8 e8 ff ff ff       	callq  401204 <fun7>　　; 递归
  40121c:	01 c0                	add    %eax,%eax   　　    ;递归返回值*2
  40121e:	eb 1d                	jmp    40123d <fun7+0x39> ;跳转至返回
```
这一段
```
  401220:	b8 00 00 00 00       	mov    $0x0,%eax　; 提前将eax置0,这其实是返回值
  401225:	39 f2                	cmp    %esi,%edx       ; 还是比较esi和edx
  401227:	74 14                	je     40123d <fun7+0x39>  ;　如果相等就跳转并返回0
  401229:	48 8b 7f 10          	mov    0x10(%rdi),%rdi　;如果不相等就　edi = (edi+16)
  40122d:	e8 d2 ff ff ff       	callq  401204 <fun7>　  ;递归2
  401232:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax  ;递归返回值　eax = 2*eax+1
  401236:	eb 05                	jmp    40123d <fun7+0x39>  跳转至返回
 
```
但是问题是我们之前分析出来需要fun7返回2才能通过，那么怎么才能返回2呢
代码细节已经注释得很清楚了，下面给一个递归的伪c语言对应版本
```
fun7(esi, void *rdi){
  if(rdi == 0)
        return -1;
  if(*rdi <= esi ){
        if(*rdi == esi)
                return 0;                                 step1
         else 
            a = fun7(esi, *(rdi+16))
            return 2*1+1                             step2
    } else {
            return 2 * fun7(esi, *(rdi+8))       step3
    }
}
```
其实我们可以看到两次递归rdi的变化是不样的，那么为了返回2,递归调用的顺序应该是step3->step2->step1
也就是*rdi的值先要　*rdi > esi  ,然后　*rdi  < esi ， 最后　*rdi == esi
而esi是我们输入的，rdi在第一次调用fun7的时候就是固定的一个数
```
  40126e:	bf f0 30 60 00       	mov    $0x6030f0,%edi　　              ;     　传参
  401273:	e8 8c ff ff ff       	callq  401204 <fun7>　　　            ;　　调用fun7
```
现在我们顺着前面分析的去看看0x6030f0放的数是什么：
![8.png](https://upload-images.jianshu.io/upload_images/8804020-4d4ddb4e892084f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
36！！所以我们输入的数,所以我们可以输入一个小于36的数去看第二步*rdi是什么
![9.png](https://upload-images.jianshu.io/upload_images/8804020-a9fc0f77351f23ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
8 ！！所以输入的数要大于8才能进入到第三步，那么继续这样直到第三步的时候就能通过*rdi == esi 这个等式来找出esi即我们应该输入的数,　接着gdb执行程序到第三步打印出rdi对应的值
![10.png](https://upload-images.jianshu.io/upload_images/8804020-7e059c6abaf0f527.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
22 ！！！！！！现在要做的只是验证22对不对

我把所有题解放到xixi文件中，执行./bomb xixi
![11.png](https://upload-images.jianshu.io/upload_images/8804020-0db315019f540a56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Wow!顺利通过六关和一个隐藏关哦，分析完这个秘密关卡已经一点半了....

























  