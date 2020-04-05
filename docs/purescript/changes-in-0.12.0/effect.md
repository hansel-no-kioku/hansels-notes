# PureScript 0.12.0 で何が変わったか - Effect編

!!! note "メモ"
    この記事は 2018/6/4 に [Qiita](https://qiita.com/) へ投稿したものです。

このシリーズでは 2018/5/22 にリリースされた [PureScript 0.12.0](https://github.com/purescript/purescript/releases/tag/v0.12.0) で何がどう変わったかを紹介していきます。

今回は新しく登場した `Effect` の話です。

なおこの記事は [PureScript](http://www.purescript.org/) に触れたことがある読者を想定しています。  
[PureScript](http://www.purescript.org/) 自体について知りたい場合には他の [Qiita の記事](https://qiita.com/tags/purescript)や [実例によるPureScript](https://aratama.github.io/purescript/index.html) などが参考になると思います。

## Effect とは何か?

[Effect](https://pursuit.purescript.org/packages/purescript-effect/2.0.0/docs/Effect) は外部への出力といった native な作用を表す型コンストラクタです。

こんな感じで使います。

```Haskell
hello ∷ Effect Unit
hello = do
  log "Hello World!"
```

## Eff と何が違うのか?

0.11.7 までの [PureScript](http://www.purescript.org/) では作用を記述するために [Eff](https://pursuit.purescript.org/packages/purescript-eff/3.2.1/docs/Control.Monad.Eff#t:Eff) を使いました。

`Eff` では下記のようにどのような種類の作用を含むのかを [row](https://github.com/purescript/documentation/blob/master/language/Types.md#rows) で記述します。

```Haskell
hello ∷ ∀ eff. Eff (console ∷ CONSOLE | eff) Unit
hello = do
  log "Hello World!"
```

`Eff` の定義は下記のとおりです[^1]。

```Haskell
foreign import data Eff :: # Effect -> Type -> Type
```

一方 `Effect` には具体的な作用を表す row がなく、下記のような定義になっています。

```Haskell
foreign import data Effect :: Type -> Type
```

今まで　`∀ eff. Eff (console ∷ CONSOLE | eff) Unit` や `∀ eff. Eff (canvas ∷ CANVAS, console ∷ CONSOLE, dom ∷ DOM, exception ∷ EXCEPTION, random ∷ RANDOM, ref ∷ REF | eff) Unit`[^2] などと書いていたものが、単に `Effect Unit` と書けるようになります。

なお `Effect` になってもコンパイル後の JavaScript のコードは `Eff` のときと変わりません。  
`Eff` と同等の最適化([magic do](https://github.com/purescript/documentation/blob/master/guides/Eff.md#the-eff-monad-is-magic))も行われます。

ちなみに `Effect` のサポート自体は破壊的な変更ではなく、今までの `Eff` もまだ使えます。  
`Effect` 以外の変更も多いため新旧のバージョンを混在させるとまずコンパイルが通らないのですが、purescript-prelude 含む各種ライブラリを PureScript 0.11.7 時代のバージョンで統一すれば `Eff` もちゃんと動きます[^3]。

## なぜ Eff が Effect になったのか?

`Eff` ではその関数が持つ作用の種類を示したり、制限することができます。

しかし一方で慣れないと判りにくいですし、作用の種類を漏れなく記述する必要があります。

結局メリットに比べて面倒なことが多い、と多くの人が考えるようになったようです。

ここに至るまでの出来事を、自分が確認できた範囲で記載しておきます。

### 2016/8/12

PureScript 版 IO として [purescript-io](https://github.com/slamdata/purescript-io) が登場しています。

このときの IO はそのままでは実行できず、IO → Aff → Eff と変換して使うようです。

### 2017/5/7

[purescript/purescript-eff](https://github.com/purescript/purescript-eff) に [Phil](https://github.com/paf31) さんから下記 PR があります。

* [Unrefined Eff type by paf31 · Pull Request #20](https://github.com/purescript/purescript-eff/pull/20)

row なしの `Eff a` を定義した `Control.Monad.Eff.Unrefined` を追加するもののようです。

この PR はマージされないまま翌月に close されています。

自分の英語力ではその理由を充分に読み取れませんが、core ライブラリに入れるなら最適化されるべき、とか、row なしの Eff が row ありの Eff に依存する形になっているのが逆であるべき、とかそんな感じのようです。

### 2017/7/4

[purescript/purescript-eff](https://github.com/purescript/purescript-eff) に下記 Issue が立ちました。

* [Use untagged `IO` as default, with tagged `Eff` to supplement. · Issue #25](https://github.com/purescript/purescript-eff/issues/25)

row なし `Eff` の方がシンプルで使いやすい、と考える人は多いようです。

### 2017/9/16

[Phil](https://github.com/paf31)さんから Twitter でアンケートです。

* [Proposal for PureScript 0.12.0: remove effect rows, making IO the default](https://twitter.com/paf31/status/908760073303764993)

結果は、賛成:50%、どちらとも言えない:36%、反対:14%、でした。

自分は当時このツイートを非公式アプリで読んだため、アンケートになっていたことに気付きませんでしたが。

### 2017/9/20

コンパイラ側の対応として [purescript/purescript](https://github.com/purescript/purescript) に下記 Issue が立ちました。

* [Switch from Eff to IO · Issue #3080](https://github.com/purescript/purescript/issues/3080)

ここでは `IO` と書かれていますが名称についてはその後も下記 Issue で議論されています。

* [Use untagged `IO` as default, with tagged `Eff` to supplement. · Issue #25](https://github.com/purescript/purescript-eff/issues/25)

`IO` や `Eff` のほか `Sync`、`Native` 等いろいろ出ていましたが、結局早い段階から [Phil](https://github.com/paf31) さんが推していた `Effect` になったようです。  

### 2018/5/22

`Effect` に対応した [PureScript 0.12.0](https://github.com/purescript/purescript/releases/tag/v0.12.0) がリリースされました。

[^1]: ここに出てくる `Effect` は kind であり、新しく追加された Effect とは別のものです。
[^2]: こんなに書いているコードが実際にあるのかは不明です。
[^3]: purescript-prelude と purescript-eff 程度なら問題ないですが、破壊的変更が多いため `Effect` とは関係なく動かなくなるライブラリも多そうです。
[^4]: [unsafeCoerceEff](https://pursuit.purescript.org/packages/purescript-eff/3.2.1/docs/Control.Monad.Eff.Unsafe#v:unsafeCoerceEff) 等逃げ道はいくらでもあると思いますが、ここでは触れません。
