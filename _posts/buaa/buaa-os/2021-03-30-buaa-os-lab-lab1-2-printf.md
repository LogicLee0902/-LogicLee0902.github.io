---
layout: "post"
title: "「BUAA-OS-Lab」 Lab1-1 内核制作、启动和 printf"
subtitle: "printf 函数 & 系统启动"
author: "roife"
date: 2021-03-30

tags: ["BUAA - 操作系统@Courses@Series", "北航@Tags@Tags", "操作系统@Tags@Tags", "MIPS Assembly@Tags@Tags", "C@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# 课上

## Exam

### 题目

修改 `printf` 函数，使其支持输出结构体。结构体格式总共有两种：

```c
struct s1 {
    int a;
    char b;
    char c;
    int d;
}

struct s2 {
    int sizec;
    int c[sizec]; // 即保证 c 数组长度为 sizec
}
```

对于 `s1`，输出格式为 `{1,b,c,4}`；对于 `s2`，输出格式为 `{2,2,3}`，注意第一个输出的数字是 `sizec`。

### 分析

主要是传入结构体参数怎么访问到其中的元素。我们可以在 `print.c` 里面按照定义一个结构体，然后用箭头运算符去访问。

也可以直接访问地址，如 `*(addr + 4)`，但是要注意结构体内数据的排列要字对齐（尤其是 `char`）。

输出就模仿原来的代码即可，这里我用宏简化了代码。

```c
struct s1 {
    int a;
    char b;
    char c;
    int d;
};

struct s2 {
    int size;
    int c[]; // 注意这里不能写 int c[size]
    // 也可以写 int c[1] 等
};

void
lp_Print(void (*output)(void *, char *, int),
         void * arg,
         char *fmt,
         va_list ap)
{
    // ...

    int stypeid;

    // ...

    for (;;) {
        // ...

        /* check for struct*/
        stypeid = 0;
        if (*fmt == '$') {
            ++fmt;
            stypeid = Ctod(*fmt++);
        }

        switch(*fmt) {
            // ...
            case 'T':
                // 注意输出 {}, 时，width 为 0
                #define PrintC(c) \
                    { \
                        length = PrintChar(buf, '{', 0, ladjust); \
                        OUTPUT(arg, buf, length); \
                    }
                #define PrintInt(x) \
                    { \
                        num = x; \
                        if (num < 0) num = -num, negFlag = 1; \
                        length = PrintNum(buf, num, 10, negFlag, width, ladjust, padc, 0); \
                        OUTPUT(arg, buf, length); \
                        negFlag = 0; \
                    }

                PrintC('{');
                if (stypeid == 1) {
                    struct s1* addr = (struct s1*)va_arg(ap, int);

                    PrintInt(addr->a);
                    PrintC(',');

                    PrintC(addr->b);
                    PrintC(',');

                    PrintC(addr->c);
                    PrintC(',');

                    PrintInt(addr->d);
                } else if (stypeid == 2) {
                    struct s2* addr = (struct s2*)va_arg(ap, int);
                    long int size = addr->size;

                    // print size
                    PrintInt(addr->size);
                    PrintC(',');

                    // print array
                    for (i=0; i<size; ++i) {
                        PrintInt(addr->c[i]);
                        if (i != size - 1) {
                            PrintC(',');
                        }
                    }
                }
                PrintC('}');
                break;
            // other cases
        }
    }
}
```

## Extra

新建一个 `my_cal` 文件夹，并且创建三个文件：`Makefile`、`my_cal.c`、`my_driver.S`。

### Task 1

#### 题目

编写 `Makefile` 编译 `my_cal.c` 和 `my_driver.S`。

#### 分析

这个模仿 `boot` 和 `init` 下面的 `Makefile` 就好了，注意 `.S` 和 `.o` 都要编译。

```makefile
INCLUDES        := -I../include

%.o: %.c
        $(CC) $(DEFS) $(CFLAGS) $(INCLUDES) -c $<
%.o: %.S
        $(CC) $(CFLAGS) $(INCLUDES) -c $<

.PHONY: clean

all: my_driver.o my_cal.o

clean:
        rm -rf *~ *.o


include ../include.mk
```

### Task 2

#### 题目

编写 `my_driver.S` 实现三个函数：
- `char _my_getchar()`：从控制台读入字符
- `void _my_putchar(char)`：输出字符
- `void _my_exit()`：退出模拟器

有几点注意事项：
- `_my_getchar()` 读入一个字符时，要立即输出，这样才能在控制台上回显
- 如果 `_my_getchar()` 读入了 `\r`，那么要补充输出一个 `\n`
- 使用 `_my_getchar()` 的时候控制台可能没有输入（即读到的内容为 `0`），此时应该有一个循环不断访问，直到要字符为止
- 设备的物理地址为 `0x10000000`，`0x00` 是读写控制台的地址，`0x10` 是终止程序的地址。向 `0x00` 写入数据可以在控制台显示数据，从 `0x00` 读入数据可以从控制台读入数据
- 汇编函数的写法可以参考 `boot/start.S`

#### 分析

首先的问题是怎么用汇编写函数，观察 `boot/start.S` 发现其结构是这样的：

```mips
LEAF(NAME)
    /* ASM */
END(NAME)
```

所以仿照这个写就行了。读取传入参数、返回值、结束函数和普通的 MIPS 函数一样，访问 `$a0`/`$v0`/`$ra` 即可。

因为这里引用了一个头文件，里面提供了寄存器的简写方式。所以写寄存器的时候可以直接用 `a0`/`v0`/`ra`。

然后的问题就是要读写哪个地址。由于我们访问的是外设，所以不能经过 cache，观察 MIPS 的内存布局可以发现应该是 `kseg1`，起始地址为 `0xA0000000`，再加上物理地址 `0x10000000`，即应该要访问的内存为 `0xB0000000` 与 `0xB0000010`。

其实从 `drivers/gxconsole` 下面的 `console.c` 和 `dev_cons.h` 文件也不难发现读写外设访问的是 `0xB0000000` 这个地址，终止程序的是 `0xB0000010`。

```mips
#include <asm/regdef.h>
#include <asm/cp0regdef.h>
#include <asm/asm.h>

LEAF(_my_getchar)
    li t0, 0xB0000000
    loop:
        lb t1, 0(t0) /* read a char to $t1 */
        beq t1, zero, loop
        nop
    end_loop:
    sb t1, 0(t0)

    /* check \r */
    li t2, 13 /* \r */
    bne t1, t2, end_check_r
    nop
    check_r:
        li t2, 10 /* \n */
        sb t2, 0(t0)
    end_check_r:

    or v0, zero, t1
    jr ra
END(_my_getchar)

LEAF(_my_putchar)
    li t0, 0xB0000000
    sb a0, 0(t0)
    jr ra
END(_my_putchar)

LEAF(_my_exit)
    li t0, 0xB0000000
    sb zero, 0x10(t0)
    jr ra
END(_my_exit)
```