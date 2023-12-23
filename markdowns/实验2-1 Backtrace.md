## 一、实验介绍

在本次实验中，你将学习到什么是`backtrace`，以及如何去实现它。

## 二、实验目的

1. 理解并掌握C程序的栈帧组织结构
2. 会使用常用工具帮助完成调试

## 三、实验要求

1. 实现要求的`backtrace`函数要求
2. 提交代码

## 四、实验步骤

### 1. 浅尝backtrace

我们知道，实现函数调用的基础在于一个名叫**栈**的数据结构，通过栈，我们可以保存过往的函数调用上下文，从而使得我们的程序拥有‘记忆’。接下来我们将使用gdb来帮助体验一下backtrace的神奇魅力。

我们可以在项目的根目录下创建一个新的文件`hello.c`，并编写如下代码：

```c
#include <stdio.h>

void bar() {
    printf("bar\n");
}

void foo() {
    printf("foo\n");
    bar();
}

int main() {
    foo();
    return 0; 
}
```

这个代码构造了一条三层的函数调用链：`main() ---> foo() ---> bar()`，接下来构建并调试它：

```
$ gcc hello.c -o hello # 编译产生hello可执行程序
$ gdb hello            # 调试该程序
# ...
Reading symbols from hello...
(No debugging symbols found in hello)
(gdb) break bar # 在bar函数入口打断点
Breakpoint 1 at 0x1151
(gdb) run # 开始运行
Starting program: hello 
foo

Breakpoint 1, 0x0000555555555151 in bar ()
(gdb) backtrace # 打印当前函数调用栈
#0  0x0000555555555151 in bar ()
#1  0x0000555555555184 in foo ()
#2  0x0000555555555199 in main ()
```

你可以看到相关函数调用栈被打印了出来。

> 在编译时加上编译选项`-g`，看看最后输出是否会发生变化？

### 2. 实现backtrace

在这个部分，我们将要自己动手实现`backtrace`这个功能。

在RISC-V架构中，我们通常使用`fp(frame pointer)`寄存器来帮助保存栈帧的地址，通过不断追溯栈帧，我们便可以获取函数的调用信息。请阅读栈帧的[详细描述](https://pdos.csail.mit.edu/6.1810/2022/lec/l-riscv.txt)，完成`src/utils.c::backtrace`函数：

```c
void backtrace() {
    // TODO:
    // follow the frame and get(print) return address from the frame
}
```

> *一些提示*
>
> 1. `uint64_t fp; asm volatile("mv %0, fp" : "=r" (fp));`可以帮助你读取当前`fp`寄存器中的值。
> 2. 循环可以帮助你有效的遍历栈帧，那么循环的终止条件又该是什么呢？
> 3. 使用`current->kstack`可以读取内核栈的地址，`+ KSTACK_SIZE`即可获取栈的起始地址（栈是向下增长的！）
> 4. 使用`kprintf`来输内核出信息

使用`riscv64-unknown-elf-addr2line -e build/kernel`可以将输入的地址转换为文件中对应的行号：

```
$ riscv64-unknown-elf-addr2line -e build/kernel
80213224
src/kernel/proc.c:91
```

你可以在任意的内核函数中插入你实现的backtrace，来检查输出是否正确。

## 五、思考

1. 依赖`fp`指针遍历栈帧真的绝对有效吗？在`scripts/cflags.mk`中的编译选项`-fno-omit-frame-pointer`有什么作用呢？
2. 尝试使用gdb调试qemu中的内核，看看能否做到和普通用户程序相似的调试体验？