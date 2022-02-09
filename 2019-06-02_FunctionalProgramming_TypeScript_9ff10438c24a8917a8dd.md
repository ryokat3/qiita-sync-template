<!--
title:   TypeScriptの（tuple値じゃなくて）tuple型を操作する
tags:    FunctionalProgramming,TypeScript
id:      9ff10438c24a8917a8dd
private: false
-->
TypeScriptのtuple型（「値」 `[4, "hello", true]`じゃなくて「型」`[number, string, boolean]`）を操作する。

もちろん型を計算しているだけなので、runtime時の負荷は0。
snippetを上からコピーしてVSCodeに貼り付けていけばOK、型計算だけなのでコンパイル不要です。

## Length (tuple型の要素数を取得）

`[string, number, boolean]`から要素数の`3`を得る

```ts
type Length<T extends any[]> = T['length']

type Target1 = Length<[string, number, boolean]> // => 3
```


## Push （tuple型の先頭に型を追加）

`[string, number]`の先頭に`boolean`を追加して、`[boolean, string, number]`を作る。

```ts
declare const None: unique symbol
type None = typeof None

// Pushする型が'None'の場合は何もしない

type Push<A, T extends Array<any>> = {
    nop: T
    push: ((a: A, ...b: T) => void) extends ((...a: infer I) => void) ? I : []
}[ A extends None ? "nop" : "push" ]

type Target2 = Push<boolean, [string, number]>  // => [boolean, string, number]
```

## Pop （tuple型の先頭の型を削除）

`[boolean, string, number]`の先頭を削除して`[string, number]`を作る。

```ts
type Pop<T extends any[]> = Length<T> extends 0 ? [] : (((...b: T) => void) extends (a:any, ...b: infer I) => void ? I : [])

type Target3 = Pop<[boolean, string, number]>  // => [string, number]
```

## Head （tuple型の先頭の型の取得）

`[boolean, string, number]`の先頭の型`boolean`を取得する。

```ts
type Head<T extends any[]> = Length<T> extends 0 ? None : T[0]

type Target4 = Head<[boolean, string, number]>  // => boolean
```

## Reverse（tuple型の順番をひっくり返す）

`[boolean, string, number]`から`[number, string, boolean]`を得る。

```ts
type Reverse<Items extends any[], Result extends Array<any> = []> = {
    done: Result
    // @ts-ignore
    continue: Reverse<Pop<Items>, Push<Head<Items>, Result>>
}[ Length<Items> extends 0  ? "done" : "continue"]

type Target5 = Reverse<[boolean, string, number]>  // => [number, string, boolean]
```
再帰呼出しのネストが深すぎてTypeScriptのコンパイラがエラーを出すので`// @ts-ignore`で抑制する。業が深い。

_# 色々手に負えなくなり、後述のnpmパッケージのboost-tsのReverseの実装では再帰呼び出し方式はやめました..._

# 型安全な関数

TypeScriptはもちろん型安全なんだけれど、関数を変形させたり、引っ張りまわしたりするうちに、
あれ、なんか型が変だぞ、俺の型どこ行った、みたいなことがままある。で、"as any"を御託宣の如く
ありがたく使い始めて破綻する、ということは避けたい。で、tuple型をいじくりまわして、便利関数を
作ってみた。


## partial （関数の部分呼出）

tuple型の操作ができると、[boost](https://www.boost.org/)みたいな`_1`とか`_2`を使った関数の部分呼出を__型安全に__実装することができた。TypeScriptの型、いいっす。

```ts
import { partial, _1, _2 } from "boost-ts/lib/funclib"

function sub (a:number, b:number):number {
    return a - b
}

// 2番目のargumentをバインドする。
const sub10 = partial(sub, _1, 10)        // sub10の型は (a:number)=>number
console.log(sub10(100))                   // 90と表示する

// 1番目と2番目のargumentの順番を変える
const reverse_sub = partial(sub, _2, _1)  // reverse_subの型は (a:number, b:number)=>number
console.log(reverse_sub(10, 100))         // 90と表示する
```

自分的には割とやりたかったことが実装できて満足。

## mkmapobj （オブジェクトのプロパティの型を変換する）

TypeScriptの Key-Value のオブジェクトのKeyは変えずに、Valueだけ変えたい、関数をつかっていっぺんに変えたい、って無茶苦茶普通のことだけど、何故か標準のライブラリに見当たらない。自作は簡単だけど、変えた後のオブジェクトのValueの型が、関数の型に引きずられてぼんやりしちゃう。具体的には、

```ts
// こんな感じのデータの値をを       
const data = {
    name: "John",
    age: 26
}

// こんな型に入れたいので、
type Box<T> = { value: T }

// こんな関数を用意して、
function boxify<T>(t: T):Box<T> {
    return { value: t }
}

// こんな感じで変換してみた！
const unexpected = Object.entries(data).reduce((acc, [key, value])=>{
    return {
        ...acc,
        [key]: boxify(value)    
    }    
}, {})

// だけど unexpected.name って、エラーになっちゃうし、
//
// 頑張ってやってはみたけど、こんな感じの型にしかならなかったよ...
// {
//     name: Box<number> | Box<string>
//     age: Box<number> | Box<string>
// }
```

と、こういうときにはめんどくさいけど、typeの Key-Value型のリストを作って頑張る。具体的には

```ts
type assocList = [
  [ number, Box<number> ],　// Key = number, Value = Box<number>
  [ string, Box<string> ]   // Key = string, Value = Box<string>
]
```

で、この`assocList`を使って型変換をおこなってやれば多分うまくいく。
でも毎回毎回こんな型つくってられないのですが、Mapped Tuple Typeをつかえば多分うまくいく。
で、ライブラリにして公開したら、割と良い感じになってきた。

```ts
// Mapped Tuple Type向けのBoxの型を用意
type BoxMapType<T> = { [P in keyof T]: [T[P], Box<T[P]>] }

// Keyとなる型を列挙
type BoxKeyType = [string, number, boolean, string[], number[]]

// BoxMapType<BoxKeyType>でさっきのassocListができるので、とりあえず型をmkmapobjで型を閉じ込めて、
const mapobj = mkmapobj<BoxMapType<BoxKeyType>>()

// 変換すると、こんな型に変換されている！！
// {
//    name: Box<string>,
//    age: Box<number>
// }
const dataBox = mapobj(data, boxify)
```

## 参照
ソースコードをまとめてNPMパッケージ [boost-ts](https://www.npmjs.com/package/boost-ts) として公開しました。`npm install boost-ts`でご利用ください。