---
layout: "post"
title: "「Core Java」 03 Interfaces, Lambda, Inner Classes"
subtitle: "接口，lambda 表达式和内部类"
author: "roife"
date: 2021-01-28

tags: ["Java@L", "Core Java@B", "未完成@D"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# 接口

## interface 简介

Java 中的 interface 有点像 Swift 的 protocol 或者 C++ 的纯虚基类，用 `interface` 声明，其中的方法默认是 `public` 的（所以不需要访问控制符）。

```java
public interface Comparable<T> {
    int compareTo(T other);
}
```

声明实现某个 interface 用 `implements`。

```java
class Employee implements Comparable<Employee> {
    public int compareTo(Employee other) {
        return Double.compare(salary, other.salary);
    }
}
```

- `java.util.Arrays`
  + `static void sort(Object[] a)`
  : 使用 mergesort 算法对数组 `a` 中的元素进行排序。 要求数组中的元素必须属于实现了 `Comparable` 接口的类，并且元素之间必须是可比较的。
- `java.lang.Integer` / `java.lang.Double`
  + `static int comparednt x, int y)`
  : 如果 `x<y` 返回一个负整数；如果 `x` 和 `y` 相等，则返回 `0`；否则返回一个负整数

## `compareTo(x)`

语言规定 `compareTo(x)` 必须保证反对称性，即 `sgn(x.compareTo(y)) = -sgn(y.compareTo(x))`。基于这个原因，比较时不能进行简单的类型转换：

```java
class Employee implements Comparable<Employee> {
    // ..
}

class Manager extends Employee {
    public int compareTo(Employee other) {
        Manager otherManager = (Manager) other; // NO
    }
}
```

如果这么写，那么 `e.compareTo(m)` 正常，但是 `m.compareTo(e)` 会抛出 `ClassCastException` 异常，这就不符合反对称性，所以要求 `compareTo()` 方法在一开始先检测两个类是否相同，不同则直接抛出 `ClassCastException`。

```java
if (getClass() != other.getClass()) throw new ClassCastException();
```

但是如果要求允许继承关系的类之间比较，那么应该只在父类上提供 `compareTo()`，并且将其设置为 `final`。

## interface 的特性

- 不能用 `new` 创建 interface 的对象，但是能创建 interface 的引用，表示指向的对象实现了这个 interface。

```java
x = new Comparable(); // Error
Comparable x; // OK
```

- interface 也可以用 `instanceof` 检查：`if (anObject instanceof Comparable)`。

- interface 也可以存在继承之类的关系，用来扩展 interface。

```java
public interface Powered extends Moveable {
    // ...
}
```

- interface 中可以包含常量。在其中定义域默认为 `public static final`

- 每个类只能够拥有一个超类，但却可以实现多个 interface（因此只能继承一个抽象类，但是能实现多个 interface）

```java
class Employee implements Comparable, Comparable
```

- Java SE8 开始允许静态方法，这样就不需要专门写一个静态方法的伴随类了（比如 `Collection/Collections`）

## 静态方法

## 默认方法

可以为方法提供一个默认实现，用 `default` 修饰。

```java
public interface Comparable<T> {
    default int compareTo(T other) { return 0; } // By default, all elements are the same
}
```

默认方法有两个用处：
- 只关心某些特殊方法，而不实现所有方法
- 接口演化。假设为旧 interface 添加了一个新方法，默认方法可以让人不花时间去为旧代码提供新方法的实现，只要执行默认方法即可

### 默认方法冲突

如果父类实现了默认方法，那么会执行父类的默认方法（子类也默认执行父类方法，即类优先）。

如果两个 interface 实现了签名相同的方法，而且某个 interface 提供了默认实现，会产生接口冲突，需要在代码中明确指出调用的方法。

```java
class Student implements Person, Named {
    public String getName() { return Person.super.getName(); }
}
```

## interface 示例

### callback

Java 中的 callback 通常是传入一个对象，并且对象实现了指定的 interface，那么对方就能调用指定的 callback 函数了。

```java
class TinePrinter implements ActionListener {
    public void actionPerformed(ActionEvent event) {
        // ...
    }
}

ActionListener listener = new TimePrinter();
Timer t = new Timer(10000, listener);
```

### clone

默认的 `clone()` 是一个浅拷贝，即 `clone()` 时对于成员不会递归地进行 `clone()`，只 `clone()` 一层。

因此，如果成员是可变的，那么有必要自定义 `clone()` 方法。

```java

```

Admitted

# Lambda Expr

匿名函数（参数类型可以由编译器推导）。

```java
(String first, String second) ->
{
    if (first.length() < second.length()) return -1;
    else if (first.length() > second.length()) return 1;
    else return 0;
}

// 推导参数类型

(first, second) ->
{
    if (first.length() < second.length()) return -1;
    else if (first.length() > second.length()) return 1;
    else return 0;
}

// 无参数 lambda 表达式

0 -> { for (int i = 100; i >= 0; i-- ) System.out.println(i); }

// 单参数可推导类型 lambda 表达式

event -> System.out.println("The time is " + new Date());
```

lambda 表达式的类型可以让编译器进行推导。如果一个 lambda 表达式只在某些分支返回一个值，而在另外一些分支不返回值，那么是不合法的，如：`(int x) -> { if (x >= 0) return 1; }` 不合法。


