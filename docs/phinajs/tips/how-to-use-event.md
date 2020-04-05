# phina.js のイベントの仕組み

!!! note "メモ"
    この記事は 2017/12/4 に [Qiita](https://qiita.com/) へ投稿したものです。

この記事では [phina.js](http://phinajs.com/) のイベントの仕組みについて書きます。[^1]

すでに多少使ったことがある人を想定しています。  
なのでこれらか入門しようと考えている方は [phina.js Tips集](https://qiita.com/alkn203/items/bca3222f6b409382fe20) 等を見ていただいたほうがわかると思います。

なおこの記事を書くにあたって参照した [phina.js](http://phinajs.com/) のバージョンは 0.2.1 です。  
ただし基本的な仕組みは今後もそんなに変わらないと思います。

## どこで使えるの?

EventDispatcher クラスを継承しているクラスのインスタンスで使うことができます。  
もちろん EventDispatcher 自体のインスタンスを作成しても OK です。[^2]

実際のところ [phina.js](http://phinajs.com/) ではゲーム制作でよく使用するクラスの大半が EventDispatcher を継承しています。  
なのでわりとどこでも使えます。

## イベントの検知方法

### イベントリスナの登録

on で始まるメソッドを用意するか、on メソッドで関数を登録することにより指定したイベントを検知できるようになります[^3]

on メソッドは1種類のイベントに対しいくつでも登録できるメリットがあります。  
また自分自身が戻り値として返るのでメソッドチェーンできます。

どちらも指定した種類のイベントが発火すると関数がコールされます。

```js
// foo に対し 'bar' というイベントが発火すると何かする
foo.onbar = function(e) {/* 何かする */};
```

```js
// foo に対し 'bar' というイベントが発火すると何かする
// 2つ登録するとそれぞれコールされて何かする
foo.on('bar', function(e) {/* 何かする */}
   .on('bar', function(e) {/* 何かする */};
```

### イベントリスナでの this について

イベント発火で実行される関数の中では this がイベントリスナ登録先のオブジェクトを指しています。

```js
foo.on('bar', function(e) { this.baz(); /* ここの this は foo */ });
```

JavaScript に日頃から親しんでいればわりと常識かもしれませんが、これを把握していないと思わぬ事故が発生するかもしれません。  
例えば下記のようなシーンを作成して真ん中の青い四角をクリックすると、期待通りにシーンが終了せず実行時エラーになってしまいます。[^4]

```js
phina.define('MainScene', {
  superClass: 'DisplayScene',

  init: function(options) {
    this.superInit(options);

    RectangleShape()
      .addChildTo(this)
      .setPosition(this.gridX.center(), this.gridY.center())
      .setInteractive(true)
      // ボタンが押されたらシーンを終了するために
      // DisplayScene(正確には Scene) クラスの exit メソッドをコールするつもりが…
      .on('pointend', function() { this.exit(); /* 実行時エラー!!!! */ });
  },
});
```

これを回避するには this を self 等に覚えておくか、[bind](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) します。

```js
// this を self に覚えておく
var self = this;
foo.on('bar', function(e) { self.baz(); });
```

```js
// bind で関数の中における this を固定する
foo.on('bar', function(e) { this.baz(); }.bind(this));
```

IE を捨てるか [babel](https://babeljs.io/) 等を使う前提であれば、[アロー関数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/arrow_functions)を使うことでも回避できます。

```js
foo.on('bar', (e) => this.baz() /* ここの this は foo ではなく外の this */);
```

なおイベントリスナに渡される引数(上記e)には target というプロパティが追加されており、そこには必ず登録先のオブジェクト(上記foo)が設定されています。  
このため this を変更しても foo を参照することができます。

### イベントリスナの登録解除

on メソッドで登録したイベントリスナの登録解除は off メソッドで行います。[^5]

```js
foo.off('bar' /* 解除したいイベントの種類 */, listener /* on に渡した関数 */
```

on メソッドに渡した関数が必要ですので、無名関数をそのまま渡して保持していない場合は off メソッドで登録を解除できなくなります。  
また [bind](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) を使用している場合、[bind](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) によって元の関数オブジェクトとは別の関数オブジェクトが生成される点にも注意が必要かもしれません。[^6]

```js
var listener = function(e) { /* 何かする */ };

foo.on('bar', listener.bind(this)); // listner とは別の関数オブジェクトを登録してしまう

foo.off('bar', listener); // 登録解除されない!!
```

```js
var listener = function(e) { /* 何かする */ }.bind(this);

foo.on('bar', listener);

foo.off('bar', listener);  // 登録解除される
```

なお指定したイベントに対し登録されているイベントリスナをまとめて解除するメソッドもあります。

```js
foo.clear('bar');
```

不要になったイベントリスナを登録したままにすると、イベント発火時に思わぬ動作になるだけでなく、イベントリスナの登録を介して不要になったオブジェクトの参照が残っていることでいつまでもメモリから消されないといったことが起こるかもしれません。

実際どのくらい問題になるのかは自分では経験不足でわかりません。  
ちょっとしたミニゲームではそうそう問題にはならないと思いますが。

## イベントの発火方法

fire または flare メソッドで発火します。
両者の違いは引数です。

fire の場合はオブジェクトを1つだけ引数に渡します。

```js
foo.fire({
  // イベントの種類
  // これを忘れると 'undefined' とう名前になってしまう
  type: 'bar',
  // イベントで送りたい情報
  // なくてもいいしいくつあってもいい
  // ただし target という名前は使用不可(fire の中で上書きされる)
  baz: 'qux',
});
```
flare の場合は文字列を1つとオブジェクト1つ(省略可)を引数に渡します。

```js
foo.flare('bar' /*イベントの種類*/,
  {
    // イベントで送りたい情報
    // なくてもいいしいくつあってもいい
    // ただし target という名前は使用不可(flare の中で上書きされる)
    // また type という名前を使用するとイベントの種類を上書きする
    baz: 'qux',
  }
);

// 何も送る情報がない場合はイベントの種類だけ渡しても OK
foo.flare('bar');
```

両者とも戻り値として自身が返ってくるのでメソッドチェーンできます。

```js
foo.flare('bar').flare('baz').flare('qux').flare('quux');
```

なおイベントは同期で実行されます。  
つまり fire や flare の中でイベントリスナとして登録された関数がコールされます。  
このため複数登録できる on メソッドはともかく、on で始まるメソッドの場合は下記とわりと同義です。[^7]

```js
foo.onbar && foo.onbar(e);
```

## 別名について

EventDispatcher の各メソッドには下記のような別名が用意されています。  
機能は全く同じです。

| メソッド | 別名 |
:--|:--|
| on | addEventListener |
| off | removeEventListener |
| clear | clearEventListener |
| has | hasEventListener |
| fire | dispatchEvent |
| flare | dispatchEventByType |

[^1]: 本当は [phina.js](http://phinajs.com/) の各クラスで使用しているイベントの種類について書くつもりでしたが、仕組みについて書き始めたら長くなってしまいました。イベントの種類もいずれ記事にしたいと思います。ちなみに今回が Qiita 初投稿です。
[^2]: テストや実験以外の用途がちょっと思いつきませんが。
[^3]: この記事では記載を省略しましたが、一度発火するとイベントリスナの登録が自動で解除される one というメソッドもあります。
[^4]: 自分はこれ、よくやってしまいます…。
[^5]: on で始まるメソッドの場合には null を代入したり delete でメソッドを消せば OK です。
[^6]: [bind](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 使えるような人に必要な心配かちょっと怪しかったかも。
[^7]: シーンやシーンの子(孫以下含む)で毎フレームコールされる update メソッドがこの、あればコールされる、という仕組みになっています。

