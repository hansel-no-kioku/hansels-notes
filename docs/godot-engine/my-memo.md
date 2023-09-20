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

https://github.com/godot-extended-libraries/godot-debug-menu

使い方は簡単
