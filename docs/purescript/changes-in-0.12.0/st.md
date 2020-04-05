# PureScript 0.12.0 で何が変わったか - ST編

!!! note "メモ"
    この記事は 2018/6/8 に [Qiita](https://qiita.com/) へ投稿したものです。

このシリーズでは 2018/5/22 にリリースされた [PureScript 0.12.0](https://github.com/purescript/purescript/releases/tag/v0.12.0) で何がどう変わったかを紹介していきます。

今回は `ST` の話です[^1]。

なおこの記事は [PureScript](http://www.purescript.org/) に触れたことがある読者を想定しています。  
[PureScript](http://www.purescript.org/) 自体について知りたい場合には他の [Qiita の記事](https://qiita.com/tags/purescript)や [実例によるPureScript](https://aratama.github.io/purescript/index.html) などが参考になると思います。

## Effect にならなかった ST

[前回の記事](https://qiita.com/hansel_no_kioku/items/c3a765f3a4da9f388eff)では新しく追加された [Effect](https://pursuit.purescript.org/packages/purescript-effect/2.0.0/docs/Effect#t:Effect) ついて書きました。

この `Effect` の追加に伴い、これまで [Eff](https://pursuit.purescript.org/packages/purescript-eff/3.2.1/docs/Control.Monad.Eff#t:Eff) を使用していた多くのライブラリが `Effect` を使うよう更新されています。

例えば [purescript-console](https://github.com/purescript/purescript-console) の [log](https://pursuit.purescript.org/packages/purescript-console/4.0.0/docs/Effect.Console#v:log) 関数は 3.0.0 から 4.0.0 のバージョンアップで下記のように変更されています[^2]。

```haskell
-- purescript-console-3.0.0
log :: forall eff. String -> Eff (console :: CONSOLE | eff) Unit
```

```haskell
-- purescript-console-4.0.0
log :: String -> Effect Unit
```

[CONSOLE](https://pursuit.purescript.org/packages/purescript-console/3.0.0/docs/Control.Monad.Eff.Console#t:CONSOLE) の他にも [RANDOM](https://pursuit.purescript.org/packages/purescript-random/3.0.0/docs/Control.Monad.Eff.Random#t:RANDOM)、[REF](https://pursuit.purescript.org/packages/purescript-refs/3.0.0/docs/Control.Monad.Eff.Ref#t:REF)、[EXCEPTION](https://pursuit.purescript.org/packages/purescript-exceptions/3.0.0/docs/Control.Monad.Eff.Exception#t:EXCEPTION) 等の作用がライブラリのバージョンアップにより `Effect` に置き換わっています。

例外が [ST](https://pursuit.purescript.org/packages/purescript-st/3.0.0/docs/Control.Monad.ST#t:ST) です。

`ST` はこうなりました。

```haskell
-- purescript-st-3.0.0

newSTRef :: forall a h r. a -> Eff (st :: ST h | r) (STRef h a)

readSTRef :: forall a h r. STRef h a -> Eff (st :: ST h | r) a

writeSTRef :: forall a h r. STRef h a -> a -> Eff (st :: ST h | r) a

modifySTRef :: forall a h r. STRef h a -> (a -> a) -> Eff (st :: ST h | r) a

runST :: forall a r. (forall h. Eff (st :: ST h | r) a) -> Eff r a

pureST :: forall a. (forall h. Eff (st :: ST h) a) -> a
```

```haskell
-- purescript-st-4.0.0

new :: forall a r. a -> ST r (STRef r a)

read :: forall a r. STRef r a -> ST r a

write :: forall a r. a -> STRef r a -> ST r a

modify :: forall r a. (a -> a) -> STRef r a -> ST r a

run :: forall a. (forall r. ST r a) -> a
```

関数名や引数の順番も変更されていますが、それはともかく `Eff` が `Effect` にはならず、かわりに新しく追加された型コンストラクタ `ST` を使うようになっています。

## それで何が変わったのか?

### 計算の実行方法が 1つになった

purescript-st 3.0.0 までの `ST` では `runST`、`pureST` 関数だけでなく、main関数等で返した作用(Eff)が処理系で実行されることでも計算を行うことが出来ました[^3]。

```haskell
-- 3.0.0 runSTによる計算例

onePlusOne ∷ ∀ eff. Eff eff Int
onePlusOne = runST do
  ref ← newSTRef 1
  modifySTRef ref (_ + 1)
```

```haskell
-- 3.0.0 pureSTによる計算例

onePlusOne ∷ Int
onePlusOne = pureST do
  ref ← newSTRef 1
  modifySTRef ref (_ + 1)
```

```haskell
-- 3.0.0 main関数による計算例

onePlusOne ∷ ∀ h eff. Eff (st ∷ ST h | eff) Int
onePlusOne = do
  ref ← newSTRef 1
  modifySTRef ref (_ + 1)

main ∷ ∀ h eff. Eff (console ∷ CONSOLE, st ∷ ST h | eff) Unit
main = do
  result ← onePlusOne
  logShow result
```

しかし purescript-st の 4.0.0 では `run` 関数でしか計算を実行出来なくなりました。

```haskell
-- 4.0.0 での計算例

onePlusOne ∷ Int
onePlusOne = run do
  ref ← new 1
  modify (_ + 1) ref
```

3.0.0 と 4.0.0 の例を比較すると、いくつかあった計算の実行方法のうち `pureST` だけが残って `run` 関数になったことが判ります。

### 他の作用と混在出来なくなった

3.0.0 では下の例のように他の作用と混在することが出来ました。

```haskell
-- 3.0.0 他の作用との混在例

-- 1～10 の間の乱数を 10回加算した結果を返す
sumRandom ∷ ∀ h eff. Eff (random ∷ RANDOM, st ∷ ST h | eff) Int
sumRandom = do
  ref ← newSTRef 0
  forE 0 10 \_ → do
    n ← randomInt 1 10
    void $ modifySTRef ref (_ + n)
  readSTRef ref
```

しかし 4.0.0 の `ST` は `Effect` と決別したため他の作用と混在出来なくなりました。

このようなときは `Ref` を使います。

```haskell
-- Ref を使った他の作用との混在例

import Effect.Ref (modify, new, read)

sumRandom ∷ Effect Int
sumRandom = do
  ref ← new 0
  forE 0 10 \_ → do
    n ← randomInt 1 10
    void $ modify (_ + n) ref
  read ref
```

また、コンパイル後の JavaScript のコードがちょっと大変なことになりますが、`StateT` を使うことも出来ます。

```haskell
-- StateT を使った作用との混在例

sumRandom ∷ Effect Int
sumRandom = 0 # execStateT do
  for_ (0 .. 9) \_ → do
    n ← lift $ randomInt 1 10
    modify (_ + n)
```

## コンパイラによる最適化は行われるのか?

PureScript のコンパイラは 3.0.0 までの ST を下記のように最適化します。

```haskell
-- 3.0.0 コンパイル前

culc ∷ Int → Int
culc n = pureST do
  ref ← newSTRef n
  _ ← modifySTRef ref (_ + 1)
  _ ← modifySTRef ref (_ * 2)
  modifySTRef ref (_ + 3)
```

```javascript
// 3.0.0 コンパイル後

var culc = function (n) {
    return Control_Monad_ST.pureST(function __do() {
        var v = Control_Monad_ST.newSTRef(n)();
        var v1 = Control_Monad_ST.modifySTRef(v)(function (v1) {
            return v1 + 1 | 0;
        })();
        var v2 = Control_Monad_ST.modifySTRef(v)(function (v2) {
            return v2 * 2 | 0;
        })();
        return Control_Monad_ST.modifySTRef(v)(function (v3) {
            return v3 + 3 | 0;
        })();
    });
};
```

ぱっと見どこが最適化されているか判りにくいですが、この例では `bind` が展開されています[^4]。

しかし 4.0.0 の `ST` ではこのような最適化は行われません。

```haskell
-- 4.0.0 コンパイル前

culc ∷ Int → Int
culc n = run do
  ref ← new n
  _ ← modify (_ + 1) ref
  _ ← modify (_ * 2) ref
  modify (_ + 3) ref
```

```javascript
// 4.0.0コンパイル後

var culc = function (n) {
    return Control_Monad_ST_Internal.run(Control_Bind.bind(Control_Monad_ST_Internal.bindST)(Control_Monad_ST_Internal["new"](n))(function (v) {
        return Control_Bind.bind(Control_Monad_ST_Internal.bindST)(Control_Monad_ST_Internal.modify(function (v1) {
            return v1 + 1 | 0;
        })(v))(function (v1) {
            return Control_Bind.bind(Control_Monad_ST_Internal.bindST)(Control_Monad_ST_Internal.modify(function (v2) {
                return v2 * 2 | 0;
            })(v))(function (v2) {
                return Control_Monad_ST_Internal.modify(function (v3) {
                    return v3 + 3 | 0;
                })(v);
            });
        });
    }));
};
```

これについては下記 Issue がありますのでいずれ最適化されるようになるかもしれません。

* [Optimizer changes for ST · Issue #3100](https://github.com/purescript/purescript/issues/3100)

## なぜ Effect にならなかったのか?

正直 [Issue](https://github.com/purescript/purescript-st/issues/8) でのやりとりを充分に理解出来ていませんが、今までの `ST` ですと `Ref` と性質や役割がかなり重なっていましたので、そこを明確に分けるために `Effect` にしなかった、と自分は理解しています。

実際 `pureST` を使用しているケースを除けば、従来の `ST` を使ったコードはほぼそのまま `Ref` で書くことが出来ます。  
そして `Ref` を使えば今までと同様の最適化も行われます。

このあたりは用途に応じて上手に使い分けていきたいところです。

[^1]: 厳密に言えば PureScript 本体ではなくライブラリの話になりますが、0.12.0 に伴う変更ということでこのシリーズで扱います。
[^2]: 2018/5/24 にはさらに 4.1.0 に更新され `MonadEffect` を使うようになっていますがここでは触れません。
[^3]: 厳密に言えば `runST` も `Eff` の row から `ST` を外すだけであり(`runST` の中身は [unsafeCoerce](https://pursuit.purescript.org/packages/purescript-unsafe-coerce/4.0.0/docs/Unsafe.Coerce#v:unsafeCoerce) と一緒)、計算は処理系での作用の実行時に行われます。
[^4]: ここでは 4.0.0 との比較のために `pureST` を使いましたが、`runST` を使えば `bind` だけでなくさらに `newSTRef` や `modifySTRef` も展開されます。
