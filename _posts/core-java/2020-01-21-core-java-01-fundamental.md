---
layout: "post"
title: "「Core Java」 01 Fundamental"
subtitle: "Fundamental Programming Structures in Java"
author: "roife"
date: 2021-01-21

tags: ["L「Java」", "B「Core Java」"]
language: zh-CN
catalog: true
header-image: ""
header-style: text
katex: true
---

由于 Java 和 C/C++ 有不少共同点，所以这里只记录 Java 的特殊之处。

# 一个简单的 Java 程序

源代码的文件名必须与公共类的名字相同，如 `FirstClass.java`。运行已编译的程序时（`$ java FirstClass`），Java 虚拟机将从指定类中的 `main` 方法开始执行。

```java
public class ClassName
{
    public static void main(String[] args) {
        // program statements
    }
}
```

## 命令行参数

`main` 函数的 `args` 参数为接收的命令行参数。注意第一个参数不是 `java`。

```shell
$ java Message -g cruel world
# args[0]: "-g"
# args[l]: "cruel"
# args[2]: "world"
```

# 数据类型

## 整型

共四种，`int`，`short`，`long`，`byte`（没有 unsigned 类型，而且大小是**平台无关**的）。

二进制可以通过数字前面加 `0b` 或 `0B` 实现（类似有十六进制、八进制）。

可以为数字字面量添加下划线：`1_000_000`。

## 浮点型

共两种，`float`，`double`（默认 `double`）。

`Double_POSITIVE_INFINITY`，`Double.NEGATIVE_INFINITY` 和 `Double.NaN` 表示三种特殊的浮点数（注意不能直接用 `==` 判断，因为“非数值”都是不相同的）。

## char

Java 中除了普通的转义字符以外，以 `\u` 开头的字符表示 Unicode 转义字符，如 `\u005B`。

特别的，`\u` 开头的转移还能混在代码中使用，如 `//\u00A0 is a newline` 不代表是一个注释，被转义为换行符，因此会报错。

### Unicode 与 char

- 码点：如 `U+0041` 表示字符的编码
- 代码单元：一个字符可以用多个代码单元存储，如 `U+1D546` 可以分为两个代码单元：`U+D835` 和 `U+DD46`

Java 中的 `char` 描述了一个 UTF-16 的代码单元。

## boolean

`true` 和 `false`，注意整型值和布尔值之间不能进行隐式类型转换。

# 变量

- 变量初始化：用 `int a = 1;` 的形式
- 常量：用 `final` 指示，如果一个常量要跨方法使用，可以用 `static final` 定义

# 运算符

## 数学计算

- （冷知识）使用 `strictfp` 关键字可以保证浮点计算行为在各平台上的一致性（不允许实现者进行中间扩展，牺牲了精度和速度），如 `public static strictfp void main(String[] args)`。同理，此时应该使用 `StrictMath` 库代替 `Math` 库。
- `Math.floorMod` 方法：求余，保证结果是正数（即 $(a \bmod b + b) \bmod b$）

## 类型转换

Java 允许的隐式类型转换如下（虚线代表可能存在精度损失），其他的类型转换必须通过强制类型转换（`int a = (int) 1.2`）。

![implicit-conversions](/img/in-post/post-core-java/implicit-conversions.png)

## 右移运算符

- `>>` 是算术右移，`>>>` 是逻辑右移

## 逗号运算符（无）

Java 没有逗号运算符，但是可以在 for 循环中使用逗号

# 字符串

Java 的字符串可以看成 Unicode 字符序列。两个字符串拼接可以直接用 `+`。

## 不可变字符串

在 Java 中字符串不能操作单个字符，如果要改变字符串内容必须通过拼接实现。

```java
greeting = greeting.substring(0, 3) + "p!"; // Hello -> Help!
```

## 码点与代码单元

`length()` 可以返回代码单元的数量，得到实际的长度则需要用 `codePointCount()`。

返回第 `i` 个位置的代码单元可以用 `charAt(i)`，但是返回码点则要用 `offsetByCodePoints(0, i)` 或 `codePointAt(index)`。

所有码点用一个或者两个 `char` 表示。注意，`char` 存不下一个码点，要用 `int`。

如果要遍历所有码点，可以用：

```java
int cp = sentence.codePointAt(i);
if (Character.isSupplementaryCodePoint(cp)) i += 2;
else i++;
// 回退
--i;
if (CharacterssSurrogate(sentence.charAt(i))) --i;
int cp = sentence.codePointAt(i);
```

也可以用流实现遍历：

```java
int[] codePoints = str.codePoints().toArray();
String str = new String(codePoints, 0, codePoints.length());
```

## 常用 API

