# Markdown チートシート

Markdown の[基本的な記法](https://daringfireball.net/projects/markdown/syntax)については省略。  
このサイトで mkdocs.yml の markdown_extensions に定義している分のみ記載する。

## extra

[Python-Markdown Extra](https://python-markdown.github.io/extensions/extra/)

### abbr

```markdown
abbr ってなあに?

*[abbr]: Abbreviation = 略語
```

abbr ってなあに?

*[abbr]: Abbreviation = 略語

### attr_list

```markdown
something {: #someid .someclass somekey='some value' }
```

### def_list

```markdown
モナド
: モナドは単なる自己関手の圏におけるモノイド対象だよ。何か問題でも？
```

モナド
: モナドは単なる自己関手の圏におけるモノイド対象だよ。何か問題でも？

### fenced_code

~~~markdown
```haskell hl_lines="4 6"
main = mapM_ print $ take 100 $ fizzBuzz <$> [1..]

fizzBuzz n
  | n `mod` 3 == 0 = "Fizz"
  | n `mod` 5 == 0 = "Buzz"
  | n `mod` 15 == 0 = "FizzBuzz"
  | otherwise = show n
```
~~~

```haskell hl_lines="4 6"
main = mapM_ print $ take 100 $ fizzBuzz <$> [1..]

fizzBuzz n
  | n `mod` 3 == 0 = "Fizz"
  | n `mod` 5 == 0 = "Buzz"
  | n `mod` 15 == 0 = "FizzBuzz"
  | otherwise = show n
```

### footnotes

```markdown
あれ[^1]は…いいものだ。

[^1]: 第37話参照。
```

あれ[^1]は…いいものだ。

[^1]: 第37話参照。

### tables

```markdown
名称 | 型式番号 | 重量
:--|:--:|--:
νガンダム | RX-93 | 27.9t
サザビー | MSN-04 | 30.5t
```

名称 | 型式番号 | 重量
:--|:--:|--:
νガンダム | RX-93 | 27.9t
α・アジール | NZ-333 | 128.6t

## admonition

```markdown
!!! danger "危険"
    死ぬぜぇ

!!! error "エラー"
    酔うな、己の戦いに

!!! attention "注意"
    死ぬほど痛いぞ

!!! warning "警告"
    お前を殺す

!!! hint "ヒント"
    事は全てエレガントに運べ

!!! important "重要"
    戦っちゃいけないんだ

!!! tip "TIPS"
    弾切れを気にする必要はない

!!! note "メモ"
    逃げも隠れもするが嘘は言わない

!!! success "成功"
    とか

!!! fail "失敗"
    とか

!!! caution "注意"
    とかもあり。たくさんある。
```

!!! danger "危険"
    死ぬぜぇ

!!! error "エラー"
    酔うな、己の戦いに

!!! attention "注意"
    死ぬほど痛いぞ

!!! warning "警告"
    お前を殺す

!!! hint "ヒント"
    事は全てエレガントに運べ

!!! important "重要"
    戦っちゃいけないんだ

!!! tip "TIPS"
    弾切れを気にする必要はない

!!! note "メモ"
    逃げも隠れもするが嘘は言わない

!!! success "成功"
    とか

!!! fail "失敗"
    とか

!!! caution "警告"
    とかもあり。たくさんある。

## codehilite

省略

## pymdownx

[PyMdown Extensions](https://facelessuser.github.io/pymdown-extensions/)

### arithmatex

#### inline

```markdown
こんな感じ → $\frac{n!}{k!(n-k)!} = \binom{n}{k}$
```

こんな感じ → $\frac{n!}{k!(n-k)!} = \binom{n}{k}$

#### block

```markdown
$$
  \require{AMScd}
  \begin{CD}
  A @>{f}>> B\\
  @V{g}VV @VV{h}V\\
  C @>>{k}> D
  \end{CD}
$$
```

$$
  \require{AMScd}
  \begin{CD}
  A @>{f}>> B\\
  @V{g}VV @VV{h}V\\
  C @>>{k}> D
  \end{CD}
$$

### caret

```markdown
^^underline^^

x^2^ + y^2^
```

^^underline^^

x^2^ + y^2^

### keys

```markdown
++ctrl+alt+delete++
```

++ctrl+alt+delete++

### mark

```markdown
==ここテストに出ます==
```

==ここテストに出ます==

### tasklist

```markdown
* [x] 苺ショート
* [ ] シュークリーム
```

* [x] 苺ショート
* [ ] シュークリーム

### tilde

```markdown
~~deleted~~

H~2~O
```

~~deleted~~

H~2~O
