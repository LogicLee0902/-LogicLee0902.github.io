---
layout: "post"
title: "「TSS」 01 Welcome Back to the Show"
subtitle: ""
author: "roife"
date: 2021-02-24

tags: ["The Seasoned Schemer@Books@Series", "Scheme@Languages@Tags", "程序语言理论@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
---

- `(two-in-a-raw? lat)` 判断 `lat` 中是否有两个连续的元素

```scheme
(define is-first?
  (lambda (a lat)
    (cond
     ((null? lat) #f)
     (else (eq? (car lat) a)))))

(define two-in-a-row
  (lambda (lat)
    (cond
     ((null? lat) #f)
     (or (is-first (car lat) (cdr lat))
         (two-in-a-row (cdr lat))))))
```

- `(sum-of-prefixes tup)` 计算前缀和列表，如 `(sum-of-prefixes '(1 1 1)) => '(1 2 3)`

```scheme
(define (sum-of-prefixes-b)
  (lambda (sonssf tup)
    (cond
     ((null? tup) '())
     (else (cons (o+ sonssf (car tup))
                 (sum-of-prefixes
                  (o+ sonssf (car tup))
                  (cdr tup)))))))

(define sum-of-prefixes
  (lambda (tup)
    (sum-of-prefixes-b 0 tup)))
```

> The Eleventh Commandment
>
> Use additional arguments when a function needs to know what other arguments to the function have been like so far


