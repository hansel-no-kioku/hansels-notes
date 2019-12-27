# Stateモナド

Effect や Maybe はなんとなく判るけど Stateモナドいまいち判らない、とか、モナド変換子さっぱり判らない、とか言っていた2017年の自分のために書いてみました。  

まずは Stateモナドの話から。

## いきなり結論

Stateモナドに**「状態」**は入ってないです。  
中身はただの**「関数」**です。  
副作用なしで状態を遷移させちゃうよ**モナドの魔法**で! とか勘違いですよー。

そして怖がらずに実装読みましょう。  
そんなに難しくないです。

…と、だいたいこれで全部ですが、昔の自分がこれで判るかどうか不安なので長々と書いてみます。

## ただ遷移するだけ

Int 型の状態を変更したり上書きする関数を複数用意してつなげてみます。

```haskell
transit :: Int -> Int
transit
    = (_ + 1)    -- 前の状態に +1 した新しい状態を返す関数
  >>> (_ + 2)    -- 前の状態に(以下略)
  >>> (\_ -> 3)  -- 前の状態に関係なく新しい状態を 3 にする関数
  >>> (_ + 4)    -- 前(以下略)
```

この関数に初期状態、例えば 3 を渡せば 3 → 4 → 6 → 3 と状態が遷移して最後は 7 で終わります。

簡単です。

