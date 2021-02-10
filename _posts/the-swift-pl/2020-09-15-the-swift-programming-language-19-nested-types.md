---
layout: "post"
title: "「Swift」 19 Nested Types"
subtitle: "嵌套类型"
author: "roife"
date: 2020-09-15

tags: ["The Swift Programming Language@B", "Swift@L"]
lang: zh
catalog: true
header-image: ""
header-style: text
---

# Nested Types

```swift
struct BlackjackCard {

    // ...
    enum Suit: Character {
        // ...
    }

    // ...
    enum Rank: Int {
        // ...
    }
    // ...
}
```

# Referring to Nested Types

在外部使用嵌套类型时, 要在嵌套类型的类型名前加上其外部类型的类型名.

```swift
let heartsSymbol = BlackjackCard.Suit.hearts.rawValue
```
