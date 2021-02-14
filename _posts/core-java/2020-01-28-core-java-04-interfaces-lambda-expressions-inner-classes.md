---
layout: "post"
title: "「Core Java」 03 Interfaces, Lambda, Inner Classes"
subtitle: "接口，lambda 表达式和内部类"
author: "roife"
date: 2021-01-28

tags: ["Java@Languages@Tags", "Core Java@Books@Series"]
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

`Object` 类中的 `clone()` 方法是 `protected` 的，因此不可以直接调用。如果要调用的话，需要自己实现 `Clonable` interface，同时重新定义 `clone()` 方法并设置为 `public`。

默认的 `clone()` 是一个浅拷贝，即 `clone()` 时对于成员不会递归地进行 `clone()`，只 `clone()` 一层。因此，如果成员是可变的，那么有必要自定义 `clone()` 方法（成员都是不可变的话，那显然就没必要了）。

```java
// 浅拷贝
class Employee implements Clonable {
    public Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }
}

// 深拷贝
class Employee implements Clonable {
    public Employee clone() throws CloneNotSupportedException {
        Employee cloned = (Employee) super.clone();

        // clone mutable fields
        cloned.hireDay = (Date) hireDay.clone();

        return cloned;
    }
}
```

如果 `clone()` 时某个对象不支持 `clone()`，那么就会抛出 `CloneNotSupportedException` 异常。

在继承关系中，`clone()` 可能会带来问题。如果父类实现了 `clone()`，那么就可以通过动态绑定进行子类的 `clone()`。如果子类没有定义 `clone()`，而且需要深拷贝或者存在不能拷贝的域，那么就会出错。

# Lambda Expr

## lambda expr 语法

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


## 函数式接口

只包含一个抽象方法的 interface 被称为函数式接口，可以用 lambda 表达式，即 lambda 表达式会被转换为接口。

Java 中没有函数类型，但是通过接口实现了 lambda 表达式。

### 常用函数式接口

| 函数式接口            | 参数   | 返回类型  | 抽象方法名 | 描述                         | 其他方法                         |
|-----------------------|--------|-----------|------------|------------------------------|----------------------------------|
| `Runnable`            | 无     | `void`    | `run`      | 作为无参数或返回值的动作运行 |                                  |
| `Supplier<T>`         | 无     | `T`       | `get`      | 提供一个 `T` 类型的值        |                                  |
| `Consumer<T>`         | `T`    | `void`    | `accept`   | 处理一个 `T` 类型的值        | `andThen`                        |
| `BiConsumer<T, U>`    | `T, U` | `void`    | `accept`   | 处理 `T` 和 `U` 类型的值     | `compose`，`andThen`，`identity` |
| `Function<T, R>`      | `T`    | `R`       | `apply`    | 有一个 `T` 类型参数的函数    | `andThen`                        |
| `BiFunction<T, U, R>` | `T, U` | `R`       | `apply`    | 有 `T` 和 `U` 类型参数的函数 | `compose`，`andThen`，`identity` |
| `UnaryOperator<T>`    | `T`    | `T`       | `apply`    | 类型 `T` 上的一元操作符      | `andThen`，`maxBy`，`minBy`      |
| `BinaryOperator<T>`   | `T, T` | `T`       | `apply`    | 类型 `T` 上的二元操作符      | `andThen`                        |
| `Predicate<T>`        | `T`    | `boolean` | `test`     | 布尔值函数                   | `and`，`or`，`negate`，`isEqual` |
| `BiPredicate<T, U>`   | `T, U` | `boolean` | `test`     | 有两个参数的布尔值函数       | `and`，`or`，`negate`            |

```java
public static void repeat(int n, Runnable action) {
    for (int i = 0; i < n; i++) action.run();
}

repeat(10, () -> System.out.println("Hello, World!"));
```

### 基本类型函数式接口

对于基本类型有特殊的接口，可以减少自动装箱，因此对于基本类型应该尽量使用这些接口。

