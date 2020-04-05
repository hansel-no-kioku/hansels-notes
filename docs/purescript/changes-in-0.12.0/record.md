# PureScript 0.12.0 で何が変わったか - レコード編

!!! note "メモ"
    この記事は 2018/6/22 に [Qiita](https://qiita.com/) へ投稿したものです。

このシリーズでは 2018/5/22 にリリースされた [PureScript 0.12.0](https://github.com/purescript/purescript/releases/tag/v0.12.0) で何がどう変わったかを紹介していきます。  
PureScript 本体だけでなく関連ライブラリの変更についても紹介します。

今回はレコードの話です。

なおこの記事は [PureScript](http://www.purescript.org/) に触れたことがある読者を想定しています。  
[PureScript](http://www.purescript.org/) 自体について知りたい場合には他の [Qiita の記事](https://qiita.com/tags/purescript)や [実例によるPureScript](https://aratama.github.io/purescript/index.html) などが参考になると思います。

## レコードとは?

[レコード](https://github.com/purescript/documentation/blob/master/language/Records.md)はラベルと型の組み合わせからなる PureScript の組み込み型です。

```haskell
-- レコードの例

pochi ∷ { name ∷ String, age ∷ Int }
pochi = { name: "Pochi", age: 3 }
```

もう少し補足すると、[`Record`](https://pursuit.purescript.org/builtins/docs/Prim#t:Record) という型コンストラクタが用意されており、型の row[^1] を渡すとレコードの型が得られます。  
`{ name ∷ String, age ∷ Int }` は `Record ( name ∷ String, age ∷ Int )` の糖衣構文です。

[PureScript Language Reference](https://github.com/purescript/documentation/blob/master/language/README.md) の [7. Records](https://github.com/purescript/documentation/blob/master/language/Records.md) や[実例によるPureScript](https://aratama.github.io/purescript/)の[第3章 関数とレコード](https://aratama.github.io/purescript/chapter03.html)に詳しい説明があります。

## 何が変わったのか

PureScript 0.12.0 にあわせて更新された [purescript-prelude](https://github.com/purescript/purescript-prelude) の [4.0.0](https://github.com/purescript/purescript-prelude/releases/tag/v4.0.0) で、レコードが下記の型クラスのインスタンスになりました。

* [class Show](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.Show#t:Show)
* [class EQ](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.Eq#t:Eq)
* [class Semiring](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.Semiring#t:Semiring)
* [class Ring](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.Ring#t:Ring)
* [class CommutativeRing](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.CommutativeRing#t:CommutativeRing)
* [class HeytingAlgebra](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.HeytingAlgebra#t:HeytingAlgebra)
* [class BooleanAlgebra](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.BooleanAlgebra#t:BooleanAlgebra)
* [class Semigroup](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.Semigroup#t:Semigroup)
* [class Monoid](https://pursuit.purescript.org/packages/purescript-prelude/4.0.0/docs/Data.Monoid#t:Monoid)

## 何が出来るようになったのか?

今回の変更によりレコードに対し何が出来るようになったのかを型クラス別に紹介します。

### class Show

レコードを表示用の文字列に変換出来るようになりました。

```haskell
-- Show の例

main ∷ Effect Unit
main = do
  log $ show { name: "Pochi", age: 3 }
```

実行結果:

```console
> pulp run
{ age: 3, name: "Pochi" }
```

なおレコードの各フィールドの型がもれなく `class Show` のインスタンスになっている必要があります。  
このような制約はこれ以降に紹介する他の型クラスにも同様に存在します。

### class EQ

レコードの値が等しいかどうかを判定出来るようになりました。

```haskell
-- EQ の例

ret1 = { name: "Pochi", age: 3 } == { name: "Pochi", age: 3 } -- true
ret2 = { name: "Pochi", age: 3 } /= { name: "Mochi", age: 3 } -- true
ret3 = { name: "Pochi", age: 3 } == { name: "Pochi", age: 2 } -- false
```

すべてのフィールドの値が等しければ等しいとみなされます。

比較対象の型がきっちり一致していなければコンパイルエラーになります。  
これは以降に紹介する他の型クラスの二項演算子でも同様です。

~~なお `class Ord` のインスタンスではないため大小の比較は出来ません[^2]~~。

!!! note "2018/12/16追記"
    2018/7/7 にリリースされた purescript-prelude 4.01 で `class Ord` のインスタンスになったので、比較もできるようになりました。

### class Semiring, class Ring, class CommutativeRing

レコードの足し算、掛け算、引き算が出来るようになりました。

```haskell
-- Semiring, Ring の例

ret1 = { x: 4, y: 3 } + { x: 2, y: 1 } -- { x: 6, y: 4 }
ret2 = { x: 4, y: 3 } * { x: 2, y: 1 } -- { x: 8, y: 3 }
ret3 = { x: 4, y: 3 } - { x: 2, y: 1 } -- { x: 2, y: 2 }
```

各フィールド毎に演算が行われます。

`zero` や `one`、`negate` ももちろん使えます。

```haskell
-- Semiring, Ring の例2

ret1 = zero ∷ { x ∷ Int, y ∷ Int } -- { x: 0, y: 0 }
ret2 = one ∷ { x ∷ Int, y ∷ Int }  -- { x: 1, y: 1 }
ret3 = - { x: 1, y: 2}             -- { x: -1, y: -2 }
```

`class CommutativeRing` は乗算の[交換法則](https://ja.wikipedia.org/wiki/%E4%BA%A4%E6%8F%9B%E6%B3%95%E5%89%87)が成り立つことを保証するのみで、関数や演算子は定義されていません。

また、ここでは 3つの型クラスまとめて紹介していますが、フィールドの型が `class Semiring` のインスタンスではあっても `class Ring` のインスタンスではない場合、足し算や掛け算は出来ても引き算は出来ないレコードになります。  
これ以降も型クラス間に super - sub の関係がある場合はまとめて紹介しますが、どこまで出来るかはフィールドの型次第となります。

なお `class EuclideanRing` のインスタンスになっていないため割り算は出来ません[^3]。

### class HeytingAlgebra, class BooleanAlgebra

レコードの論理演算が出来るようになりました[^4]。

```haskell
-- HeytingAlgebra の例

ret1 = { a: true, b: false } && { a: true, b: true } -- { a: true, b: false }
ret2 = { a: true, b: false } || { a: true, b: true } -- { a: true, b: true }
ret3 = not { a: true, b: false }                     -- { a: false, b: true }
```

各フィールド毎に演算が行われます。

`class BooleanAlgebra` は[排中律](https://ja.wikipedia.org/wiki/%E6%8E%92%E4%B8%AD%E5%BE%8B)が成り立つことを保証するのみで、関数や演算子は定義されていません。

### class Semigroup, class Monoid

レコードの連結が出来るようになりました[^5]。

```haskell
-- Semigroup, Monoid の例

ret1 = { a: [ 1 ], b: "Foo" } <> { a: [ 2 ], b: "Bar" } -- { a: [1,2], b: "FooBar" }
ret2 = mempty ∷ { a ∷ Array Int, b ∷ String }           -- { a: [], b: "" }
```

各フィールド毎に連結されます。
`mempty` も見たままです。

## どうやって実現しているのか?

すべて [PureScript 0.11.6](https://github.com/purescript/purescript/releases/tag/v0.11.6) で登場した `RowToList` を使って実現しています。

ここでは `class EQ` を例に説明します。

### class EQ のインスタンス定義

レコードに対する `class EQ` のインスタンス定義は下記のとおりです。

```haskell
-- EQ のインスタンス定義

instance eqRec :: (RL.RowToList row list, EqRecord list row) => Eq (Record row) where
  eq = eqRecord (RLProxy :: RLProxy list)
```

早速 `RowToList` が出て来ました。  
`RowToList` については [PureScript: RowToList](https://liamgoodacre.github.io/purescript/rows/records/2017/07/10/purescript-row-to-list.html) に説明がある他、ここ [Qiita](https://qiita.com/) でも [Justin Woo](https://qiita.com/kimagure) さんがたくさん記事を書かれています。
ただしいずれも英語です。  
この記事を書いている時点で日本語の記事は [Justin Woo](https://qiita.com/kimagure) さんの記事を [@oreshinya](https://qiita.com/oreshinya) さんが翻訳した [PureScriptで簡単にJSONをパースする - Qiita](https://qiita.com/oreshinya/items/6bb8ef2639f2f739b7e7) くらいしか見つかりませんでした。

しかしここで説明しきれる気がしないためとりあえず先に進みます。

レコード比較の本体は `class EqRecord` とその関数 `eqRecord` にあります。

### class EqRecord と eqRecord

`class EqRecord` と `eqRecord` の定義はこのようになっています。

```haskell
-- EqRecord とインスタンスの定義

class EqRecord rowlist row where
  eqRecord :: RLProxy rowlist -> Record row -> Record row -> Boolean

instance eqRowNil :: EqRecord RL.Nil row where
  eqRecord _ _ _ = true

instance eqRowCons
    :: ( EqRecord rowlistTail row
       , Row.Cons key focus rowTail row
       , IsSymbol key
       , Eq focus
       )
    => EqRecord (RL.Cons key focus rowlistTail) row where
  eqRecord _ ra rb = (get ra == get rb) && tail
    where
      key = reflectSymbol (SProxy :: SProxy key)
      get = unsafeGet key :: Record row -> focus
      tail = eqRecord (RLProxy :: RLProxy rowlistTail) ra rb
```

`eqRowNil` は中身が空のレコード、つまり `{}` 同士の比較にマッチします。  
この場合関数 `eqRecord` は常に `true` を返します。

`eqRowCons` の方は空ではないレコード同士の比較にマッチします。  
`tail` の定義にも `eqRecord` が登場していることから判るようにレコードの内容を1つづつ再帰的に比較しています。  
関数 `eqRecord` をコールするたびに `rowlistTail` の中のフィールドが1つ減り、空になると `eqRowNil` の方にマッチして再帰呼び出しが終了する仕組みです[^6]。

このように再帰的に処理出来るのは、実は `RowToList` のおかげです。  
row には「順序」がないので「先頭」と「残り」に分割できません。  
`RowToList` によって row を `RowList` という「順序」[^7]を持った型レベルのリストに変換することで、`Cons` で「先頭」と「残り」のリストに分割して再帰的に処理できるようになります。

[^1]: row の定訳が「行」なのか「列」なのかその他なのか判りませんでした。自分は普段そのまま row(ロウ) と呼んでいるので今回はとりあえずこのままにします。いずれ用語も整理したいところです。
[^2]: 一応ソース上にコードが書かれているのですが、ラベルのアルファベット順に比較する実装でよいのか疑問があるようで、コメントアウトされています。
[^3]: [Record instances of Show, Eq, Ord etc · Issue #154](https://github.com/purescript/purescript-prelude/issues/154) によると[整域](https://ja.wikipedia.org/wiki/%E6%95%B4%E5%9F%9F)の定義「任意の非零元の積は非零である」を満たさないからのようです。そして[ユークリッド環](https://ja.wikipedia.org/wiki/%E3%83%A6%E3%83%BC%E3%82%AF%E3%83%AA%E3%83%83%E3%83%89%E7%92%B0)(Euclidean ring)は整域の真部分集合なので整域でなければユークリッド環じゃない、ということらしいです。割り算出来ればいいじゃん、とはいかない厳しい世界のようです。
[^4]: Prelude から再export されていないこともあり記載を省略しましたが `ff` や `tt`、`implies` も使えます。
[^5]: Prelude から再export されていないこともあり記載を省略しましたが `power` や `guard` も使えます。
[^6]: フィールドが不一致ならそこですぐに比較を終了して欲しいところですが、実際にはすべてのフィールドが比較されるようです。`{a: 1, b: 2, c: 3} == {a: 2, b: 2, c: 3}` が `1 == 2 && (2 == 2 && (3 == 3 && true)` に展開され、かつ右から評価されるイメージです。
[^7]: `RowList` ではラベルのアルファベット順になります。

