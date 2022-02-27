---
layout:     post
title:      "「Head First Java」 03 Focus on the program"
subtitle:   进一步补充01的内容
date:       2022-02-10
author:     Leo

header-img: ""
catalog: true
tags: ["Java@Languages@Tags", "Head First Java@Books@Series"]
lang: zh
header-style: text
katex: true
---

*感觉《Head First Java》这本书写的有点乱*

# 书中提到编写程序的要点

* 伪代码需要包含实例变量声明，方法声明和**方法之间的逻辑**（重点）
* 写完伪代码之后不要急着代码，先写测试代码。（写测试代码的过程中，也能够再次理清题目，重新发现一些忽略 的情况和细节）
* 随机数`Math.random()`，需要`import java.util.*`

# 常用的两个包

## ArrayList

### 部分常用自带方法

* `add(Object elem)`：向list中加入对象参数

> 据说add有着很多灵活怪异的使用

* `remove(int index)`：依靠索引参数删除对象
* `remove(Object elem)`：移除对象
* `contains(Object elem)`: 查找是否包含该对象，如有返回True，否则返回False
* `isEmpty()`: 判断为空
* `indexOf(Object elem)`:返回对象参数的索引或-1

​	对应的有一个`lastIndexOf(Object elem)`返回最后依次出现的索引

* `size()`返回元素个数
* `get(int index)`:返回当前索引参数
* `clone()`:拷贝，是浅拷贝

> **浅拷贝**只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存， 所以**如果其中一个对象改变了这个地址，就会影响到另一个对象**。。
>
> 浅拷贝对应的就是**深拷贝**，深拷贝是将一个对象从内存中完整的拷贝一份出来,从堆内存中开辟一个新的区域存放新对象,且**修改新对象不会影响原对象**。

```java
import java.util.ArrayList;

class Main {
    public static void main(String[] args){

        // 创建一个数组
        ArrayList<String> sites = new ArrayList<>();

        sites.add("Google");
        sites.add("Runoob");
        sites.add("Taobao");
        System.out.println("网站列表: " + sites);

        // 对 sites 进行拷贝
        ArrayList<String> cloneSites = (ArrayList<String>)sites.clone();
        //注意格式
        System.out.println("拷贝 ArrayList: " + cloneSites);
    }
}
```

* `sort(Comparator c)`: 排序，comparator利用`java.util.Comparator`,升序是`Comparator.naturalOrder()`, 降序是`Comparator.reverseOrder()`
* `toArray([T[] arr])`:转换为数组，不填参数就返回Object类型的数组

```java
import java.util.ArrayList;
import java.util.Comparator;
class Main {
    public static void main(String[] args){

        // 创建一个动态数组
        ArrayList<String> sites = new ArrayList<>();
       
        sites.add("Runoob");
        sites.add("Google");
        sites.add("Wiki");
        sites.add("Taobao");
        System.out.println("网站列表: " + sites);

        // 创建一个新的 String 类型的数组
        // 数组长度和 ArrayList 长度一样
        String[] arr = new String[sites.size()];

        // 将ArrayList对象转换成数组
        sites.toArray(arr);

        // 输出所有数组的元素
        System.out.print("Array: ");
        for(String item:arr) {
            System.out.print(item+", ");
        }
    }
}
/* Output：
 *Array: Runoob, Google, Wiki, Taobao, */
```
* `removeRange(int fromIndex, int toIndex)`:删除索引之间的元素

### 简单的一些基本操作

```java
//create an arraylist
ArrayList<Egg> myList = new ArrayList<Egg>();
//add an element
Egg s = new Egg();
myList.add(s);
Egg b = new Egg();
myList.add(b);
//query for the size
int theSize = myList.size();

//query for the specific element
boolean isIn = myList.contains(s);

//query for the index of the elemen
int idx = myList.indexOf(b); //start from the 0

//check if it's empty
boolean empty = myList.isEmpty();

//delete the element
myList.remove(s);

```

把它当python中的list或C++中的vector用就还好。

----

Array为参数化类型，即`ArrayList<String>`, 其中`<String>`就是参数类型，用此可以方便声明

## String类

String类本身也是一个对象，它除了按照赋值方式`String s = "Leo"`创建对象以外，也可以按照构造函数的方式创建`String s = new String("Leo")`

此外，String还有别的初始化的方法，这边介绍一个字符数组的

```java
public class StringDemo{
   public static void main(String args[]){
      char[] helloArray = { 'r', 'u', 'n', 'o', 'o', 'b'};
      String helloString = new String(helloArray);  
      System.out.println( helloString );
   }
}
```

