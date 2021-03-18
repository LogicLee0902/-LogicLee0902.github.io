---
layout: "post"
title: "「BUAA-OO-Lab」 01 表达式求导"
subtitle: "求导"
author: "roife"
date: 2021-03-14
tags: ["BUAA - 面向对象设计与构造实验@Courses@Series", "北航@Tags@Tags", "面向对象@Tags@Tags", "Java@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# 问题

[题目链接](/file/in-post/post-buaao-oo/Unit%201%20Differential%20Homework%203.pdf)

# 分析

我们要做的是对输入的表达式进行求导，并且化简输出，所以不难想到应该分为几步：

$$\mathrm{inStr} \xrightarrow{\mathrm{parse}} \mathrm{Expr} \xrightarrow{\mathrm{simpify}} \mathrm{simpE} \xrightarrow{\mathrm{getDerivative}} \mathrm{derivative} \xrightarrow{\mathrm{simplify}} \mathrm{simpD} \xrightarrow{\mathrm{display}} \mathrm{ansStr}$$

首先，用递归下降法解析输入，然后对得到的表达式进行第一次化简。之所以这里要插入一次求导，是因为求导的时候表达式可能会急剧膨胀，先化简有利于减小后面的负担。接着对化简结果进行求导，并且再次对结果进行化简，最后输出结果。

# 架构

我的代码分为了两个 package，其中 `parser` 用于解析输入，`expression` 用于处理表达式。这个架构还是比较直观的，分类方法是“因子类型”或者说是“求导法则”。

其中，`Expr` 是一个 `Term` 的容器，表示所有 `Term` 加起来。`Term`

```
|- MainClass
|- expression (package)
    |- Factor (abstract class)
        |- Constant
            | BigInteger value
        |- Var
        |- Sin
            | Factor content
        |- Cos
            | Factor content
        |- Power
            | Factor base
            | BigInteger exp
        |- Term
            | BigInteger coe
            | HashSet<Power> powers
        |- Expr
            | HashSet<Term> terms
|- parser (package)
    |- Parser
    |- Tokenizer
        |- TokenType (enum)
```

Factor 总共有三个方法：
- `abstract Factor getDerivative()`：求导
- `abstract String display()`：打印当前因子
- `abstract Factor simplify()`：化简因子
- `abstract Factor expand();`：展开因子
- `abstract boolean equals(Object o)`：比较
- `abstract int hashCode()`：求哈希值

# 解析类 `Parser`

我将 parser 分为两部分：`parser` 和 `tokenizer`。

解析前先在表达式两端添加一对括号，如 `sin(x)` 变成 `(sin(x))`。

## `Tokenizer`

Tokenizer 负责将输入表达式字符串分解成基本的语法单元，并且记录其类型。

例如 `2 * sin((2*x))**2` 解析为 `[2, *, sin, (, (, 2, *, x, ), ), **, 2, EOL]`。这样的好处是可以预先将不合法的字符和空白字符排除掉，减少 parser 的复杂度。

解析完可以用以下方法操作：
- `getTokenType()` 获取当前语法单元类型
- `consumeToken()` 吃掉一个语法单元
- `getToken()` 获取语法单元
- `getTokenPos()` 获取语法单元在字符串中的位置（用来应对“带符号整数中不能有空格”的情况）

语法类型有以下几种。

```java
public enum TokenType {
    ERR, EOL,
    NUM, X, LPAREN, RPAREN,
    PLUS, MINUS, MULTI, EXP, SIN, COS
}
```

其实特殊的两种是 `ERR` 和 `EOL`，前者表示解析错误（遇到了非法的语法单元），后者表示读到行尾。

## 递归下降 `Parser`

Parser 使用递归下降法解析表达式（不推荐用正则表达式）。这里简单介绍一下。

递归下降是一种根据形式化语法对字符串进行解析的算法。

递归下降的思想是对于每一个语法元素，都用单独的函数对其进行读取。例如对 `sin` 的解析。首先在 Tokenizer 里面已经去掉了空白项，所以语法变成 `sin（因子）`，由三部分组成：`sin()` 和因子。其中难点在于因子的解析，使用递归下降时只要直接调用对应的解析函数 `consumeXXX()` 即可。

`consumeXXX()` 的要点在于，调用时要只要保证当前位置恰好在 `XXX` 的开头，而且调用完他就能解析完这个元素，而具体这些儿子部分是怎么解析的就交给对应的方法。可以看出这个方法比较像分治。

```java
private static Sin consumeSin() throws WrongFormatException {
    // parse sin
    if (tokenizer.getTokenType() == TokenType.SIN) {
        tokenizer.consumeToken();
    } else {
        // if not, throw error
        throw new WrongFormatException();
    }

    // parse (
    if (tokenizer.getTokenType() == TokenType.LPAREN) {
        tokenizer.consumeToken();
    } else {
        throw new WrongFormatException();
    }

    // parse factor
    Factor factor = consumeFactor();

    // parse )
    if (tokenizer.getTokenType() == TokenType.RPAREN) {
        tokenizer.consumeToken();
    } else {
        throw new WrongFormatException();
    }

    return new Sin(factor);
}
```

另一个例子就是对 `Expr` 的解析。`Expr ::= ([+-] Term [+-] Term [+-] ...)`。读取第一项前要先判断是否存在 `[+-]`，然后每次读取的 `Term` 存在一个 `ArrayList<Term>` 中。

```java
private static Expr consumeExpr() throws WrongFormatException {
    // parse (
    // ...

    BigInteger sign = BigInteger.ONE; // +-
    if (tokenizer.getTokenType() == TokenType.MINUS) {
        tokenizer.consumeToken();
        sign = sign.negate();
    } else if (tokenizer.getTokenType() == TokenType.PLUS) {
        tokenizer.consumeToken();
    }

    ArrayList<Term> terms = new ArrayList<>();
    terms.add(consumeTerm() * sign);

    while (tokenizer.getTokenType() != TokenType.RPAREN) {
        switch (tokenizer.getTokenType()) {
            case PLUS:
                tokenizer.consumeToken();
                terms.add(consumeTerm(BigInteger.ONE));
                break;
            case MINUS:
                // ...
            default:
                throw new WrongFormatException();
        }
    }

    // parse )
    // return
    // ...
}
```

# 表达式类

## 显示`display`

打印的过程比较简单，这里讲一些细节：

- `Power`
  - 注意 `Expr` 和 `Term` 不能用乘方运算，所以如果 `base instanceof Expr | Term` 的话，我们需要输出时进行展开：
  ```java
  displayString = Stream.generate(() -> baseDisplay)
                    .limit(exp.intValue())
                    .collect(Collectors.joining("*"));
  ```
  - 如果 `exp` 为 1，那么不需要加上 `**exp`
  - 可以进行一个优化：`x**2 -> x*2`
  - 否则返回 `base**exp`
- `Term`
  - 如果 `powers` 为空，直接返回系数
  - 如果系数为 1，返回 `powerStr`；如果系数为 -1，返回 `-powerStr`
  - 否则返回 `(coe * power1 * power2 *...)`，这里之所以要加一对括号，是因为对于 `sin((2*x))` 的情况
- `Expr`
  - 如果 `terms` 为空，直接返回 0
  - 将所有项转换为字符串后，可以将正项放前面（比如 `x-1` 比 `-1+x` 更好）
  - 如果开头是 `+` 那么去掉它
  - 返回 `([-]term1 [+-] term2 [+-] ...)`

## 求导 `derivative`

求导直接按照求导法则做就好了，我们这样的分类让求导变得异常简单。这里以 `Power` 为例：

```java
// (f**a)' = a * f**(a-1) * f'
ArrayList<Power> factors = new ArrayList<>();
factors.add(new Power(base, exp.subtract(BigInteger.ONE))); // f**(a-1)
factors.add(new Power(base.derivative())); // f'
return new Term(exp, factors); // a * f**(a-1) * f'
```

比较复杂的一个可能是 `Term` 的求导：

```java
// (coe1 * p1 * p2 * ...)'
// = (coe * p1' * p2 * ...) + (coe * p1 * p2' * ...) + ......
ArrayList<Term> derivedTerms = new ArrayList<>();
this.powers.forEach(power -> {
    ArrayList<Power> newTerm = new ArrayList<>(this.powers);
    Term derivedPower = power.derivative(); // pi'
    newTerm.remove(power); // p1 * p2 * ... * pi-1 * pi+1 * ...
    newTerm.addAll(derivedPower.getPowers());

    derivedTerms.add(new Term(this.coe.multiply(derivedPower.getCoe()), newTerm));
    // coe * p1 * p2 * ... * pi-1 * pi+1 * ...
});

return new Expr(derivedTerms); // add
```

# 化简 `simplify`

`simplify()` 函数编写的思路是，`f.simplify()` 返回的一定是最简形式。例如一个 `Expr` 的 `2`，返回的是一个 `Constant` 类型，而不是 `Expr(Term(Power(Constant)))`。之所以这样是因为如果你要合并两个同类项，那么要保证两个式子都化简到最简形式，才能保证哈希值相同。

## Trivial

### `Power`

- $0^{\mathrm{exp}} = 0$
- $1^{\mathrm{exp}} = 1$
- $Y^0 = 1$
- $(Y^{\mathrm{e1}})^{\mathrm{e2}} = Y^{\mathrm{e1} * \mathrm{e2}}$

主要如果 `base` 是常数，那么也要展开。

### `Term`

- $0 * [Y] = 0$

## 类型拆包

在化简的过程中可能会遇到类型过度包装的问题。比如一个简单的数字，parse 的时候可能变成了一个 `Expr(Term(Power(Constant(2), 1), 1))`，即 $1 * 2^1$。显然这样是不利于我们化简的，因为 `equals` 和 `hashCode()` 会判断这样的式子和 $2$ 不等价。因此我们要进行“拆包”。

实际上类型拆包并没有化简什么，只是减少了类型的封装层次。

类型拆包可以在返回化简结果前调用。

- `Power`
  - 拆 `Power`：$Y^1 = Y$
- `Term`
  - `Powers` 为空：$C * [\ ] = C$
  - 拆 `Term`：$1*[Y^{\mathrm{exp}}] = Y^{\mathrm{exp}}$
  - 拆 `Term`和 `Power`：$1*[Y^{\mathrm{1}}] = Y$
- `Expr`
  - `Terms` 为空：$[\ ] = 0$
  - `Terms` 只有一项 $(T_0)$
    - Powers 为空 $T_0 = C*[\ ]$：$(C*[\ ]) = C$
    - 系数为 1，而且 Powers 只有一项 $T_0 = 1 * [Y^{\mathrm{exp}}]$：$(1 * [Y^{\mathrm{exp}}]) = Y^{\mathrm{exp}}$

还有一些拆包工作是**展开嵌套类型**。这些展开需要根据当前式子的类型对于子因子进行的拆包，比如 `Expr` 内套 `Expr` 这种，需要在子因子被化简后，对化简结果进行拆包：

- `Term`
  - 提取常数：$C * [C_1 * Y] = C * C_1 * [Y]$
  - 展开嵌套 `Term`：$C * [(C_0 * Y_0) * Y_1] = C * C_0 * [Y_0 * Y_1]$
- `Expr`
  - 展开嵌套 `Expr`：$((T_0) + T_1) = T_0 + T_1$

类似的还有装包，在合并同类项之前用，便于统一类型进行合并：
- `Power.pack(f)`
  - 幂（不变） $Y^{\mathrm{exp}} = Y^{\mathrm{exp}}$
  - 否则 $Y = Y^1$
- `Term.pack(f)`
  - 项（不变） $C * [Y] = C * [Y]$
  - 常数 $C = C*[\ ]$
  - 指数 $Y^{\mathrm{exp}} = 1 * [Y^{\mathrm{exp}}]$
  - 否则 $Y = 1 * [Y^1]$

## 合并同类型

合并同类型的一个简单思路是递归比较两个对象是否相等（即先比较儿子，如果都相等然后比较自己）。但是这样递归比较的复杂度可能会很大，所以我们不难想到可以用 hash 来实现比较的加速。因此在 `Expr` 和 `Term` 中，我用 `HashSet` 来存储儿子，而不用 `List`，因为 `List` 在计算哈希值时会考虑到元素的相对顺序问题，而 `Set` 则无关顺序，只要成员相同可以。

为了使用 `HashSet`，我们需要为每一个类定义 `hashCode` 和 `equals` 方法。这个方法直接由 IDEA 生成即可。

我们知道，`Set` 里面的元素都是不重复的，但是我们一开始肯定有一些重复的元素，经过合并之后才能放到 `Set` 里面。所以显然这些可能重复的元素应该存在 `List` 中。所以我们的构造器应当是这样的：接受一个 `List`，调用一个合并表达式的参数将其转换成一个 `Set`。

假设现在我们在 `Expr` 中，手上一个有一个 `ArrayList<Term> terms`。我们先创建一个 `HashMap`，这样在合并就可以用一个成员方法：`void merge(key, value, remapping)`。其中 `key` 表示键值，应该用除去常数的因子，即 `term.getPowers()`，`value` 表示插入的值，即系数 `term.getCoe()`，`remapping` 表示一个函数，当插入键重复的时候使用。这里我们希望如果两个项的 `key` 相同，那么进行合并，所以我们可以用 `BigInteger::add` 作为参数。合并完成后，我们去掉所有系数为 0 的项。

```java
HashMap<HashSet<Power>, BigInteger> hashMap = new HashMap<>();
terms.forEach(term ->
    hashMap.merge(term.getPowers(), term.getCoe(), BigInteger::add));
hashMap.entrySet().removeIf(entry -> entry.getValue().equals(BigInteger.ZERO));
```

接下来要做的就是把这个 `HashMap` 转换成 `HashSet`，这一步比较简单：

```java
HashSet<Term> hashSet = new HashSet<>();
hashMap.forEach((powers, coe) -> hashSet.add(new Term(coe, powers)));
return hashSet;
```

`Term` 的合并和这个几乎是一模一样的，因为因子的合并也是将指数相加。因为我们可以发现 `Expr` 和 `Term` 其实是很像的两个东西，唯一的区别在于二者连接儿子的是用加号还是乘号。

前面化简子因子的时候已经对于所有子因子进行类型拆包，但是这里我们合并的时候要封装成统一的类型（例如合并 Expr 要让所有子因子都变成 `Term`）。

- `Term`
  - `Constant c`：`coe *= c`
  - `Term t`：`coe *= t.coe`，`powers += t.powers`
  - `else f`：`powers += Power.pack(f)`
- `Expr`
  - `Expr e`：`terms += e.terms`
  - `else f`：`terms += Term.pack(f)`

## 三角优化

关于三角函数，我只做了一个优化：$a \sin^2 x + b \cos^2 x + c$。
我的思路是做这个优化有两种可能，要么是 $\sin \rightarrow \cos$，要么是 $\cos \rightarrow \sin$，这两个涵盖了所有情况，包括 $1 - \sin^2 x$ 这样的。（想一想，为什么）

这个合并过程显然在 `Expr` 中进行，针对 `Term` 进行合并。

合并时，我们先创建三个 `HashMap`，分别存储含有 $\sin^2 x$，含有 $\cos^2 x$ 以及不含二者的项。（考虑到会有 $\sin^2 x * \cos^2 x$，为了方便我将其当作 $\sin^2 x$，不考虑同一个 `HashMap` 内部两个元素的合并了）。将 `term.getPowers()` 去掉对应的项后作为 key 存入 `HashMap` ，将 `term` 的系数作为 value。

例如 `2 * x**2 * sin(x)**2`，我们去掉 `sin(x)**2`，将 `x**2` 作为 `key`，将 `2` 作为 value 存入 `hashMapSin`。

```java
for (Term term : terms) {
    HashSet<Power> powers = term.getPowers();
    if (powers.contains(sinX2)) { // sin | sin*cos
        powers.remove(sinX2);
        mapSinX2.put(powers, term.getCoe());
    } else if (powers.contains(cosX2)) { // only cos
        powers.remove(cosX2);
        mapCosX2.put(powers, term.getCoe());
    } else {
        mapConst.put(powers, term.getCoe());
    }
}
```

然后我们遍历 $\sin^2 x$ 的所有项，并且记下 value 为 `a`，同理在 $\cos^2 x$ 和不含二者的 `HashMap` 中查找这个 key，结果分别记为 `b`，`c`（如果找不到那么就是 0）。即我们现在有 $a \sin^2 x + b \cos^2 x + c$。那么接下来有两种可能：

- $\sin \rightarrow \cos : \sin^2 = 1 - \cos^2 \Rightarrow (b-a)\cos^2 + (a+c)$
- $\cos \rightarrow \sin : \cos^2 = 1 - \sin^2 \Rightarrow (a-b)\sin^2 + (b+c)$

那么我们要在两个里面选一个，显然就是比较显示的时候二者长度比较短的那一个。只要将对应的系数 `toString()` 后加起来即可。这里需要考虑一点小细节（这里会比较丑陋）：
1. 如果某个数字（比如 $a+c$）是 0，那么它不会显示，不算入长度
2. 如果两个数字都是正数，那么它们之间要用一个加号连接，即长度会加一

假如我们选择了 $\sin \rightarrow \cos$，那么只要将 $\sin$ 系数变为 `0`，另外两个根据公式在对应的 `HashMap` 中存入新值即可存入。即执行以下操作：

```java
if (sin2cos <= cos2sin) { // sin -> cos
    mapConst.put(key, apc); // a + c
    mapCosX2.put(key, asb.negate()); // a - b
    mapSinX2.put(key, BigInteger.ZERO); // 0
} else { // cos -> sin
    mapConst.put(key, bpc); // b + c
    mapCosX2.put(key, BigInteger.ZERO); // 0
    mapSinX2.put(key, asb); // a - b
}
```

最后，我们将 `HashMap` 里面的内容恢复到 `terms` 中。遍历 `hashMapSin`，如果一个 `key` 的 `value != 0`，那么复制一下这个 `key`，然后加入 $\sin^2$（因为这个 `key` 对象可能被多个 `HashMap` 共享，我们不能干扰另外两个，所以要先复制一份）。再将其加入 `terms` 即可。

## 括号展开

我采用的策略是“要么全部不展开”，“要么全部展开”。

为了方便表达式化简，需要事先定义 `Term * Term` 和 `Expr * Expr` 的运算。

括号展开主要是三个方面：
- `Power`：对于 $(Y)^{\mathrm{exp}}$ 形式的表达式需要展开
- `Term`：对于 $C * A * B * (Y) * (Z)$，首先提取出非表达式部分将其转换成 `Expr` $(F) = (C * A * B)$，然后用乘法计算 $(F) * (Y) * (Z)$。
- `Expr`：对于每一个 Term 进行展开，然后合并

需要注意的是对于每一个子因子展开后都要进行化简，避免出现类型过度包装。