# gdb常用指令

`gdb -rui bomb`：进入可视化界面

`layout split`：可以看源码和汇编代码

`layout reg`：可以看寄存器，方便直接引用寄存器地址，找到地址的内容

`b main`：函数打断

`b *0x80483c3 `：地址断点

`stepi`：下一步

`next`：（跳过函数）下一步

`continue`：进入下一个断点

`x/s 0xffffffff`：打印地址内容

`p /x test` ：打印变量的地址

`i b`： 查看设置的断点

`disp i`：查看变量值

`i display`：查看配置的变量

`backtrace`：运行路径

`delete`：删除断点