[こちら](https://try.purescript.han-sel.com/?gist=ffb9f03650cd58cd0e70f2fd69134e36)で実際に試すことができます。

## 途中の状態になにかしたい

人は皆 `even :: Int -> Boolean` で途中の状態が偶数かどうか調べたくなったりします。  
さらに調べた結果や途中の状態自体を `doSomething :: forall a. Show a => a -> Unit` のような関数に渡したくなったりします。

でも先程の一連の関数たちの途中に `even` や `doSomething` を割り込ませると、状態を次の関数に引き継げなくなりますし、そもそも型が合わなくなってコンパイルも通りません。

[実際に試す](https://try.purescript.han-sel.com/?gist=7388980e85b9da492db4d06d050fb9fc)

では、なにかした結果と状態の両方を引き継げるように関数を改良してみましょう。  
例えばそれぞれの関数の型を `Int -> Int` から `Tuple a Int -> Tuple b Int` に変えてみます。  
ここで `a` はなにかした結果の型です。  
なにもしていないときは `Unit` にするルールにします。

```Haskell
transit :: forall a. Tuple a Int -> Tuple Unit Int
transit
    = (\(Tuple _ s) -> Tuple unit            (s + 1))  -- 状態を +1 する
  >>> (\(Tuple _ s) -> Tuple (even s)         s     )  -- 現在の状態が偶数か調べる
  >>> (\(Tuple b s) -> Tuple (doSomething b)  s     )  -- 調べた結果でなにかする
  >>> (\(Tuple _ s) -> Tuple unit            (s + 2))  -- 状態を(以下略)
  >>> (\(Tuple _ s) -> Tuple (doSomething s)  s     )  -- 現在の状態でなにかする
  >>> (\(Tuple _ _) -> Tuple unit                 3 )  -- 状態を 3 にする
  >>> (\(Tuple _ s) -> Tuple unit            (s + 4))  -- (以下略)
```

[実際に試す](https://try.purescript.han-sel.com/?gist=9a8c0100f5dfbd388ce897e36b5eb1e2)

## かっこよくする

でもなんだか同じようなコードが何度も出てきたり、やたらとタプタプしてかっこ悪いです。  
なので意図が伝わりやすいよう名前をつけてかっこよくしてみましょう。

```haskell
type State s a = s -> Tuple a s         -- 別名を付ける

bind m f = m >>> \(Tuple a s) -> f a s  -- 出力と状態を次の関数に渡す
infixl 1 bind as >>-                    -- 簡潔に書くために演算子を定義

modify f = \s -> Tuple unit  (f s)      -- 状態を変更する
gets   f = \s -> Tuple (f s) s          -- 現在の状態でなにかする
get      = \s -> Tuple s     s          -- 現在の状態を得る
put    s = \_ -> Tuple unit  s          -- 状態を設定する
lift   m = \s -> Tuple m     s          -- なにかする

execState m = m >>> snd                 -- 初期状態を渡して最後の状態を得る
```

これらを使うとこんな感じにかっこよくなります。

```haskell
transit :: State Int Unit
transit   = modify (_ + 1)          -- 状態を +1 する
  >>- \_ -> gets   even             -- 現在の状態が偶数か調べる
  >>- \b -> lift   (doSomething b)  -- 調べた結果でなにかする
  >>- \_ -> modify (_ + 2)          -- 状態を(以下略)
  >>- \_ -> get                     -- 状態を取得する
  >>- \s -> lift   (doSomething s)  -- 現在の状態でなにかする
  >>- \_ -> put    3                -- 状態を 3 にする
  >>- \_ -> modify (_ + 4)          -- (以下略)
```

[実際に試す](https://try.purescript.han-sel.com/?gist=761235a95b0354e8ba81dc43f4164cfc)

## do記法で書きたい

実は `bind` さえあればdo記法でもかけちゃいます。

```haskell
transit :: State Int Unit
transit = do
  _ <- modify (_ + 1)
  b <- gets even
  _ <- lift $ doSomething b
  _ <- modify (_ + 2)
  s <- get
  _ <- lift $ doSomething s
  _ <- put 3
  modify (_ + 4)
```

さらに `discard ＝ bind` しておけば `_ <-` も省略できます。

```haskell
transit :: State Int Unit
transit = do
  modify (_ + 1)
  b <- gets even
  lift $ doSomething b
  modify (_ + 2)
  s <- get
  lift $ doSomething s
  put 3
  modify (_ + 4)
```

[実際に試す](https://try.purescript.han-sel.com/?gist=47df53fcd108470a573f6f6ee740c696)

## Monad のインスタンスにする

`bind` さえあればdo記法使えます、とはいってもさすがに横着が過ぎる気がしますので、ちゃんと `Monad` のインスタンスにしてみましょう。

そのためにまずはただの別名ではなく新しい型として定義します。

```haskell
newtype State s a = State (s -> Tuple a s)
```

そしてインスタンスを定義します。

```haskell
instance monadState :: Monad (State s)
```

すると…

```
  No type class instance was found for

    Control.Applicative.Applicative (State s0)
```

怒られましたね。

`Applicative` のインスタンスにする必要があるようです。  
仕方がないので追加します。

```haskell
instance applicativeState :: Applicative (State s)
```

すると…

```
  The following type class members have not been implemented:
  pure :: forall a. a -> State s a
```

今度は `pure` 関数がないと怒られました。

仕方がないので追加しましょう  
…という感じでエラーが出なくなるまで続けます。

最終的にこんな感じになります。

```haskell
-- 新しい型にする
newtype State s a = State (s -> Tuple a s)

-- Monad のインスタンスにする
instance functorState :: Functor (State s) where
  map f (State a) = State \s -> case a s of Tuple b s' -> Tuple (f b) s'

instance applicativeState :: Applicative (State s) where
  pure a = State (\s -> Tuple a s)

instance applyState :: Apply (State s) where
  apply = ap

instance bindState :: Bind (State s) where
  bind (State x) f = State \s ->
    case x s of Tuple v s' -> case f v of State f' -> f' s'

instance monadState :: Monad (State s)

-- いろいろな関数を用意する
modify f = State \s -> Tuple unit  (f s)  -- 状態を変更する
gets   f = State \s -> Tuple (f s) s      -- 現在の状態でなにかする
get      = State \s -> Tuple s     s      -- 現在の状態を得る
put    s = State \_ -> Tuple unit  s      -- 状態を設定する
lift   m = State \s -> Tuple m     s      -- なにかする

execState (State m) = m >>> snd           -- 初期状態を渡して最後の状態を得る
```

完成です!

[実際に試す](https://try.purescript.han-sel.com/?gist=30f166595eab37a41de5eaad9fb30975)

## 次回予告

[次回](/purescript/tips/statet-monad)ではなんと `StateT` に進化しちゃいます。
