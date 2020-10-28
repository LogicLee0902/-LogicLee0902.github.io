---
layout: "post"
title: "「Logical Foundations」 07 Ind Prop"
subtitle: "Inductively Defined Propositions"
author: "roife"
date: 2020-04-05

tags: ["B「Software Foundations」", "Benjamin C. Pierce", "B「Logical Foundations」", "L「Coq」", "PL", "笔记"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
---

# Inductively defined propositions

可以用几条 Rule 递归定义一个命题, 如定义 evenness:

- Rule `ev_0`: `0` 是偶数
- Rule `ev_SS`: 如果 `n` 是偶数, 那么 `S (S n)` 也是偶数.

之间的定义方法则是 `evenb n = true` 和 `exists k, n = double k`.

## Inference Rule & Proof Tree

Inference rules 是一种推理过程的写法, 横线上方写 premises, 横线下方写 conclusion, 横线右边写规则名称.
当横线上方没有 premises 时, 说明 conclusion 恒成立.

如定义 evenness:

\[\dfrac {} {even\ 0} \textbf {ev_0}\]
\[\dfrac {even\ n} {even\ (S\ (S\ n))} \textbf {ev_SS}\]

类似的, inference rules 也可以连续书写, 形成一棵 proof tree.

\[
\dfrac {} {
\dfrac {even\ 0} {
\dfrac {even\ 2} {
even\ 4
} \textbf {ev_SS}
} \textbf {ev_SS}
} \textbf {ev_0}
\]

## Inductive definition

``` coq
Inductive even : nat -> Prop :=
| ev_0 : even 0
| ev_SS (n : nat) (H : even n) : even (S (S n)).

(* 或者可以写成 Theorem 的形式 *)
Inductive even' : nat -> Prop :=
| ev_0 : even 0
| ev_SS : forall n, even n -> even (S (S n)).
```

- even 的类型为 `nat -> Prop`, 即 property of numbers. 其中 H 也被称为 `evidence`.
- `nat` 定义出现在 `:` 右侧, 称为 `index`; 而 Polymorphic list 中的 `X : Type` 出现在 `:` 左侧, 称为 `parameter`.
  - 对于 parameter 而言, `list X` 定义了一种类型, 它的所有 constructor 中的 X 都相同.
  - 对于 index 而言, `even nat` 定义了一类类型, 它的 constructor 的参数个数和类型都没有限制. 如 `ev_SS` 中 H 的类型可以是 `even 0` 或者 `even 2`.
- 可以将这种定义看做是一个伴随了两个 Theorem 的 property, constructor 名就是 Theorem 的名字, 可以像 Theorem 一样使用.

<!-- end list -->

``` coq
Theorem ev_4 : even 4.
Proof. apply ev_SS. apply ev_SS. apply ev_0. Qed.

(* 类似函数的构造写法 *)
Theorem ev_4' : even 4.
Proof. apply (ev_SS 2 (ev_SS 0 ev_0)). Qed.
```

# Evidence

## Destruct on evidence

可以直接对 evidence 进行 `destruct`. 类似于普通的 `Inductive` 类型, `destruct` 可以对
evidence 和 indices 的 constructor 分开讨论.

``` coq
Theorem ev_minus2 : forall n : nat,
  even n -> even (pred (pred n)).
Proof.
  intros n E.
  destruct E as [| n' E'] eqn:EQN.
- (* EQN = ev_0 *) simpl. apply ev_0.
- (* EQN = ev_SS n' E' *) simpl. apply E'.
Qed.
```

但是在一些证明中 `destruct` 会失效, 如:

``` coq
Theorem evSS_ev : forall n,
  even (S (S n)) -> even n.
Proof.
  intros n E.
  destruct E as [| n' E'] eqn:EQN.
- (* E = ev_0. *)
    (* 这里不能生成 assumptions, 因为 [destruct] 的作用是将 [S (S n)] 替换成另一个值, 但是要证明的结论是里没有出现 [S (S n)] *)
Abort.

(* 可以手动改写 hypotheses, 写出 assumptions explicitly *)
Theorem ev_inversion :
  forall (n : nat), even n ->
    (n = 0) \/ (exists n', n = S (S n') /\ even n').
Proof.
  intros n E.
  destruct E as [ | n' E'].
- (* E = ev_0 : even 0 *)
    left. reflexivity.
- (* E = ev_SS n' E' : even (S (S n')) *)
    right. exists n'. split. reflexivity. apply E'.
Qed.

Theorem evSS_ev : forall n, even (S (S n)) -> even n.
Proof. intros n H. apply ev_inversion in H. destruct H.
 - discriminate H.
 - destruct H as [n' [Hnm Hev]]. injection Hnm.
   intro Heq. rewrite Heq. apply Hev.
Qed.
```

## Inversion on evidence

- Inversion
  : 结合了 `destruct`, `discriminate`, `injection`, `intros`, `rewrite`, 是一个综合的 tactic, 可以看成一个特殊的 `destruct`.

<!-- end list -->

``` coq
Theorem evSS_ev' : forall n,
  even (S (S n)) -> even n.
Proof.
  intros n E.
  inversion E as [| n' E'].
  (* [E = ev_SS n' E'] *)
- apply E'.
Qed.
```

## Induction on evidence

``` coq
Lemma ev_even : forall n,
  even n -> exists k, n = double k.
Proof.
  intros n E.
  induction E as [|n' E' IH].
- (* E = ev_0 *)
    exists 0. reflexivity.
- (* E = ev_SS n' E'
       with IH : exists k', n' = double k' *)
    destruct IH as [k' Hk'].
    rewrite Hk'. exists (S k'). reflexivity.
Qed.
```

注意 `E'` 和 `IH` 的区别, `E'` 是对 `E` 的归纳, `IH` 是对 conclusion 的归纳.

对 evidence 进行归纳是很常用的技巧, 尤其在形式化程序语言中经常用到.

# Relations

`property` 只有一个参数, 可以看做定义了一个满足 `property` 的子集; `relation` 有两个参数,
可以看做定义了一个 `pair` 的子集.

``` coq
Inductive le : nat -> nat -> Prop :=
| le_n n : le n n
| le_S n m (H : le n m) : le n (S m).

Notation "m <= n" := (le m n). (* 注意和之前的 <=? 的区别 *)
```

# Case Study

## Regular Expressions

### Syntax

``` coq
Inductive reg_exp {T : Type} : Type :=
| EmptySet
| EmptyStr
| Char (t : T)
| App (r1 r2 : reg_exp)
| Union (r1 r2 : reg_exp)
| Star (r : reg_exp).
```

### Matching Rules

\[ \dfrac {} {[] =\sim {\rm EmptyStr}} \textbf{(MEmpty)} \]
\[ \dfrac {} {[x] =\sim {\rm Char}\ x} \textbf{(MChar)} \]
\[ \dfrac {s_1 =\sim re_1 \quad s_2 =\sim re_2} {s_1 ++ s_2 =\sim {\rm App}\ s_1 s_2} \textbf{(MApp)} \]
\[ \dfrac {s_1 =\sim re_1} {s_1 =\sim {\rm Union}\ re_1 re_2} \textbf{(MUnionL)} \]
\[ \dfrac {s_1 =\sim re_2} {s_1 =\sim {\rm Union}\ re_1 re_2} \textbf{(MUnionR)} \]
\[ \dfrac {} {[] =\sim {\rm Star}\ re} \textbf{(MStar0)} \]
\[ \dfrac {s_1 =\sim re_1 \quad s_2 =\sim {\rm Star}\ re} {s_1 ++ s_2 =\sim {\rm Star}\ re} \textbf{(MStarApp)} \]

``` coq
Inductive exp_match {T} : list T -> reg_exp -> Prop :=
| MEmpty : exp_match [] EmptyStr
| MChar x : exp_match [x] (Char x)
| MApp s1 re1 s2 re2
       (H1 : exp_match s1 re1)
       (H2 : exp_match s2 re2) :
    exp_match (s1 ++ s2) (App re1 re2)
| MUnionL s1 re1 re2
          (H1 : exp_match s1 re1) :
    exp_match s1 (Union re1 re2)
| MUnionR re1 s2 re2
          (H2 : exp_match s2 re2) :
    exp_match s2 (Union re1 re2)
| MStar0 re : exp_match [] (Star re)
| MStarApp s1 s2 re
           (H1 : exp_match s1 re)
           (H2 : exp_match s2 (Star re)) :
    exp_match (s1 ++ s2) (Star re).
```

### remember tactic

使用 `induction` 时可能会丢失一些信息, 导致证明无法继续进行. 比如 `s1 =~ Star re` 经过 `induction`
会得到 7 个 subgoals, 但是实际上只有 `MStar0` 和 `MStarApp` 满足条件. (因为把 `Star re`
看做一个整体)

- remember
  : 记忆某些信息防止在 `induction` 时丢失

<!-- end list -->

``` coq
Lemma star_app: forall T (s1 s2 : list T) (re : reg_exp),
    s1 =~ Star re ->
    s2 =~ Star re ->
    s1 ++ s2 =~ Star re.
Proof.
  intros T s1 s2 re H1.
  remember (Star re) as re'.
  generalize dependent s2.
  induction H1
    as [|x'|s1 re1 s2' re2 Hmatch1 IH1 Hmatch2 IH2
        |s1 re1 re2 Hmatch IH|re1 s2' re2 Hmatch IH
        |re''|s1 s2' re'' Hmatch1 IH1 Hmatch2 IH2].
- (* MEmpty *)  discriminate.
- (* MChar *)   discriminate.
- (* MApp *)    discriminate.
- (* MUnionL *) discriminate.
- (* MUnionR *) discriminate.
- (* MStar0 *)
    injection Heqre'. intros Heqre'' s H. apply H.
- (* MStarApp *) (* 此时会产生一个等价于 remember 产生的 hypotheses *)
    injection Heqre'. intros H0.
    intros s2 H1. rewrite <- app_assoc.
    apply MStarApp.
    + apply Hmatch1.
    + apply IH2.
      * rewrite H0. reflexivity.
      * apply H1.
Qed.
```

### Pumping Lemma

Pumping Lemma: 对于足够长的字符串 s 和一个匹配的正则表达式 re, 将其一个子串重复(pump) n 遍后,
得到的新字符串仍然能匹配 re.

``` coq
Fixpoint pumping_constant {T} (re : @reg_exp T) : nat := (* 定义"足够长"为字符串长度大于正则表达式的长度 *)
  match re with
  | EmptySet => 0
  | EmptyStr => 1
  | Char _ => 2
  | App re1 re2 =>
    pumping_constant re1 + pumping_constant re2
  | Union re1 re2 =>
    pumping_constant re1 + pumping_constant re2
  | Star _ => 1
  end.

Fixpoint napp {T} (n : nat) (l : list T) : list T := (* 定义一个重复字符串的函数和相关Lemma *)
  match n with
  | 0 => []
  | S n' => l ++ napp n' l
  end.

Lemma napp_plus: forall T (n m : nat) (l : list T),
    napp (n + m) l = napp n l ++ napp m l.
Proof.
  intros T n m l.
  induction n as [|n IHn].
- reflexivity.
- simpl. rewrite IHn, app_assoc. reflexivity.
Qed.
```

## Reflection

利用 reflection 可以将 `Prop` 和一个 `Bool` 绑定(一般是和与之对应的 `Bool` 绑定), 并利用两个
`Theorem` 进行轻松转化, 方便证明.

``` coq
Inductive reflect (P : Prop) : bool -> Prop :=
| ReflectT (H :   P) : reflect P true
| ReflectF (H : ~ P) : reflect P false.
```

``` coq
(* iff 与 reflect 互相转化 *)
Theorem iff_reflect : forall P b, (P <-> b = true) -> reflect P b.
Proof.
  (* WORKED IN CLASS *)
  intros P b H. destruct b.
- apply ReflectT. rewrite H. reflexivity.
- apply ReflectF. rewrite H. discriminate.
Qed.

Theorem reflect_iff : forall P b, reflect P b -> (P <-> b = true).
Proof.
  intros P b H. split.
- intros HP. destruct H.
    + reflexivity.
    + apply H in HP. destruct HP.
- intros Hb. rewrite Hb in H. inversion H. apply H0.
Qed.
```

``` coq
(* 一个使用例子 *)
Lemma eqbP : forall n m, reflect (n = m) (n =? m).
Proof.
  intros n m. apply iff_reflect. rewrite eqb_eq. reflexivity.
Qed.

Theorem filter_not_empty_In' : forall n l,
    filter (fun x => n =? x) l <> [] ->
    In n l.
Proof.
  intros n l. induction l as [|m l' IHl'].
- (* l = [] *)
    simpl. intros H. apply H. reflexivity.
- (* l = m :: l' *)
    simpl. destruct (eqbP n m) as [H | H]. (* 注意此处 destruct 直接将对应的命题转换为 [P] 和 [~P] 两种形式 *)
    + (* n = m *)
      intros _. rewrite H. left. reflexivity.
    + (* n <> m *)
      intros H'. right. apply IHl'. apply H'.
Qed.
```

`Reflect` 能大大缩短证明长度, 并且因 Coq 的 `SSReflect (small-scale reflection)`
库开始流行, 这个库证明了许多数学定理.

## Verified RE matcher

TODO
