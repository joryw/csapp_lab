# BombLab

安装gdb：`sudo yum install gdb`

[gdb命令](http://csapp.cs.cmu.edu/2e/docs/gdbnotes-x86-64.pdf)

[gdbGuide1](http://beej.us/guide/bggdb/)

[gdbGuide2](https://heather.cs.ucdavis.edu/~matloff/UnixAndC/CLanguage/Debug.html)

使用`objdump -d ./bomb >> bomb.s`反编译得到汇编代码

### 关卡1

思路，在验证字符串是否符合的位置，找到汇编代码中用到哪个寄存器里的数据进行字符串验证，打印出里面的值就行。

首先前面会将输入的字符串存入到%rax中

![image-20211020162409116](D:\project\csapp\document\实验2：BombLab.assets\image-20211020162409116.png)

我们可以看到phase_1所在的代码如图红框所示，汇编代码含义如下

1. 分配栈帧
2. 将立即数0x402400分配给%esi。（这里的立即数就是用来检验的字符串）
3. 调用`strings_not_equal`函数，比较字符串是否正确，（在调用前就应该把用于验证的字符串存入，可以推理出0x402400就是答案）
4. test %eax, %eax的含义是将%eax和自己按位与。实际上就是判断寄存器的值是不是0. 如果是0，那么je发生跳转，否则，调用`explode_bomb`函数，发生爆炸。然后恢复栈帧，返回函数。



最后我们可以调用 `x/s 0x402400`得到结果`Border relations with Canada have never been better.`

```assembly
(gdb) x/s 0x402400
0x402400:       "Border relations with Canada have never been better."
```

### 关卡2

#### 1.phase_2

```assembly
0000000000400efc <phase_2>:
  #分配40大小的栈帧
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp 
  400f02:	48 89 e6             	mov    %rsp,%rsi    
  #调用read_six_numbers，判断输入长度是否为6
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
  #先跟1进行比较，不相等发生爆炸
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)   
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb> 
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  #0x4(%rsp)= rbx，则-0x4(%rbx)=%rsp，就是上个栈存储的数值。进行翻倍比较。相等进入下一轮循环
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq   
```

故循环6位数的结果为1 2 4 8 16 32

### 2.read_six_numbers

```assembly
000000000040145c <read_six_numbers>:
  #起始位置
  40145c:	48 83 ec 18          	sub    $0x18,%rsp 
  #%rsi由%rsp给出，%rdx由%rsi给出，所以rdx存放第一个参数
  401460:	48 89 f2             	mov    %rsi,%rdx  
  #%rcx存放第二个参数
  401463:	48 8d 4e 04          	lea    0x4(%rsi),%rcx  
  # 0x8(%rsp)存放第六个参数
  401467:	48 8d 46 14          	lea    0x14(%rsi),%rax   
  40146b:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  # (%rsp)存放第五个参数
  401470:	48 8d 46 10          	lea    0x10(%rsi),%rax
  401474:	48 89 04 24          	mov    %rax,(%rsp)
  #第四个参数存放在%r9
  401478:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  #第三个参数存放在%r8
  40147c:	4c 8d 46 08          	lea    0x8(%rsi),%r8
  401480:	be c3 25 40 00       	mov    $0x4025c3,%esi
  401485:	b8 00 00 00 00       	mov    $0x0,%eax
  40148a:	e8 61 f7 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  40148f:	83 f8 05             	cmp    $0x5,%eax
  401492:	7f 05                	jg     401499 <read_six_numbers+0x3d>
  401494:	e8 a1 ff ff ff       	callq  40143a <explode_bomb>
  401499:	48 83 c4 18          	add    $0x18,%rsp
  40149d:	c3                   	retq  
```

![image-20211020201616336](D:\project\csapp\document\实验2：BombLab.assets\image-20211020201616336.png)

从 0x4025c3可以看出，需要6位数字

```assembly
(gdb) x/s 0x4025c3
0x4025c3:       "%d %d %d %d %d %d"
```

### 关卡3

#### 1.phase_3

![image-20211020222651772](D:\project\csapp\document\实验2：BombLab.assets\image-20211020222651772.png)

sscanf(rdi,esi,rdx,rcx)

![image-20211020222913317](D:\project\csapp\document\实验2：BombLab.assets\image-20211020222913317.png)

- 利用`%rdx` 也就是`0x8+%rsp`和利用`%rcx` 也就是`0xc+%rsp`传递`sscanf`函数用的第三和第四个参数
- 第二个参数为一个常数`0x4025cf`随后调用`<__isoc99_sscanf@plt>` 这里注意一下`sscanf`如果调用成功的话会返回2这里如果成功的话会跳转到`400f6a`这里读入的`rdx`和`rcx`的值就是我们输入的第一个和第二个数

我们可以看出

```assembly
0000000000400f43 <phase_3>:
  #分配栈帧，rcx为第二个参数位置，rdx为第一个参数位置
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  #检验sscanf参数是否合法，不成立爆炸
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
```

这里会验证输入参数个数

```assembly
(gdb) x/s 0x4025cf
0x4025cf:       "%d %d"
```

从中可以看出，需要超过2个参数

继续往下看phase_3

```assembly
  #第一个参数需要在小于等于7  否则爆炸
  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
  400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
  #这里对应0x8(%rsp)的值，也就是第1个参数。在7以内，选择对应的值
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
  #比较第二个参数和选择对应的值，相等则正确。
  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq   
```

故结果可以为

```
0 207
1 311
2 707
3 256
4 389
5 206
6 682
7 327
```

### 关卡4

#### 1.phase_4

```assembly
000000000040100c <phase_4>:
  #分配栈帧，同关卡3
  40100c:	48 83 ec 18          	sub    $0x18,%rsp 
  401010:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  401015:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  40101a:	be cf 25 40 00       	mov    $0x4025cf,%esi
  40101f:	b8 00 00 00 00       	mov    $0x0,%eax
  401024:	e8 c7 fb ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  401029:	83 f8 02             	cmp    $0x2,%eax
  40102c:	75 07                	jne    401035 <phase_4+0x29>
  #第一个参数要<=15,否则爆炸
  40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
  401033:	76 05                	jbe    40103a <phase_4+0x2e>
  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
  # edx=15  esi=0 edi=第一个参数
  40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
  40103f:	be 00 00 00 00       	mov    $0x0,%esi
  401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
  401048:	e8 81 ff ff ff       	callq  400fce <func4>
  # 检查返回值是否为0，不为0发生爆炸
  40104d:	85 c0                	test   %eax,%eax
  40104f:	75 07                	jne    401058 <phase_4+0x4c>
  #第二个参数等于0，恢复帧 返回。否则爆炸
  401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  401056:	74 05                	je     40105d <phase_4+0x51>
  401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
  40105d:	48 83 c4 18          	add    $0x18,%rsp
  401061:	c3                   	retq   
```

#### 2.func4

```assembly
0000000000400fce <func4>:
  #分配大小为8的栈，eax=edx=15 eax-esi=15
  400fce:	48 83 ec 08          	sub    $0x8,%rsp
  400fd2:	89 d0                	mov    %edx,%eax
  400fd4:	29 f0                	sub    %esi,%eax
  400fd6:	89 c1                	mov    %eax,%ecx
  #移动
  400fd8:	c1 e9 1f             	shr    $0x1f,%ecx
  400fdb:	01 c8                	add    %ecx,%eax
  400fdd:	d1 f8                	sar    %eax
  400fdf:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx
  400fe2:	39 f9                	cmp    %edi,%ecx
  400fe4:	7e 0c                	jle    400ff2 <func4+0x24>
  400fe6:	8d 51 ff             	lea    -0x1(%rcx),%edx
  400fe9:	e8 e0 ff ff ff       	callq  400fce <func4>
  400fee:	01 c0                	add    %eax,%eax
  400ff0:	eb 15                	jmp    401007 <func4+0x39>
  400ff2:	b8 00 00 00 00       	mov    $0x0,%eax
  400ff7:	39 f9                	cmp    %edi,%ecx
  400ff9:	7d 0c                	jge    401007 <func4+0x39>
  400ffb:	8d 71 01             	lea    0x1(%rcx),%esi
  400ffe:	e8 cb ff ff ff       	callq  400fce <func4>
  401003:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  401007:	48 83 c4 08          	add    $0x8,%rsp
  40100b:	c3                   	retq 
```