- `java.lang.string`
  + `char charAt(int index)`
  : 返回给定位置的代码单元。
  + `int codePointAt(int Index)`
  : 返回从给定位置开始的码点。
  + `int offsetByCodePoints(int startIndex, int cpCount)`
  : 返回从 `startIndex` 码点开始， 位移 `cpCount` 后的码点索引。
  + `int compareTo(String other)`
  : 按照字典顺序，相等返回 0，比 `other` 小返回负数，否则为正数。
  + `IntStream codePoints()`
  : 将这个字符串的码点作为一个流返回，一般用 `toArray` 将它们放在一个数组中。
  + `new String(int[] codePoints, int offset, int count)`
  : 用数组中从 `offset` 开始的 `count` 个码点构造一个字符串。
  + `boolean equals(0bject other)`
  : 比较是否相等
  + `boolean equalsIgnoreCase(String other)`
  : 忽略大小写比较是否相等
  + `boolean startsWith(String prefix)`，`boolean endsWith(String suffix)`
  : 如果字符串以 `suffix` 开头或结尾， 则返回 `true`
  + `int indexOf(String str)`，`int indexOf(String str, int fromIndex)`，`int indexOf(int cp)`，`int indexOf(int cp, int fromIndex)`
  : 返回与字符串 `str` 或代码点 `cp` 匹配的第一个子串的开始位置。这个位置从 `0` 或 `fromIndex` 开始计算。 如果在原始串中不存在 `str` 返回 `-1`。
  + `int lastIndexOf(String str)`，`int lastIndexOf(String str, int fromIndex)`，`int lastindexOf(int cp)`，`int lastindexOf(int cp, int fromIndex)`
  : 返回与字符串 `str` 或码点 `cp` 匹配的最后一个子串的开始位置。这个位置从原始串尾端或 fromIndex 开始计算。
  + `int length()`
  : 返回字符串的长度。
  + `int codePointCount(int startIndex, int endIndex)`
  : 返回 `[b, e)` 之间的码点数量。
  + `String replace(CharSequence oldString, CharSequence newString)`
  : 替换。可以用 `String`/`StringBuilder` 对象作为 CharSequence 参数。
  + `String substring(int beginIndex)`，`String substring(int beginIndex, int endIndex)`
  : 返回子串，为原串从 `[b, e)` 或到尾部的所有**代码单元**。
  + `String toLowerCase()`，`String toUpperCase()`
  : 大小写转换
  + `String trim()`
  : 删除原串头尾的空格。
  + `String join(CharSequence delimiter, CharSequence... elements)`
  : 用给定的定界符连接所有元素。


## StringBuilder

`StringBuilder` 可以用来构建字符串，比字符串拼接效率更高。拼接完记得要转换回字符串。

```java
StringBuilder builder = new StringBuilder();
builder.append(ch); // appends a single character
bui1der.append(str); // appends a string
String completedString = builder.toString();
```

- `builder.length()`
  : 返回长度
- `s.appencl(ch)`，`s.appencl(str)`，`s.append(cp)`
  : 追加一个字符 / 字符串 / 码点
- `s.setCharAt(i, c)`
  : 把第 `i` 位设置成 `c`
- `s.insert(oddset, str)`，`s.insert(offset, c)`
  : 在第 `offset` 处插入字符串 / 字符
- `s.delete(start, end)`
  : 删除 `[s, e)` 的字符序列
- `s.toString()`
  : 转换为 `String`

# 输入输出

## 输入

`Scanner` 类定义在 `java.util` 包中，需要预先导入。

```java
import java.util.*;

// ...
Scanner in = new Scanner(System.in);    // 标准输入
String name = in.nextLine();            // 下一行
String firstName = in.next();           // 下一个单词

int age = in.nextInt();                 // 读整数，类似有 nextDouble()

boolean nxt = in.hasNext();             // 是否还有单词，类似有 hasNextInt(), hasNextDouble()
```

如果要读取密码，可以用 `Console` 类的 `con.readPassword`，注意这个类只能读取一行。读取密码后存放在一维字符数组，而不是字符串中。

```java
Console cons = System.console();
String username = cons.readLine("Username: ");
char[] passwd = cons.readPassword("Password: ");
```

## 直接输出

```java
System.out.print(x);
```

## 格式化

格式化输出可以用 `printf` 也可以用 `String.format`。`%` 后面接转换符，表示输出变量的类型。中间可以加标志和宽度，且标志可以重复使用，用于控制输出格式。

```java
System.out.printf("%8.2f", x);
```

对于日期，使用 `%t + others` 的方式控制格式。

```java
System.out.printf("%tc", new Date()); // 打印当前的日期时间
```

对于多次重复使用的参数，有两种方法可以避免重复写参数：
- 使用索引 `number + $` 表示使用第几个参数（序号从 1 开始）
  : `System.out.printf("%1$s %2$tB %2$te, %2$tY", "Due date:", new Date());`