**注意:**String 类是不可改变的，所以你一旦创建了 String 对象，那它的值就无法改变了，如果需要修改就用StringBuffer&StringBuilder

### 方法

* `length()`:长度

* `concat()`: 连接两个字符串

  ```java
  "我的名字是 ".concat("Leo");
  ```

  但**更常用**也可以直接拼接+

  ```java
  public class StringDemo {
      public static void main(String args[]) {     
          String string1 = "Leo";     
          System.out.println("My Name is " + string1 + "!");  
      }
  }
  /*
  Output:
  My Name is Leo!
  */
  ```

* `charAt(inyt index)`: 返回索引处的char值

* `CompareTo(String s)`: 比较两个字符串：

  有两种模式

  **1、不同的字符在字符串长度之内时**

  返回值=原字符串与参数字符串中第一个不同字符相差的ASCII码值，为原减参。

  例子如下：

  ```java
  String str1="javasdrip";
  String str2="javdscript";
  str1.compareTo(str2);
  ```

  此时返回值**为-3**，是a的ASCII码（97）减去了d的ASCII码值（100）得到。

  注意：只比较第一个不同的字符，后面的d和c也不一样但不会进行比较了。

  **2、不同的字符在较短字符串长度之外时**（即一个串是另一个的子串时）

  返回值=原字符串与参数字符串相差的字符个数，原字符串长度大时为正，反之为负。

  例子如下：

  ```java
  String str1="java";
  String str2="javascript";
  str1.compareTo(str2);
  ```

  此时返回值为**-6**，是str1相比str2少去的字符个数。

  **注意**：此时只比较位数，而无关ASCII码值，并非是0的ASCII码值减去s的ASCII码值，在参数字符串前面字符和原字符串一样时，返回值就是两者相差的字符个数，即使改变后面的字符也不会影响到返回的值，比如String str2="java123$%^"，此时结果仍是-6。

  相同的时候返回0

* 有一个类似的`compareToIgnoreCase(String str)`，这个比较时忽略大小写

* `copyVaule(char[] data[, int offset, int count])`,后面两个可选，表示从offset开始，有count个

  ```java
  public class Test {
      public static void main(String args[]) {
          char[] Str1 = {'h', 'e', 'l', 'l', 'o', ' ', 'r', 'u', 'n', 'o', 'o', 'b'};
          String Str2 = "";
   
          Str2 = Str2.copyValueOf( Str1 );
          System.out.println("返回结果：" + Str2);
   
          Str2 = Str2.copyValueOf( Str1, 2, 6 );
          System.out.println("返回结果：" + Str2);
      }//如果Str原先有字符，则不会有影响，这是直接拷贝过来的
  }
  
  /*
  Output:
  返回结果：hello runoob
  返回结果：llo ru
  */
  ```

* `endsWith(String suffix):boolean` 是否以指定的后缀结束

* `startWith(String prefix):boolean` 是否以指定的前缀开始

* `equals(Object anObject):boolean`与指定的对象比较，**注意**不能使用`==`比较，它是比较**引用地址**是否一样的

* `hashCode():int`返回字符串的哈希值

* `indexOf(int ch/String str, [int fromIndex]):int`可以让其从指定的索引开始找

* `lastIndexOf(int ch, [int fromIndex]):int`:最后一次出现的位置

* `matches(String regex):boolean`，是否与正则表达式匹配

* `replace(char oldChar, char newChar):String`

* `replaceAll(Strung regex, String replacement):String`用replacement替换此字符串匹配给定的正则表达式的第一个子串

* `split(String regex[, int limit]):String`依靠正则表达式吃拆分，可以设置最多拆分几块

* `substring(int beginIndex[, int endIndex]):String`获得子串

* `toCharArray():char[]`将字符串转换为一个新的字符数组

* `toLowerCase():String`使用默认规则转换成小写

* `toUpperCase():String`使用默认规则转换成大写

* `toString():String`

* `trim():String`:忽略前导空白和尾部空白

* `contains(CharSequence chars):boolean`:是否包含

* `isEmpty():String`

格式化字符串和python 的一样。

 

# Java的API

----

其实也就是函数库，Java的API中，类是被包装在包中的

Java函数库的每个类都属于某个包，需要用完整的文件夹来表示，指明函数库类的完整名称，文件夹利用`.`的区分

**需要全名的原因：**

:crossed_fingers:可以制造出名称空间（如同C++的namespace），以便错开相同的类，并且包可以通过限制同一包之间的类相互存取以维护安全性。

