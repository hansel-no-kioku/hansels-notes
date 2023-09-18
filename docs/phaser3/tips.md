# メモ

## シーン

### ゲーム全般

TileMap 使うときは pixelArt を true にしないとタイルの継ぎ目に線が出てしまう

### シーンの起動

[start()](https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#start__anchor) 等に直接 [Phaser.Scene](https://photonstorm.github.io/phaser3-docs/Phaser.Scene.html) のオブジェクトを渡しても、[Phaser.Scenes. SceneManager](https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.SceneManager.html) に同一オブジェクトが登録されていない場合は起動しない(単に無視される)。  
つまり key を渡す以外の起動は現実的ではない。

### Arcade

[Phaser.Physics.Arcade.Sprite](https://photonstorm.github.io/phaser3-docs/Phaser.Physics.Arcade.Sprite.html) の [body](https://photonstorm.github.io/phaser3-docs/Phaser.Physics.Arcade.Sprite.html#body__anchor) も nullable であり、[Phaser.Physics.Arcade. World](https://photonstorm.github.io/phaser3-docs/Phaser.Physics.Arcade.World.html) で [enable](https://photonstorm.github.io/phaser3-docs/Phaser.Physics.Arcade.World.html#enable__anchor) しないと設定されない。

[Phaser.Physics.Arcade. Body](https://photonstorm.github.io/phaser3-docs/Phaser.Physics.Arcade.Body.html) は回転しない。なので GameObject より小さくしたり offset を指定すると、GameObject が回転したときにずれる。
回避するなら表示用 GameObject とは別に衝突判定用 GameObject を用意しグループ化する
