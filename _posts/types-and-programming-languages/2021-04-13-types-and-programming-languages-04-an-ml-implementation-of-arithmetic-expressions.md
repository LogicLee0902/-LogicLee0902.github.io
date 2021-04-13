---
layout: "post"
title: "「TAPL」 04 An ML Implementation of Arithmetic Expressions"
subtitle: "用 ML 实现 AE"
author: "roife"
date: 2021-04-13

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "类型系统@Tags@Tags", "OCaml@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# Syntax

```ocaml
type term =
  TmTrue of info
| TmFalse of info
| TmIf of info * term * term * term
| TmZero of info
| TmSucc of info * term
| TmPred of info * term
| TmIsZero of info * term
```

- 检验一个 `term` 是否是数字：

    ```ocaml
    let rec isnumericval t = match t with
    TmZero(_) → true
    | TmSucc(_, t1) → isnumericval t1
    | _ → false
    ```

- 检验 `term` 的合法性：

    ```ocaml
    let rec isval t = match t with
    TmTrue(_) → true
    | TmFalse(_) → true
    | t when isnumericval t → true
    | _ → false
    ```

# Evaluation

计算函数是一个 partial function。如果传入的 `term` 非法，则返回 the next step of evaluation。

```ocaml
exception NoRuleApplies

let rec eval1 t = match t with
  TmIf(_,TmTrue(_),t2,t3) →
    t2
| TmIf(_,TmFalse(_),t2,t3) →
    t3
| TmIf(fi,t1,t2,t3) →
    let t1’ = eval1 t1 in
        TmIf(fi, t1’, t2, t3)
| TmSucc(fi,t1) →
    let t1’ = eval1 t1 in
        TmSucc(fi, t1’)
| TmPred(_,TmZero(_)) →
    TmZero(dummyinfo)
| TmPred(_,TmSucc(_,nv1)) when (isnumericval nv1) →
    nv1
| TmPred(fi,t1) →
    let t1’ = eval1 t1 in
        TmPred(fi, t1’)
| TmIsZero(_,TmZero(_)) →
    TmTrue(dummyinfo)
| TmIsZero(_,TmSucc(_,nv1)) when (isnumericval nv1) →
    TmFalse(dummyinfo)
| TmIsZero(fi,t1) →
    let t1’ = eval1 t1 in
        TmIsZero(fi, t1’)
| _→
    raise NoRuleApplies
```