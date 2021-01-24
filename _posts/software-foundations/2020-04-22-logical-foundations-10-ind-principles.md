---
layout: "post"
title: "「Logical Foundations」 10 Ind Principles (unfinished)"
subtitle: "Induction Principles"
author: "roife"
date: 2020-04-22

tags: ["B「Software Foundations」", "Benjamin C. Pierce", "B「Logical Foundations」", "L「Coq」", "PL", "unfinished"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
---

# Basics

Coq 会为 `Inductive` 类型定义一个归纳规则. 对于 `xxx`, 其归纳法则为 `xxx_ind`.

``` coq
Check nat_ind.
(* ==> *)
nat_ind
     : forall P : nat -> Prop,
       P 0 -> (forall n : nat, P n -> P (S n)) -> forall n : nat, P n
```

`induction` tactic 可以看作是 `apply xxx_ind` 的包装, 但是二者有细微区别:

- 使用归纳规则需要手动在 case 里进行 `intros`
- 使用归纳规则前一般不会 `intros` 归纳变量, 因为 `apply` 要求命题和结论完全相同, 如果 conclusion 中含有量词, 那么当前也要保留量词.

<!-- end list -->

``` coq
Theorem mult_0_r' : forall n:nat,
  n * 0 = 0.
Proof.
  apply nat_ind.
- (* n = O *) reflexivity.
- (* n = S n' *) simpl. intros n' IHn'. rewrite -> IHn'.
    reflexivity.  Qed.
```

对于 constructor `c t1 t2 ... a1 a2 ...` (其中 `ti` 为相应的类型, `ai` 为其它类型),
归纳规则产生的命题为:

``` coq
(forall a1 a2 ...) (t1 t2 ... : T),
P t1 -> P t2 ->
... ->
forall tn, P tn ->
P (c t1 t2 ... a1 a2 ...).
```

对于不是递归定义的类型, 可以用它证明某些命题对于所有情况都成立.

``` coq
t_ind : forall P : t -> Prop,
  ... case for c1 ... ->
  ... case for c2 ... -> ...
  ... case for cn ... ->
  forall n : t, P n
```

对于 polymorphic types, 效果类似的:

``` coq
list_ind :
  forall (X : Type) (P : list X -> Prop),
     P [] ->
     (forall (x : X) (l : list X), P l -> P (x :: l)) ->
     forall l : list X, P l
```

# Induction hypotheses

归纳假设即蕴涵式的前提部分, 使用时需要显式 `intros`.

``` coq
Definition P_m0r (n:nat) : Prop :=
  n * 0 = 0.

(* 等价表述 *)
Definition P_m0r' : nat->Prop :=
  fun n => n * 0 = 0.

Theorem mult_0_r'' : forall n:nat,
  P_m0r n.
Proof.
  apply nat_ind.
- (* n = O *) reflexivity.
- (* n = S n' *)
    intros n IHn. (* IHn 为归纳假设 *)
    unfold P_m0r in IHn. unfold P_m0r. simpl. apply IHn. Qed.
```

# More on the `induction` Tactic

注意到用 `xxx_ind` 之前不能进行 `intros`, 而 `induction` 会进行 `re-generalize`.
