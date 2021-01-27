---
layout: "post"
title: "「The Swift PL」 03 Strings and Characters"
subtitle: "字符串和字符"
author: "roife"
date: 2020-09-12

tags: ["B「The Swift Programming Language」", "L「Swift」"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
---

# String Literals

```swift
let someString = "Some string literal value"
```

## Multiline String Literals

```swift
let quotation = """
The White Rabbit put on his spectacles.
"Where shall I begin, please your Majesty?" he asked.

"Begin at the beginning," the King said gravely, "and go on
till you come to the end; then stop."
"""
```

`"""` 所在的行不会被包含进去.

用 `\` 可以防止换行.

```swift
let softWrappedQuotation = """
The White Rabbit put on his spectacles.  "Where shall I begin, \
please your Majesty?" he asked.

"Begin at the beginning," the King said gravely, "and go on \
till you come to the end; then stop."
"""
```

多行字符串中, 与结尾的 `"""` 的缩进对齐的空白会被忽略.

```swift
var multiString = """
    This is a Multiline String Literals
    """

// 输出 “This is a Multiline String Literals”
```

多行字符串中可以使用 `"` 和 `""`.

## Special Characters in String Literals

- The escaped special characters (`\n` 等)
- An arbitrary Unicode scalar value (`\u{n}` 等)

```swift
let dollarSign = "\u{24}"        // $,  Unicode scalar U+0024
let blackHeart = "\u{2665}"      // ♥,  Unicode scalar U+2665
```

## Extended String Delimiters

在引号前后加若干 `#` 可以打印 raw string.

```swift
let rawString =  #"Line 1\nLine 2"#

let threeMoreDoubleQuotationMarks = #"""
Here are three more double quotes: """
"""#
```

raw string 中仍然可以使用 escaped characters, 只要在 `\` 后加上相应数量的 `#`.

```swift
let escapeString =  ###"Line1\###nLine2"###
```

# Initializing an Empty String

```swift
var emptyString = ""               // empty string literal
var anotherEmptyString = String()  // initializer syntax

if emptyString.isEmpty {
    print("Nothing to see here")
}
```

# String Mutability

用 `var` 定义的字符串不可变, 用 `let` 定义的字符串可变.

# Strings Are Value Types

在赋值和传参时 String 会复制, 但是编译器会 Copy-on-Write 来降低开销.

# Working with Characters

```swift
for character in "Dog!🐶" {
    print(character)
}

let exclamationMark: Character = "!"
```

# Concatenating Strings and Characters

直接用 `+`, `+=`或者 `append()` method.

```swift
let string1 = "hello"
let string2 = " there"
var welcome = string1 + string2

var instruction = "look over"
instruction += string2

let exclamationMark: Character = "!"
welcome.append(exclamationMark)
```

在处理多行字符串的拼接时, 要注意衔接处是否要求有换行符.

```swift
let badStart = """
one
two
"""
let end = """
three
"""
print(badStart + end)
// Prints two lines:
// one
// twothree

let goodStart = """
one
two

"""
print(goodStart + end)
// Prints three lines:
// one
// two
// three
```

# String Interpolation

```swift
let multiplier = 3
let message = "\(multiplier) times 2.5 is \(Double(multiplier) * 2.5)"
```

在 Extended String 中, 只有在 `\` 后面加上了对应的 `#` 的字符串才会被计算.

```swift
print(#"Write an interpolated string in Swift using \(multiplier)."#)
// Prints "Write an interpolated string in Swift using \(multiplier)."

print(#"6 times 7 is \#(6 * 7)."#)
// Prints "6 times 7 is 42."
```

# Unicode

## Unicode Scalar Values

一个 Unicode 标量是一个 21-bit 数字.

## Extended Grapheme Clusters

Swift 中一个 Character 类型代表了一个 Extended Grapheme Clusters.

Unicode 之间可以组合, 如 `e` (`U+0065`) + `U+0301` = `é` (`U+00E9`). 此时 `é` 表示一个单一的 Unicode 字符.

```swift
let eAcute: Character = "\u{E9}"                         // é
let combinedEAcute: Character = "\u{65}\u{301}"          // e 后面加上  ́
```

## Counting Characters

Extended Grapheme Clusters 在 Swift 中算一个字符.

```swift
var word = "cafe"
print("the number of characters in \(word) is \(word.count)")
// “the number of characters in cafe is 4”

word += "\u{301}"    // 拼接一个重音，U+0301

print("the number of characters in \(word) is \(word.count)")
// “the number of characters in café is 4”
```

# Accessing and Modifying a String

## String Indices

由于 Unicode 字符中不同字符所占空间不一样, 因此不能用数字作为下标访问 Character, 必须用对应的 index.

- `s.startIndex`: 返回首索引
- `s.endIndex`: 获取尾后索引
- `s.index(before: )`: 上一个索引
- `s.index(after)`: 下一个索引
- `s.index(_:offsetBy:)`: 获取指定偏移的索引

```swift
let greeting = "Guten Tag!"
greeting[greeting.startIndex]
// G
greeting[greeting.index(before: greeting.endIndex)]
// !
greeting[greeting.index(after: greeting.startIndex)]
// u
let index = greeting.index(greeting.startIndex, offsetBy: 7)
greeting[index]
// a
```

用 `s.indices` 可以得到一个 ranges.

```swift
for index in greeting.indices {
   print("\(greeting[index]) ", terminator: "")
}
// 打印输出“G u t e n   T a g ! ”
```

## Inserting and Removing

- `s.insert(_:at:)`: 在指定索引处插入字符
- `s.insert(contentsOf:at:)`: 在指定处插入字符串
- `s.remove(at:)`: 在指定处删除一个字符
- `s.removeSubrange`: 在指定处删除一个子字符串

```swift
var welcome = "hello"
welcome.insert("!", at: welcome.endIndex)
// welcome 变量现在等于 "hello!"

welcome.insert(contentsOf:" there", at: welcome.index(before: welcome.endIndex))
// welcome 变量现在等于 "hello there!"

welcome.remove(at: welcome.index(before: welcome.endIndex))
// welcome 现在等于 "hello there"

let range = welcome.index(welcome.endIndex, offsetBy: -6)..<welcome.endIndex
welcome.removeSubrange(range)
// welcome 现在等于 "hello"
```

# Substrings

用下标或者 `prefix(_:)` 方法可以得到一个子字符串 (Substring), 类似于 Rust 中的 slice, 会重用原字符串的空间. 需要时应该涌动转换为 String 类型.

```swift
let greeting = "Hello, world!"
let index = greeting.firstIndex(of: ",") ?? greeting.endIndex
let beginning = greeting[..<index]
// beginning 的值为 "Hello"

// 把结果转化为 String 以便长期存储。
let newString = String(beginning)
```

# Comparing Strings

## String and Character Equality

两个 Extended Scalar Values 如果是 canonically equivalent 的, 那么他们会被认为是相等的. 比如 `\u{E9}` 等价于 `\u{65}\u{301}`.
但是看起来相同的 `U+0041` 与 `U+0410` 本质上是不同的.

- `s.hasPrefix(_:)` 是否拥有前缀
- `s.suffix(_:)` 是否拥有后缀

# Unicode Representations of Strings

通过 Unicode Representations 可以访问字符串在不同编码下的值.

- UTF-8: `s.utf8`
- UTF-16: `s.utf16`
- UTF-32 / 21-bit Unicode 标量集合: `s.unicodeScalars`

```swift
for codeUnit in dogString.utf8 {
    print("\(codeUnit) ", terminator: "")
}
print("")
// 68 111 103 226 128 188 240 159 144 182
```
