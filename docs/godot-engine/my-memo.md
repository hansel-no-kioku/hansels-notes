# Godot Engine 自分メモ

!!! note
    [Godot](https://godotengine.org/) Ver.4.1.1 時点

## Godot Editor での補完

### 公式

[Static typing in GDScript — Godot Engine (4.1) documentation in English](https://docs.godotengine.org/en/4.1/tutorials/scripting/gdscript/static_typing.html)

ようするに型が決まればよい

### 静的型付の強制

型が動的か静的かはテキストエディタの行番号が緑色かどうかで判断できる  
緑色なら静的に決定されており、灰色なら動的に解決されている

2023/9/12 に [Add an optional `untyped_declaration` warning by ryanabx · Pull Request #81355 · godotengine/godot · GitHub](https://github.com/godotengine/godot/pull/81355) がマージされているっぽいのでそのうち `プロジェクトの設定` > `デバッグ` > `untyped_declaration` で動的型付部分に警告出させることができそう

### Node#get_node に対する補完

[Node#get_node](https://docs.godotengine.org/en/4.1/classes/class_node.html#class-node-method-get-node) または $[ノード名] の返り値の型は [Node](https://docs.godotengine.org/en/4.1/classes/class_node.html#class-node) なので自作クラスの場合追加したメンバが補完されない。

自作クラスに `class_name MyClass` と名前を登録した上で `($ChildNode as MyClass).` のようにダウンキャストする。  
何度も出てくる場合 `var child_node := $ChildNode as MyClass` とするといいかも。  
ここで `var child_node: MyClass = $ChildNode` としてはダメ。  
たぶん `as` を使わないと `Node` の参照が代入されるイメージになるっぽい。

## デバッグ

### FPS 等の表示

デバッグ実行時は Godot Editor 下にある `デバッガー` > `モニター` で確認できる

#### AssetLib

https://github.com/godot-extended-libraries/godot-debug-menu

使い方は簡単

## Node の使い分け

### アクションゲーム等マップ上で動くキャラクターやエンティティ

基本となる Node に画像や衝突範囲を表す子ノードを追加して構成する

#### 基本

| Node             | 特徴                                                                                    |
| ---------------- | --------------------------------------------------------------------------------------- |
| Area2D           | 物理演算は行わず衝突判定のみ行う。速度等のプロパティは持たずコードで位置を指示          |
| CharacterBody2D  | 他の *Body2D と物理演算可能。移動はコードで指示                                         |
| RigidBody2D      | 他の *Body2D と物理演算可能。コードからは力を加えるのみ                                 |
| StaticBody2D     | 他の *Body2D と物理演算可能。動かない壁。コードで移動させた場合物理演算なしでワープする |
| AnimatableBody2D | StaticBody2D の派生。コードで移動させた場合に経路上の他の物体と衝突する                 |

https://docs.godotengine.org/en/stable/tutorials/physics/physics_introduction.html

#### 画像

AnimatedSprite2D や Sprite2D を基本に追加

#### 衝突形状

基本の子に CollisionShape2D を追加し、CollisionShape2D の shape プロパティに *Shape2D を設定する

## Pixelゲーム

基本拡大でぼやけさせないはず

### 全体

`プロジェクト` > `プロジェクト設定` > `表示` > `ウィンドウ` > `ストレッチ` > `モード` を `viewport` にする

### Sprite

scale を 1 より大きくする可能性がある場合以下が有効

`インスペクター` > `Texture` > `Filter` を `Nearest` にする  
ただし親ノードが `Nearest` であれば `inherit` でもよいはず
