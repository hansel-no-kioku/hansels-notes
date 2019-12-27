# StateTモナド

[Stateモナド](/purescript/tips/state-monad)の続きです。

## 1. コンソールに出力したい…けど

よく考えると `doSomething :: forall a. Show a => a -> Unit` に渡したところで何一つ面白いことがありません。  
副作用がない PureScript において `Unit` を返す関数は結局なにもしてくれないのです。

面白いこと…例えば `logShow :: forall a. Show a => a -> Effect Unit` に渡してコンソールに出力とかしてみたいですね。

というわけで[前回最後のコード](https://try.purescript.han-sel.com/?gist=30f166595eab37a41de5eaad9fb30975)の `doSomething` を `logShow` に置き換えてみます。

```haskell
transit :: State Int Unit
transit = do
  modify (_ + 1)
  b <- gets even
  _ <- lift $ logShow b  -- 現在の状態が偶数かどうかを表示する
  modify (_ + 2)
  s <- get
  _ <- lift $ logShow s  -- 現在の状態を表示する
  put 3
  modify (_ + 4)
```

[実際に試す](https://try.purescript.han-sel.com/?gist=c3327ea51d1541e21aee26f738398f30)

…何も出力されませんね。

実は `_ <- lift $ logShow b` の `_ <-` で**作用**を捨てちゃっているからなんです。

`logShow` が返す `Effect unit` 型の値は、コンソールに何かを出力するという作用を表す**ただの値**です。  
この値は main 関数から処理系に渡すことで実際の作用が発生します。

つまり `_ <-` で捨てたりせずなんとかして最後まで受け渡す必要があります。

## 2. まずは力技でなんとかする

do記法はもともとただの糖衣構文であり脱糖した後は `bind` で結合したただのクロージャになっています。  
つまり `a <-` で一度名前に束縛した値は以降どこでも参照できるのです。

これを利用して作用を捨てずに拾って返すようにしてみましょう。


```haskell
transit :: State Int (Effect Unit)
transit = do
  modify (_ + 1)
  b <- gets even
  e <- lift $ logShow b  -- 現在の状態が偶数かどうかを表示する
  modify (_ + 2)
  s <- get
  f <- lift $ logShow s  -- 現在の状態を表示する
  put 3
  modify (_ + 4)
  pure do                -- 作用を1つに繋げて出力する
    e
    f
```

できました!

あとで状態だけでなく出力も取り出す必要があるので `execState` の代わりにこんな関数を用意すると便利そうです。

```haskell
runState (State m) = m  -- 初期状態を渡して最後の状態と出力を得る
```

[実際に試す](https://try.purescript.han-sel.com/?gist=99ab2f2a08476d0215ab27eff9f7763e)

## 3. さらに力技でなんとかする

しかしクロージャじゃないとダメというのも不便なときがありそうです。  
ではいっそ状態を `Int` から `Effect Int` にしてはどうでしょう?

```haskell
transit :: State (Effect Int) Unit
transit = do
  modify    $ map (_ + 1)    -- Effect の中で状態を +1 する
  b <- gets $ map even       -- Effect の中で状態が(以下省略)
  _ <- lift $ logShow =<< b  -- Effect から結果を取り出して表示
  modify    $ map (_ + 2)    -- Effect の中で(以下省略)
  s <- get
  f <- lift $ logShow =<< s  -- Effect から(以下省略)
  put       $ pure 3         -- 状態を Effect に包んで設定
  modify    $ map (_ + 4)    -- (以下省略)
```

もうなんだひどいことになっていますが、とにかくできそうです。

[実際に試す](https://try.purescript.han-sel.com/?gist=2727352a2557e7a0594e862034fa5eaa)

## 4. かっこいい方法を考える

ここまで来たらいっそ `Int -> Effect (Tuple a Int)` を繋げるようにしたらどうでしょう?

一旦モナドは忘れてまずはこんな感じで定義してみます。

```haskell
type StateT s a = s -> Effect (Tuple a s)

bind' m f = \s -> m s >>= \(Tuple v s') -> f v s'
infixl 1 bind' as >>-

modify f = \s -> pure $ Tuple unit  (f s)
gets   f = \s -> pure $ Tuple (f s) s
get      = \s -> pure $ Tuple s     s
put    s = \_ -> pure $ Tuple unit  s
lift   m = \s -> m >>= \x -> pure $ Tuple x s

execStateT m s = snd <$> m s
```

こうすれば特別に意識しなくても作用を状態遷移中に書けそうです。

```haskell
transit :: StateT Int Unit
transit   = modify (_ + 1)          -- 状態を +1 する
  >>- \_ -> gets   even             -- 現在の状態が偶数か調べる
  >>- \b -> lift   (logShow b)      -- 調べた結果を表示する
  >>- \_ -> modify (_ + 2)          -- 状態を(以下略)
  >>- \_ -> get                     -- 状態を取得する
  >>- \s -> lift   (logShow s)      -- 現在の状態を表示する
  >>- \_ -> put    3                -- 状態を 3 にする
  >>- \_ -> modify (_ + 4)          -- (以下略)
```

[実際に試す](https://try.purescript.han-sel.com/?gist=d376536b5dd856c9be9cc62dd1fd023a)

## 5. Effect 以外でも使えるようにする

4.で作成した `StateT` は別に `Effect` じゃなくても `pure` や `bind` があれば使えそうです。  
つまり `Monad` ならなんでもよさそうです。

なので `Effect` を `m` に置き換えてみます。

```haskell
type StateT s m a = s -> m (Tuple a s)
```

これだけで `Effect` 以外でも OK になります。

試しに `Aff` でやってみましょう。

```haskell
transit :: StateT Int Aff Unit
transit   = modify (_ + 1)          -- 状態を +1 する
  >>- \_ -> gets   even             -- 現在の状態が偶数か調べる
  >>- \b -> lift   (logShow b)      -- 調べた結果を表示する
  >>- \_ -> lift   (delaySec 2.0)   -- 2秒待つ
  >>- \_ -> modify (_ + 2)          -- 状態を(以下略)
  >>- \_ -> get                     -- 状態を取得する
  >>- \s -> lift   (logShow s)      -- 現在の状態を表示する
  >>- \_ -> lift   (delaySec 2.0)   -- 2秒待つ
  >>- \_ -> put    3                -- 状態を 3 にする
  >>- \_ -> modify (_ + 4)          -- (以下略)
```

[実際に試す](https://try.purescript.han-sel.com/?gist=3e6b461e6c3caacd246fcdfeda837aad)

## 6. StateT も Monad のインスタンスにする

後は[前回](/purescript/tips/state-monad)と同じです。

```haskell
-- 新しい型にする
newtype StateT s m a = StateT (s -> m (Tuple a s))

-- Monad のインスタンスにする
instance functorStateT :: Functor m => Functor (StateT s m) where
  map f (StateT a) = StateT (\s -> map (\(Tuple b s') -> Tuple (f b) s') (a s))

instance applicativeStateT :: Monad m => Applicative (StateT s m) where
  pure a = StateT \s -> pure $ Tuple a s

instance applyStateT :: Monad m => Apply (StateT s m) where
  apply = ap

instance bindStateT :: Monad m => Bind (StateT s m) where
  bind (StateT x) f = StateT \s ->
    x s >>= \(Tuple v s') -> case f v of StateT st -> st s'

instance monadStateT :: Monad m => Monad (StateT s m)

-- いろいろな関数を用意する
modify f = StateT \s -> pure $ Tuple unit  (f s)
gets   f = StateT \s -> pure $ Tuple (f s) s
get      = StateT \s -> pure $ Tuple s     s
put    s = StateT \_ -> pure $ Tuple unit  s
lift   m = StateT \s -> m >>= \x -> pure $ Tuple x s

execStateT (StateT m) s = snd <$> m s
```

[実際に試す](https://try.purescript.han-sel.com/?gist=ed367e433871efe11c13b4b85159468d)

## 7. まとめ

ある型コンストラクタが別の型コンストラクタを引数になっていて、両方とも Monad のインスタンスで、さらに相互に変換する仕組みがあれば、それがモナド変換子です。

判ってしまえば難しいことは何もないのですが…初めはいくら説明されてもなかなか脳が受け付けません。  
いろいろ足掻いているうちにある日突然脳が、これか! と納得するのです。

なので実際のところ2017年の自分がこの記事を読んですぐに判るかというと…あやしいですね(笑)
