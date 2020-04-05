# phina.js でフレームレート(fps)やメモリ使用量を表示する

!!! note "メモ"
    この記事は 2018/3/21 に [Qiita](https://qiita.com/) へ投稿したものです。

この記事では [phina.js](http://phinajs.com/) に組み込まれている [stats.js](https://github.com/mrdoob/stats.js/) の機能を使って、開発中のゲームのフレームレート(fps)やメモリ使用量を確認する方法について書きます。

なおこの記事を書くにあたって参照した [phina.js](http://phinajs.com/) のバージョンは 0.2.3 です。  
ただし基本的な仕組みは今後もそんなに変わらないと思います。

## 基本的な使い方

すでに [[phina.js] サンプルから覚えるphina.js](https://qiita.com/simiraaaa/items/ba83ce70cb091e8bdfab#appenablestats) や [[phina.js-Tips-44] ゲームのfpsを変えてみる](https://qiita.com/alkn203/items/ffd7c59498cdc932f323) 等でも紹介されていますので、[phina.js](http://phinajs.com/) を使っている方であればすでに知っているかもしれませんが一応書いておきます。

使い方は本当に簡単で GameApp のメソッド enableStats() をコールするだけです。

```JavaScript
phina.main(function() {
  var app = GameApp();

  app.enableStats();  // コレだけ!!!

  app.run();
});
```

[Runstant](http://runstant.com/projects?tag=phina.js) でもみなさん表示されていますね。

グラフをクリックすると、フレームレート(fps) → 1フレームあたりの時間(ms) → メモリ使用量(Mbyte)、と切り替わります。

## 仕組み解説

enableStats() は、GameApp クラスのずっと上のスーパークラスの BaseApp (GameApp ← CanvasApp ← DomApp ← BaseApp) で定義されています。[^1]

メソッドをコールすると、global にすでに Stats が定義されている(≒[stats.js](https://github.com/mrdoob/stats.js/) が読み込まれている)ならその Stats を、なければインターネットから [stats.js](https://github.com/mrdoob/stats.js/) を自動的に読み込んで表示します。

事前に [stats.js](https://github.com/mrdoob/stats.js/) をダウンロードして html から script タグ等で読み込むことでインターネットに接続していない環境でも動作させることが出来ます。

[phina.js](http://phinajs.com/) から自動で読み込む場合は [stats.js](https://github.com/mrdoob/stats.js/) の r14 が読み込まれます。  
自前で読み込む場合には好きなバージョンのものを使うことが出来ますが互換性に注意が必要です。  
2018/3/21 時点の [stats.js](https://github.com/mrdoob/stats.js/) の最新は r17 です。  
r14 と比べると表示位置がデフォルトで左上になる等いろいろ変わっているようです。

## stats.js の表示内容をプログラムで変更する

enableStats() をコールしてグラフを表示すると app に stats というプロパティが生成されます。  
この stats のメソッド setMode() をコールすることでグラフの表示内容を変更することが出来ます。

```JavaScript
// グラフの表示を表示内容を変更する
//   0: フレームレート, 1: 1フレームあたりの時間, 2: メモリ使用量
//   r15 以降は showPanel() でも OK
app.stats.setMode(1);
```

ただし stats プロパティが生成されるタイミングについて注意する必要があります。  
事前に [stats.js](https://github.com/mrdoob/stats.js/) を読み込んでいれば enableStats() コール直後に生成されているのですが、[phina.js](http://phinajs.com/) から自動で読み込む場合は非同期となり少し時間が経ってから でないと stats プロパティが生成されません。

## ユーザ定義パネルの使い方

[stats.js](https://github.com/mrdoob/stats.js/) では r15 からユーザ定義パネルの機能が追加されています。

こんな感じで簡単に使うことが出来ます。

```JavaScript
  // stats.js を有効にする(注: r15 以降を事前に読み込んでおく)
  app.enableStats();

  // ユーザ定義パネル生成(name, fg, bg)
  app.myPanel = new Stats.Panel('Test', '#cc6', '#440');

  // ユーザ定義パネル追加
  app.stats.addPanel(app.myPanel);

  // 4つめのパネル(=上で追加したパネル)を表示
  app.stats.showPanel(3);
```

追加したパネルに対し update() メソッドをコールするとグラフが更新されます。

```JavaScript
// ユーザ定義パネルのグラフを更新
app.myPanel.update(newValue, maxValue);
```

簡単なサンプルは [Runstant](http://runstant.com/Hansel/projects/93e3c3fa) にあります。

[^1]: [ここ](https://github.com/phinajs/phina.js/blob/v0.2.3/src/app/baseapp.js#L133)です。