- 使用 `<` 表示使用上一个参数
  : `System.out.printf("%s %tB %<te, %<tY", "Due date:", new Date());`

![format](/img/in-post/post-core-java/format.png)

## 文件 I/O

读取时需要传入一个 File 对象，输出直接传入路径。

```java
Scanner in = new Scanner(Paths.get("myfile.txt"), "UTF-8");
// 注意不是 Scanner in = new Scanner("myfile.txt");
PrintWriter out = new PrintWriter("myfile.txt", "UTF-8");
```

如果读入时文件不存在，或者输出时文件名不符合操作系统要求，则会抛出异常，需要在函数后标注 `throws IOException`。

# 控制流

## 变量定义作用域

Java 不允许在嵌套的作用域中重复定义变量：

```java
public static void main(String[] args) {
    int n;
    {
        int n; // 错误
    }
}
```

## switch

Java 的 `switch` 和 C 行为一致。

使用 `-Xlint:fallthrough` 可以对缺少 `break` 的 `case` 提出警告（用 `@SuppressWamings("fallthrough")` 可以忽略警告）。

`case` 标签可以是 `char`/`byte`/`short`/`int` 的**常量表达式**，枚举常量或者字符串字面量。

在 `switch` 中使用枚举常量时可以不指定枚举名。

```java
Size sz = ...;
switch (sz) {
    case SMALL: // no need to use Size.SMALL ...
    break;
}
```

## 标签 break/continue

Java 没有 `goto`，但是语句块（循环/条件等）可以带标签，用 `break label` 实现跳出。

```java
label: {
    // ...
    if(condition) break label;//exitsblock ...
}
```

类似的，有带标签的 `continue`。

# 大数值

`BigInteger` 和 `BigDecimal` 是两个处理大数值和高浮点精度的类。运算时不能直接用运算符，必须用函数。

- `bi.valueOf`：
  : 转换为大数值 `Biglnteger a = Biglnteger.valueOf(100);`。其中 `BigDecimal` 可以指定阶数 `scale`，即 $a \times 10^{scale}$。
- `bi.add`，`bi.subtract`，`bi.multiply`，`bi.divide`，`bi.mod`（`BigInteger`）
  : 四则运算。`bi.BigDecimal` 在计算除法时**必须**指定舍入方式，如 `RoundingMode.HALF_UP` 为四舍五入。
- `bi.compareTo`
  : 比较大小，如果当前大于参数返回正数。

# 数组

## 定义和初始化

```java
int[] a = new int[100]; // 数组长度不要求是常量，可以为 0
// 也可以用 int a[]
```

创建时值会进行值初始化（`0`/`false`/`null`），也可以自己初始化。注意对于字符串来说，初始值不是 `""` 而是 `null`。

```java
int[] a = {1, 2, 3};
```

## foreach（range-for）

即 range for 结构。

```java
for (int element : a)
    System.out.println(element);
```

## 匿名数组

匿名数组可以在不创建新变量的情况下重新初始化数组。

```java
smallPrimes = new int[] { 17, 19, 23, 29, 31, 37 };
```

## 数组拷贝

数组默认为引用类型，所以拷贝时默认为浅拷贝。如果要拷贝出一个新数组则要用 `copyOf(arr, length)`。

这个方法可以用来增加数组的长度。

```java
luckyNumbers = Arrays.copyOf(luckyNumbers, 2 * luckyNumbers.length);
```

## 其它常用函数

- `Array.toString(arr)`
  : 返回字符串形式的数组
- `Array.copyOf(arr, length)`，`Array.copyOfRange(arr, start, end)`
  : 拷贝数组
- `Array.sort(arr)`
  : 排序
- `Array.binarySearch(arr, v)`，`Array.binarySearch(arr, start, end, v)`
  : 二分查找
- `Array.fill(arr, v)`
  : 充填
- `Array.equals(arr1, arr2)`
  : 比较是否相等

## 多维数组

Java 的数组实际上是数组的数组。

```java
double[][] balances = new double [10][6];

int[][] magicSquare = {
    {16, 3, 2, 13},
    {5, 10, 11, 8},
    {9, 6, 7, 12},
    {4, 15, 14, 1}
};
```

注意使用 foreach 访问多维数组时，外层变量应该为一个数组：

```java
for (double[] row : a)
    for (double value : row)
        // ...
```

打印多维数组可以用 `System.out.println(Arrays.deepToString(a));`。

### 不规则数组

Java 中多维数组的每一个维度的大小可以不同。

```java
int[][] odds = new int[NMAX + 1][];
for (int n = 0; n <= NMAX; n++)
    odds[n] = new int[n+1];
```
