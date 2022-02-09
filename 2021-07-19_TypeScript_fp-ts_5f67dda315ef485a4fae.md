<!--
title:   fp-ts 代数型の型クラス
tags:    TypeScript,fp-ts,圏論
id:      5f67dda315ef485a4fae
private: false
-->
fp-tsは[static-land](https://github.com/fantasyland/static-land)で定義されたTypeScriptの代数型を参照して実装されている。
馴染みのない用語だらけなので、整理がてらに[仕様](https://github.com/fantasyland/static-land/blob/master/docs/spec.md)のまとめ。

![継承関係](https://github.com/fantasyland/fantasy-land/blob/master/figures/dependencies.png?raw=true)

| 代数型                            | 要約                                                      | 継承               |
|-----------------------------------|-----------------------------------------------------------|--------------------|
| [Setoid](#setoid)                 | 等値関係                                                  |                    |
| [Ord](#ord)                       | 大小関係                                                  | Setoid             |
| [Semigroup](#semigroup)           | 半群、二項演算                                            |                    |
| [Monoid](#monoid)                 | モノイド、二項演算 + 単位元                               | Semigroup          |
| [Group](#group)                   | 群、二項演算 + 単位元 + 逆元                              | Monoid             |
| [Semiroupid](#semiroupid)         | 半圏、恒等射のない圏                                      |                    |
| [Category](#category)             | 圏                                                        | Semigroupid        |
| [Functor](#functor)               | 関手                                                      |                    |
| [Bifunctor](#bifunctor)           | 双関手                                                    | Functor            |
| [Contravariant](#contravariant)   | 反変関手                                                  |                    |
| [Profunctor](#profunctor)         | 対角関手                                                  | Functor            |
| [Apply](#apply)                   | 評価射、積と冪からなる随伴の余単位                        | Functor            |
| [Applicative](#applicative)       | 強Laxモノイダル関手                                       | Apply              | 
| [Chain](#chain)                   | モナドの結合法則                                          | Apply              |
| [Monad](#monad)                   | モナド                                                    | Chain, Applicative |
| [Extend](#extend)                 | コモナドの結合法則                                        | Functor            |
| [Comonad](#comonad)               | コモナド                                                  | Extend             |
| [Alt](#alt)                       | 関手の（圏論的な意味でない）結合法則と分配法則            | Functor            |
| [Plus](#plus)                     | 関手の（圏論的な意味でない）結合法則と分配法則と単位元    | Alt                |
| [Alternative](#alternative)       |                                                           | Plus, Applicative  |
| [Filterable](#filterable)         | フィルタリング                                            |                    | 
| [ChainRec](#chainrec)             | 末尾再帰のChain                                           | Chain              |
| [Foldable](#foldable)             | catamorphism                                              |                    |
| [Traversable](#traversable)       | 計算効果の簡約化                                          | Functor, Foldable  |

「型クラス」とは言え、HaskellとかScalaとは違い、言語的サポートがないTypeScriptでは、単なるインタフェースとして定義されてる。
便利でもないし、扱いやすくもない。fp-tsではTypeScriptのわりと特殊な（だけども素晴らしい設計の）型システムを
最大限利用して高階カインド（らしきもの）をひねり出しているので、すこしは扱いやすくなっている。

## Setoid

等値関係が定義された集合。

```typescript
Setoid<T> {
  equals: (T, T) => boolean
}
```

**法則**

  1. Reflexivity: `S.equals(a, a) === true`
  1. Symmetry: `S.equals(a, b) === S.equals(b, a)`
  1. Transitivity: if `S.equals(a, b)` and `S.equals(b, c)`, then `S.equals(a, c)`


## Ord

大小関係が全ての要素に対して定義されている集合。

```typescript
Ord<T> {
  lte: (T, T) => boolean
}
```

**法則**

  1. Totality: `S.lte(a, b)` or `S.lte(b, a)`
  1. Antisymmetry: if `S.lte(a, b)` and `S.lte(b, a)`, then `S.equals(a, b)`
  1. Transitivity: if `S.lte(a, b)` and `S.lte(b, c)`, then `S.lte(a, c)`


## Semigroup

２項演算が定義されている集合

```typescript
Semigroup<T> {
  concat: (T, T) => T
}
```

**法則**

  1. 結合則: `S.concat(S.concat(a, b), c) ≡ S.concat(a, S.concat(b, c))`


## Monoid

２項演算に加えて、単位元が定義されている集合

```typescript
Monoid<T> {
  empty: () => T
}
```

**法則**

  1. 右単位元: `M.concat(a, M.empty()) ≡ a`
  1. 左単位元: `M.concat(M.empty(), a) ≡ a`


## Group

２項演算、単位元に加えて、逆元が定義されている集合

```typescript
Group<T> {
  invert: (T) => T
}
```

**法則**

  1. 右逆元: `G.concat(a, G.invert(a)) ≡ G.empty()`
  1. 左逆元: `G.concat(G.invert(a), a) ≡ G.empty()`


## Semigroupoid

恒等射がない圏（恒等射がない時点で圏の定義から外れるので、圏じゃないのだけれど...）。

```typescript
Semigroupoid<T> {
  compose: <i, j, k>(T<i, j>, T<j, k>) => T<i, k>
}
```

**法則**

  1. 結合則: `S.compose(S.compose(a, b), c) ≡ S.compose(a, S.compose(b, c))`


## Category

Semigroupoidに恒等射を加えて圏になる。

```typescript
Category<T> {
  id: <i, j>() => T<i, j>
}
```

**法則**

  1. 右単位元: `M.compose(a, M.id()) ≡ a`
  1. 左単位元: `M.compose(M.id(), a) ≡ a`


## Functor

```typescript
Functor<T> {
  map: <a, b>(a => b, T<a>) => T<b>
}
```

**法則**

  1. 単位元: `F.map(x => x, a) ≡ a`
  1. 合成: `F.map(x => f(g(x)), a) ≡ F.map(f, F.map(g, a))`


## Bifunctor

```typescript
Bifunctor<T> {
  bimap: <a, b, c, d>(a => b, c => d, T<a, c>) => T<b, d>
}
```

**法則**

  1. 単位元: `B.bimap(x => x, x => x, a) ≡ a`
  1. 合成: `B.bimap(x => f(g(x)), x => h(i(x)), a) ≡ B.bimap(f, h, B.bimap(g, i, a))`

**mapの導出**

  1. Functor's map: `A.map = (f, u) => A.bimap(x => x, f, u)`


## Contravariant

```typescript
Contravariant<T> {
  contramap: <a, b>(a => b, T<b>) => T<a>
}
```

**法則**

  1. 単位元: `F.contramap(x => x, a) ≡ a`
  1. 合成: `F.contramap(x => f(g(x)), a) ≡ F.contramap(g, F.contramap(f, a))`


## Profunctor

```typescript
Profunctor<T> {
  promap: <a, b, c, d>(a => b, c => d, T<b, c>) => T<a, d>
}
```

**法則**

  1. 単位元: `P.promap(x => x, x => x, a) ≡ a`
  1. 合成: `P.promap(x => f(g(x)), x => h(i(x)), a) ≡ P.promap(g, h, P.promap(f, i, a))`

**mapの導出**

  1. Functor's map: `A.map = (f, u) => A.promap(x => x, f, u)`

Haskellだと`promap`でなく`dimap`

```haskell
dimap :: (a -> b) -> (c -> d) -> p b c -> p a d
```


## Apply

評価射、積と冪からなる随伴の余単位。

```typescript
Apply<T> {
  ap: <a, b>(T<a => b>, T<a>) => T<b>
}
```

**法則**

  1. 合成: `A.ap(A.ap(A.map(f => g => x => f(g(x)), a), u), v) ≡ A.ap(a, A.ap(u, v))`


## Applicative

Applicative関手についてはこちらのサイト - [アプリカティブ関手ってなに？モノイド圏との関係は？調べてみました！](https://blog.miz-ar.info/2018/12/applicative-functor/) - が詳しくて日本語でお勧めです。
（中身を理解していないので伝聞調、信頼性なし）

圏論的には strong lax monoidal functor というらしい。laxは「〔規律などが〕緩い」という意味だそうで、無理くり訳すと「強くて緩いモノイダル関手」となって、訳さない、カタカナのママがよい用語の好例なのかもしれない。

[StackExchange: Explaining Applicative functor in categorical terms - monoidal functors](https://cstheory.stackexchange.com/questions/12412/explaining-applicative-functor-in-categorical-terms-monoidal-functors/12414)によると、カルテシアン閉圏のモノイダル関手が「強」であることと、自然変換`map:(A⇒B)→(F(A)⇒F(B))`を持つことは同値となる。モノイダル閉圏なので`(A⇒B)`も`(F(A)⇒F(B))`も内部Homとして存在して、それらの間に射があるというが「強」の条件となっている。

またlax monoidal functorは、モノイダル関手（モノイダル圏(C,⊗,I) から (D,⊕,J) への関手）に対して自然変換`i:J→F(I)`を条件としているらしいので、これが`of`のインタフェースになっているっぽい。

Wikipediaでカルテシアン閉圏を調べると[デカルト閉圏](https://ja.wikipedia.org/wiki/%E3%83%87%E3%82%AB%E3%83%AB%E3%83%88%E9%96%89%E5%9C%8F)が出てきて、「デカルト閉（英語: cartesian closed）な圏はラムダ計算の自然な設定ができるという点で数理論理学およびプログラミングの理論において特に重要である。デカルト閉圏の概念はモノイド圏に一般化される」んだそうで、プログラミングにおいてはこの辺りの性質がガバガバで利用できる、っぽい。

```typescript
Applicative<T> {
  of: <a>(a) => T<a>
}
```

**法則**

  1. Identity: `A.ap(A.of(x => x), v) ≡ v`
  1. Homomorphism: `A.ap(A.of(f), A.of(x)) ≡ A.of(f(x))`
  1. Interchange: `A.ap(u, A.of(y)) ≡ A.ap(A.of(f => f(y)), u)`

**mapの導出**

  1. Functor's map: `A.map = (f, u) => A.ap(A.of(f), u)`


## Chain

```js
Chain<T> {
  chain: <a, b>(a => T<b>, T<a>) => T<b>
}
```

**法則**

  1. 結合則: `M.chain(g, M.chain(f, u)) ≡ M.chain(x => M.chain(g, f(x)), u)`

**apの導出**

  1. Apply's ap: `A.ap = (uf, ux) => A.chain(f => A.map(f, ux), uf)`


## Monad

モナド。

**法則**

  1. 左単位元: `M.chain(f, M.of(a)) ≡ f(a)`
  1. 右単位元: `M.chain(M.of, u) ≡ u`

**mapの導出**

  1. Functor's map: `A.map = (f, u) => A.chain(x => A.of(f(x)), u)`


## Extend

[Chain](#chain)の双対。

```js
Extend<T> {
  extend: <a, b>(T<a> => b, T<a>) => T<b>
}
```

**法則**

  1. 結合則: `E.extend(f, E.extend(g, w)) ≡ E.extend(_w => f(E.extend(g, _w)), w)`


## Comonad

コモナド。

```js
Comonad<T> {
  extract: <a>(T<a>) => a
}
```

**法則**

  1. 左単位元: `C.extend(C.extract, w) ≡ w`
  1. 右単位元: `C.extract(C.extend(f, w)) ≡ f(w)`


## Alt

関手の並列的な合成。合成とは言え、圏論的な意味合いではなく、
`Alt`という名のごとく、同時に実行して、その結果からどちらかを
「選択」する、もしくは単純な「和」をとるようなことする場合に使う。

ある文字列を parse するのに、それが数字なのかキーワードなのか時間なのか、
それぞれの parser を `Alt` でまとめて最初にうまく行った parser の結果に
より選択する、とか、代数的というよりはプログラミング的な意味合い。

```js
Alt<T> {
  alt: <a>(T<a>, T<a>) => T<a>
}
```

**法則**

  1. 結合則: `A.alt(A.alt(a, b), c) ≡ A.alt(a, A.alt(b, c))`
  2. 分配則: `A.map(f, A.alt(a, b)) ≡ A.alt(A.map(f, a), A.map(f, b))`


## Plus

fp-tsではPlusの実装はなく、zeroはAlternativeで定義している。

```typescript
Plus<T> {
  zero: <a>() => T<a>
}
```

**法則**

  1. 右単位元: `P.alt(a, P.zero()) ≡ a`
  2. 左単位元: `P.alt(P.zero(), a) ≡ a`
  3. 零化: `P.map(f, P.zero()) ≡ P.zero()`


## Alternative

「和」である`alt`に対して、`ap`が「積」を期待されているようにみえる。

**法則**

  1. 分配則: `A.ap(A.alt(a, b), c) ≡ A.alt(A.ap(a, c), A.ap(b, c))`
  2. 零化: `A.ap(A.zero(), a) ≡ A.zero()`


## Filterable

```typescript
Filterable<T> {
  filter: <a>(a => boolean, T<a>) => T<a>
}
```

**法則**

  1. 分配則: `F.filter(x => f(x) && g(x), a) ≡ F.filter(g, F.filter(f, a))`
  1. 単位元: `F.filter(x => true, a) ≡ a`
  1. 零化: `F.filter(x => false, a) ≡ F.filter(x => false, b)`


## ChainRec

`Rec`はrecursion（再帰）の意味。Stack overflowを避けるのにtail recursion（末尾再帰）の実装をする（して欲しい）。

```typescript
ChainRec<T> {
  chainRec: <a, b>((a => Next<a>, b => Done<b>, a) => T<Next<a> | Done<b>>, a) => T<b>
}
```

**法則**

  1. 等価性: `C.chainRec((next, done, v) => p(v) ? C.map(done, d(v)) : C.map(next, n(v)), i) ≡ (function step(v) { return p(v) ? d(v) : C.chain(step, n(v)) }(i))`
  2. Stack usage of `C.chainRec(f, i)` must be at most a constant multiple of the stack usage of `f` itself.

```typescript
// fp-tsでの定義は違う

export interface ChainRec<F> extends Chain<F> {
  readonly chainRec: <A, B>(a: A, f: (a: A) => HKT<F, Either<A, B>>) => HKT<F, B>
}

// FがIdentityの場合のchainRecの実装は用意されている
// 
// よくあるEitherの使い方ではない。rightが正常値、leftがエラー値ではない。読み間違えないように

export const tailRec = <A, B>(startWith: A, f: (a: A) => Either<A, B>): B => {
  let ab = f(startWith)
  while (ab._tag === 'Left') {
    ab = f(ab.left)
  }
  return ab.right
}
```


## Foldable

```typescript
Foldable<T> {
  reduce: <a, b>((a, b) => a, a, T<b>) => a
}
```

- 関手Tに対するcatamorphisim（の一種）。第1引数と第2引数が[F代数](https://ja.wikipedia.org/wiki/F%E4%BB%A3%E6%95%B0)（を作るために必要なブツ）、キャリア型が`<a,b>`

**法則**

  1. `F.reduce ≡ (f, x, u) => F.reduce((acc, y) => acc.concat([y]), [], u).reduce(f, x)` 
     - 最初のreduceで、uを`[ ]`(配列)に移し替えた後、その配列に対して2番目のreduceを行っても結果は同じ。

**fp-tsインスタンス**

- Array
- Either
- Identity
- Map
- NonEmptyArray
- Option
- ReadonlyArray
- ReadonlyMap
- ReadonlyNonEmptyArray
- ReadonlyRecord
- ReadonlyTuple
- Record
- These
- Tree
- Tuple


## Traversable

`U`は何がしかの計算効果をもつものとして、その計算効果をTの各要素の計算に個別に（複数回）適用するのでなく、Tの各要素の計算全体に（１回だけ）適用する。

[#] 計算効果

```typescript
Traversable<T> {
  traverse: <U, a, b>(Applicative<U>, a => U<b>, T<a>) => U<T<b>>
}
```

**法則**

 1. Naturality: `f(T.traverse(A, x => x, u)) ≡ T.traverse(B, f, u)` for any `f` such that `B.map(g, f(a)) ≡ f(A.map(g, a))` 
     - `a`に対して、関数`f`（計算効果を伴う）を適用した後に`g`を適用することと、`g`を適用した後に関数`f`を適用することの結果は同じとみなせる。

 2. 単位元: `T.traverse(F, F.of, u) ≡ F.of(u)` for any Applicative `F`
 3. 合成: `T.traverse(Compose(A, B), x => x, u) ≡ A.map(v => T.traverse(B, x => x, v), T.traverse(A, x => x, u))` for `Compose` defined bellow and for any Applicatives `A` and `B`

```js
function Compose(A, B) {
  return {

    of(x) {
      return A.of(B.of(x))
    },

    ap(a1, a2) {
      return A.ap(A.map(b1 => b2 => B.ap(b1, b2), a1), a2)
    },

    map(f, a) {
      return A.map(b => B.map(f, b), a)
    },

  }
}
```

**reduceの導出**

```typescript
F.reduce = (f, acc, u) => {
  const of = () => acc
  const map = (_, x) => x
  const ap = f
  return F.traverse({of, map, ap}, x => x, u)
}
```

**mapの導出**

`js
F.map = (f, u) => {
  const of = (x) => x
  const map = (f, a) => f(a)
  const ap = (f, a) => f(a)
  return F.traverse({of, map, ap}, f, u)
}