| 函数式接口            | 参数   | 返回类型  | 抽象方法名     |
|-----------------------|--------|-----------|----------------|
| `BooleanSupplier`     | 无     | `boolean` | `getAsBoolean` |
| `PSupplier`           | 无     | `p`       | `getAsP`       |
| `PConsumer`           | `p`    | `void`    | `accept`       |
| `ObjPConsumer<T>`     | `T, p` | `void`    | `accept`       |
| `PFunction<T>`        | `p`    | `T`       | `apply`        |
| `PToQFunction`        | `p`    | `q`       | `applyAsQ`     |
| `ToPFunction<T>`      | `T`    | `p`       | `applyAsP`     |
| `ToPBiFunction<T, U>` | `T, U` | `p`       | `applyAsP`     |
| `PUnaryOperator`      | `p`    | `p`       | `applyAsP`     |
| `PBinaryOperator`     | `p, p` | `p`       | `applyAsP`     |
| `PPredicate`          | `p`    | `boolean` | `test`         |

`p, q` 为 `int, long, double`；`P, Q` 为 `Int, Long, Double`。

### 自定义函数式接口

自定义接口可以用 `@FunctionalInterface` 标记，保证 interface 满足函数式接口定义，并且在导出后 javadoc 会将其标记为函数式接口。

## 方法引用

### 普通方法引用

直接用方法名作为 lambda 表达式。

- `object::instanceMethod`：`System.out::println` 等价于 `x -> System.out.println(x)`，可以是 `this::instanceMethod` 或 `super::instanceMethod`
- `Class::static Method`：`Math::pow` 等价于 `(x, y) -> Math.pow(x, y)`
- `Class::instanceMethod`：第 1 个参数会成为方法的目标，即 `String::compareToIgnoreCase` 等价于 `(x, y) -> x.compareToIgnoreCase(y)`

如果由重栽方法，那么编译器会自行进行推断。

### 构造器引用

用 `new` 作为构造器的函数名，如 `Person::new` 等价于 `str -> Person(str)`。

```java
ArrayList<String> names = ...;
Stream<Person> stream = names.stream().map(Person::new);
List<Person> people = stream.collect(Col1ectors.toList());
```

## 变量作用域

- 自由变量：lambda 表达式可以捕获自由变量，但是要求捕获的变量值初始化后就不能改变，即隐式的 `final`。
- 变量作用域：lambda 表达式中内部变量的作用域与外部相同，因此内部变量不能与外部同名

```java
Path first = Paths.get("/usr/bin");
Comparator<String> comp =
    (first, second) -> first.length() - second.length(); // first 与外部同名，错误
```

- `this`：lambda 表达式中使用 `this` 表示方法所在对象，而不是方法本身

```java
public class Application() {
    public void init() {
        ActionListener listener = event -> {
            System.out.println(this.toString()); // this 表示 Application 类
        }
    }
}
```

## Comparator

Comparator interface 包含很多 static 方法用于比较：
- `Comparator.comparing()` 将类型 `T` 映射到一个可以用于比较的类型，如 `Arrays.sort(people, Comparator.comparing(Person::getName));`
- `thenComparing()` 用于比较第二关键字，如 `Arrays.sort(people, Comparator.comparing(Person::getLastName).thenComparing(Person::getFirstName));`
- `comparing()` 和 `thenComparing()` 可以指定比较器：`Arrays.sort(people, Comparator.comparing(Person::getName, (s, t) -> Integer.compare(s.1ength(), t.length())));`
- 对于基本类型有特定的静态函数防止装箱：`Arrays.sort(people, Comparator.comparingInt(p -> p.getName().length()));`
- `naturalOrder()` 表示正向排序，`reverseOrder()` 等价于 `naturalOrder().reversed()` 表示逆向排序
- 如果比较内容可能为 `null`，可以用 `nullFirst(cmp)` 或 `nullLast(cmp)` 按照 `cmp` 排序并使 `null` 排在开头或末尾：`Arrays.sort(people, comparing(Person::getMiddleName, nulIsFirst(naturalOrder())));`

# 内部类

