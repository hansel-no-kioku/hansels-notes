# phina.js の Shape での origin による座標計算は padding を含む

!!! note "メモ"
    この記事は 2018/7/8 に [Qiita](https://qiita.com/) へ投稿したものです。

[phina.js](http://phinajs.com/) の小ネタです。

以前に調べて知っていたはずなのに、久しぶりにまた引っかかったので記事に残しておきます。

実のところタイトルがほぼ全てなのですが、それでは寂しいので少し解説を書いてみます。

## origin について

`Object2D` を継承している各クラスは `origin` で座標の基準を指定することが出来ます。  
(0.0, 0.0) で左上が、(0.5, 0.5) で真ん中が、(1.0, 1.0) では右下が基準になります。  
0.0 ～ 1.0 の範囲外を指定することで図形の外に基準を置くことも出来ます。  
ちなみにデフォルトは (0.5, 0.5) です。

参考: [[phina.js-Tips] Shapeの原点を変更する](https://qiita.com/alkn203/items/d0954528f23f8c63766d)

## Shape の origin

ここからが本題です。

`RectangleShape` などの各 `Shape` も `Object2D` を継承していますので、もちろん `origin` を指定できます。

ただし `Shape` を継承したクラスの場合は `origin` による座標の計算時に `width` や `height` だけでなく `padding` も考慮されます。

例えば `RectangleShape` の場合 `origin` の (0, 0) は矩形の左上ではなく矩形からさらに `padding` 分左上の位置になります。

こんな感じです。

```javascript
phina.globalize();

phina.define('MainScene', {
  superClass: 'DisplayScene',

  init: function(params) {
    this.superInit(params);

    // シーンの背景を赤にしておく
    this.backgroundColor = "red";

    // シーンと同じ大きさの矩形を左上に合わせて追加(したつもり)
    var rect = RectangleShape()
    　.setOrigin(0.0, 0.0)
    　.setPosition(0, 0)
    　.setSize(this.width, this.height)
      .addChildTo(this);
  },
});

phina.main(function() {
  // 結果を見やすくするためにゲームサイズを小さくする
  GameApp({
    width: 100,
    height: 100,
    startLabel: 'main'
  }).run();
});
```

実行結果:  
![RectangleShape.png](/images/articles/coordinates-of-shapes1.png)

コードだけ見るとシーン全体を矩形で埋め尽くしそうな気がしますが、実際には `padding` 分右下にずれて描画されました。

ちなみに `padding` のデフォルトは 8 です。

## padding について

上の例では `padding` を 0 にすることで矩形がシーンにぴったり重なるようになります。

しかし `padding` のデフォルトがなぜ 0 ではないのか? あるいはそもそも `padding` の役割は? といった話はググっても全然ヒットしませんでした。

なので、元の話題からちょっと派生して `padding` についても簡単に書いておきます。

`Shape` の派生クラスでは `stroke` と `strokeWidth` で枠を描画することが出来ます。  
そしてこの枠は  `strokeWidth` の半分の幅で元の図形の**外側**に描画されます。

例えば、

```javascript
var rect = RectangleShape({
  width: 100,
  height: 100,
  stroke: "red",
  strokeWidth: 10
});
```

と指定すると、縦横 100px の矩形の外側に幅 5px の赤い枠が描画されます。

ここでもし `padding` を 0 にしてしまうと、この赤い枠が描画されなくなります。

なぜなら `RectangleShape` では、`width` および `height` で指定された範囲 + `padding` 分の領域だけシーンに描画される仕組みになっているためです[^1]。

`padding` のデフォルトは 8 でしたので、`strokeWidth` が 16 を超えると、それ以上いくら数字を大きくしても枠は太くなりません[^2]。  
この場合 `padding` をその分大きくすることで描画されるようになります。

なお `shadow` と `shadowBlur` を指定する場合も同様です。

[^1]: `Shape` から派生している他のクラスも基本的に同様です。
[^2]: 丸い角が直角になるという変化はあったりします。
