# phina.js のイベント一覧

!!! note "メモ"
    この記事は 2017/12/12 に [Qiita](https://qiita.com/) へ投稿したものです。

[前回の記事](https://qiita.com/hansel_no_kioku/items/a4fd71fd5817c1327799)では phina.js のイベントの仕組みについて書きました。  
今回は [phina.js](http://phinajs.com/) で実際に使われているイベントの種類について書きます。

ひたすら表を並べていきます。[^1]

なお調査した [phina.js](http://phinajs.com/) のバージョンは 0.2.1 です。

自分ではまだ試していないものも記載しています。  
実際の動作と異なるものが見つかったら随時更新していきたいと思います。

## phina.js イベント一覧

### 表のルール

下記のようなコードがあった場合に、`Foo` を発火させるクラス、`buz` をイベント、`qux: 'quux'` をプロパティとして表に記載します。

```js
phina.define('Foo',        // 発火させるクラス
{
  superClass: 'EventDispatcher',

  init: function() {
    this.superInit();

    this.bar = Bar();      // イベントを受けるクラス(表のタイトル)
    this.bar.flare('buz',  // イベント
      {qux: 'quux'}        // プロパティ
    );
  },
});
```

### phina.util.Ticker

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| tick | なし | 自身 | 毎フレーム |

### phina.util.ChangeDispatcher

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| change | なし | 自身 | 登録したいずれかのプロパティの値変更 |

### phina.asset.AssetLoader

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| progress | key: ロード完了したアセット<br>asset: アセットの種類<br>progress: 全体の進捗(0.0～1.0) | 自身 | 1つロードする毎 |
| load | なし | 自身 | 全てロード |

### phina.asset.Sound

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| ended | なし | 自身 | 再生終了 |
| stop | なし | 自身 | stop メソッドによる停止 |
| pause | なし | 自身 | pause メソッドによる一時停止 |
| resume | なし | 自身 | resume メソッドによる一時停止 |
| decodeerror | なし | 自身 | 音声ファイルのデコード失敗 |
| notfound | なし | 自身 | 音声ファイルで 404 きたら |
| servererror | なし | 自身 | 音声ファイルで 500 等きたら |
| loaderror | なし | 自身 | notfound や servererror と同時 |

### phina.input.Keyboard

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| keydown | keyCode: 数値 | 自身 | キーを押したとき |
| keyup | keyCode: 数値 | 自身 | キーを離したとき |
| keypress | keyCode: 数値 | 自身 | 非修飾キーを押したとき |

いずれも1フレーム1キーしか検知出来ないはずです(未実験)。

### phina.input.GamepadManager

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| connected | gamepad: Gamepad のインスタンス | 自身 | ゲームパット接続 |
| disconnected | gamepad: Gamepad のインスタンス | 自身 | ゲームパット接続解除 |

この辺は全く使ったことがないのでちょっと自信がありません。  
[Window.ongamepadconnected - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/ongamepadconnected) によると IE ではイベント来ないようです。

### phina.app.Element

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| enterframe | app: GameApp のインスタンス[^2] | Updater | 毎フレーム(updateの直前) |
| added | なし | 追加先の Element | addChildTo 等で親に追加されたとき |
| removed | なし | 元親の Element | remove 等で親から切り離されたとき |
| dragstart | なし | Draggable | ドラッグ開始時 |
| drag| なし | Draggable | ドラッグ移動中毎フレーム |
| dragend| なし | Draggable | ドラッグ終了時 |
| click | なし | DomElement | クリックされたとき[^3] |

enterframe はカレントシーンまたはその子(孫以下含む)であり、かつ親も含めて起きている(awake)ときにのみ発火します。

dragstart/drag/dragend は Draggable が attach されているときにのみ発火します。  
なお Draggable を含む一部 Accessory は Element クラスの対象プロパティを参照するだけで自動的に attach されます。

click はカレントシーンまたはその子(孫以下含む)のときに発火します。

### phina.app.Object2D / phina.display.DisplayScene

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| pointover | pointer: Touch か Mouse のインスタンス<br>interactive: Interactive のインスタンス<br>over: true/false | Interactive | カーソルが上に来たとき |
| pointout | pointer: Touch か Mouse のインスタンス<br>interactive: Interactive のインスタンス<br>over: true/false | Interactive | カーソルが外れたとき |
| pointstart | pointer: Touch か Mouse のインスタンス<br>interactive: Interactive のインスタンス<br>over: true/false | Interactive | タッチしたりマウスボタンを押したとき |
| pointstay | pointer: Touch か Mouse のインスタンス<br>interactive: Interactive のインスタンス<br>over: true/false | Interactive | タッチ中やマウスボタン押下中毎フレーム |
| pointmove | pointer: Touch か Mouse のインスタンス<br>interactive: Interactive のインスタンス<br>over: true/false | Interactive | タッチ中やマウスボタン押下中に動かしたとき |
| pointend | pointer: Touch か Mouse のインスタンス<br>interactive: Interactive のインスタンス<br>over: true/false | Interactive | 指やマウスボタンを離したとき |

いずれもカレントシーンまたはその子(孫以下含む)であり、かつ親も含めて起きて(awake)いて、自身が interactive なときにのみ発火します。

### phina.app.BaseApp

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| replace | なし | 自身 | replaceScene メソッドコール |
| push | なし | 自身 | pushScene メソッドコール |
| pushed | なし | 自身 | pushScene メソッドコールでシーン変更後 |
| pop | なし | 自身 | popScene メソッドコール |
| poped | なし | 自身 | popScene メソッドコールでシーン変更後 |
| changescene | なし | 自身 | replace や push、pop と同時 |
| enterframe | なし | 自身 | 毎フレーム(updateの直前) |

### phina.app.Scene

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| enter | app: GameApp のインスタンス[^2] | BaseApp | カレントシーンになったとき |
| pause | app: GameApp のインスタンス[^2] | BaseApp | 上に別のシーンが乗ったとき |
| exit | app: GameApp のインスタンス[^2] | BaseApp | popScene により消されるとき |
| resume | app: GameApp のインスタンス[^2]<br>prevScene: 上にあったシーン | BaseApp | 上に乗っていたシーンが消えたとき |
| keydown | keyCode: 数値 | DomApp | キーを押したとき |
| keyup | keyCode: 数値 | DomApp | キーを離したとき |
| keypress | keyCode: 数値 | DomApp | 非修飾キーを押したとき |
| focus | なし | DomApp | ウィンドウがフォーカスされたとき |
| blur | なし | DomApp | ウィンドウからフォーカスが外れたとき |

### phina.accessory.Accessory

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| attached | なし | Element | Element に attach されたとき |
| detached | なし | Element | Element から切り離されたとき |

### phina.accessory.Tweener

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| tween | なし | 自身 | アニメ中毎フレーム |

### phina.accessory.Draggable

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| dragstart | なし | 自身| ドラッグ開始時 |
| drag| なし | 自身| ドラッグ中毎フレーム |
| dragend| なし | 自身| ドラッグ終了時 |
| backend | なし | 自身| ドラッグ開始位置に戻ったとき |

### phina.accessory.Flickable

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| flickstart | なし | 自身| フリック開始時 |
| flickcancel| なし | 自身| フリックせずに離したとき |

### phina.graphics.CanvasRecorder

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| finished | なし | 自身 | アニメ生成終了? |

[gif.js](https://github.com/jnordberg/gif.js) を使って Canvas を録画するっぽいですが、まだ全く触ったことがないのでぜんぜん判りません。  
[[phina.js] サンプルから覚えるphina.js](https://qiita.com/simiraaaa/items/ba83ce70cb091e8bdfab#canvasrecorder) に説明があります…というか他で見たことがないです。

### phina.display.DomApp

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| focus | なし | 自身 | ウィンドウがフォーカスされたとき |
| blur | なし | 自身 | ウィンドウからフォーカスが外れたとき |

### phina.ui.Button

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| push | なし | 自身 | ボタンが押されとき[^3] |

### phina.ui.Gauge

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| change | なし | 自身 | ゲージが変化するとき |
| changed | なし | 自身 | ゲージが変化し終わったとき |
| empty | なし | 自身 | ゲージが空になったとき |
| full | なし | 自身 | ゲージが満タンになったとき |

### phina.game.ManagerScene

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| finish | なし | 自身 | 次のシーンがなかったとき |

### phina.game.LoadingScene

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| loaded | なし | 自身 | ロード完了時 |

### phina.game.CountScene

| イベント | プロパティ | 発火させるクラス | 発火条件 |
|:-- |:-- |:-- |:-- |
| finish | なし | 自身 | カウンタが 0 になったとき |

[^1]: とにかくたくさん紹介することを優先したため説明がちょっと雑です。よく使いそうなものについてはいずれもう少し詳しく書きたいと思います。
[^2]: 正確には BaseApp またはその派生クラスのインスタンスですが、大抵の人は GameApp を使っていると思います。
[^3]: 指やマウスボタンを押したときではなく離したときに発火します。
