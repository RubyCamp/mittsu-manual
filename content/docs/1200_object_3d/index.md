---
title: 'Object3Dクラス'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 1200
summary: Mittsuにおける3Dオブジェクトの基本クラスであるObject3Dについて説明します。
---

## 3D空間の物体に共通する振る舞いや属性をまとめる

「Object3D」クラスは、Mittsuにおける3Dキャラクタの基本となるクラスです。

3D空間に配置され、Rendererによってウィンドウ上に描画されることになる全てのオブジェクトは、
このObject3Dクラスを継承します。

これは、立方体や球体、トーラスなどの物体だけではなく、カメラやライトのような光源類、
そしてSceneクラスそのものも、全てObject3Dを継承し、そのメソッドが利用できます。

言い換えれば、Object3Dの主要な機能を理解してさえいれば、カメラ・光源・物体といった3D空間における
登場人物の全てを同じ操作で扱えるということになります。

## 主要な属性

##### id[読み取り専用]《int》

オブジェクトに与えられる一意な識別子です。

##### name[読み書き可能]《string》

オブジェクトに与えられる名称です。

オブジェクトに分かりやすい名称を付けておくと、後述する「get_object_by_property(get_object_by_name)」
メソッドで希望する子オブジェクト（childオブジェクト）を簡単に探し出すことができたりします。

##### children[読み書き可能]《Array》

当該オブジェクトに属するchildオブジェクトが全て格納された配列です。

例えば、以下のように使うと、シーンに属する全てのオブジェクトの名称を階層的に表示したりする
ことも可能です。

```ruby
p scene.name
scene.children.each do |child|
  p " - #{child.name}"
  child.children.each do |child2|
    p "   - #{child2.name}"
  end
end
```

###### 実行結果

```
"<Scene #1>"
" - <Mesh #3>"
"   - <Mesh #4>"
```

なお、単に階層構造を確認するだけならば、「print_tree」メソッドを使えば1行で実行できます。

childrenを使う場合は、全てのchildオブジェクトに共通した操作を与えたい場合などがよいでしょう。

類似したメソッドとして「traverse」があります。


##### parent[読み書き可能]《Object3D》

自身の親オブジェクトを返します。

基本的にObject3Dを継承したクラスのインスタンスが返されますが、例外として最上位のSceneオブジェクト
親オブジェクトにはparentが無いため、その場合はnilが返されます。

##### position[読み書き可能]《Vector3》

当該オブジェクトのローカル座標系における現在位置をVector3インスタンスとして返します。

Vector3インスタンスは、各次元の値をそれぞれ「x」「y」「z」という属性としてアクセスする
ことができるので、

```
p obj.position.x   #=> objのX座標値を返す
obj.position.x = 5 #=> oobjのX座標を5に設定する
```

のように扱うこともできれば、

```
obj.position = Mittsu::Vector3.new(5, 0, 0) #=> objの3次元座標を、x: 5, y: 0, z: 0 に設定する
```

のように扱うこともできます。

3つの値をまとめて変更する場合、上記の他にVector3のsetメソッドを使って、

```
obj.position.set(5, 0, 0) #=> objの3次元座標を、x: 5, y: 0, z: 0 に設定する
```

のように書くこともでき、多少タイプ量を減らすことも可能です。

##### rotation[読み書き可能]《Euler》

当該オブジェクトの回転状態をEuler（オイラー）インスタンスとして返します。

Eulerは、その名の通り「オイラー角」を表現するクラスです。

EulerインスタンスもVector3同様、各次元の値をそれぞれ「x」「y」「z」という属性としてアクセスする
ことができます。


##### scale[読み書き可能]《Vector3》

当該オブジェクトの拡大率をVector3インスタンスとして返します。

positionと同様、各次元の値をそれぞれ「x」「y」「z」という属性としてアクセスする
ことができます。

各次元毎の数値は、そのまま拡大率を意味します。

1.0でオリジナルサイズ、0.5で1/2、2.0で2倍のサイズという意味になります。

----

## 主要なメソッド

Object3Dクラスには多種多様なメソッドが用意されていますが、本稿ではRuby合宿2022において必要となる
最小限のメソッドを紹介します。

※ 以下、断りのない限り全てインスタンスメソッドになります。

##### add

```
add(Object3D, ...) -> self
```

引数で渡されたObject3Dを継承するオブジェクトを自身のchildオブジェクトとして登録します。

これにより、childオブジェクトとして登録された3Dオブジェクトは、親オブジェクトの
位置・回転・サイズ変化に連動するようになります。

引数は任意の個数与えることができます。

##### remove

```
remove(Object3D, ...) -> nil
```

引数で渡されたObject3Dを継承するインスタンスを自身のchildren（childオブジェクトの集合）
から除去します。

除去されたオブジェクトは、他に親を持たない限り、レンダリング対象から外れることになります。

引数は任意の個数与えることができます。

##### get_object_by_property

```
get_object_by_property(name, value) -> Object3D
```

引数「name」で渡された属性名が引数「value」であるchildオブジェクトを取得します。

本メソッドの簡易表記的なメソッドとして、「get_object_by_id(value)」「get_object_by_name(value)」
があります。

それぞれ挙動としては、

```
get_object_by_property(:id, value) -> Object3D
get_object_by_property(:name, value) -> Object3D
```

と同じ意味になります。

##### rotate_＊【＊: x, y, zのいずれか】

```
rotate_x(degree) -> self
rotate_y(degree) -> self
rotate_z(degree) -> self
```

X軸、Y軸、Z軸それぞれに用意されているオブジェクトの軸回転用メソッドです。

オブジェクトのローカル座標系における各軸を中心として回転します。

従って、

```ruby
obj.rotation.x += 0.05
obj.rotation.y += 0.05
```

と、

```ruby
obj.rotation_x(0.05)
obj.rotation_y(0.05)
```

は、同じ方向に同じ角度ずつ回転を行っていることになりますが、振る舞いが異なります。


##### translate_＊【＊: x, y, zのいずれか】

```
translate_x(distance) -> self
translate_y(distance) -> self
translate_z(distance) -> self
```

X軸、Y軸、Z軸それぞれに用意されているオブジェクトの移動用メソッドです。

各軸に対して、指定された距離分移動を行います。

##### look_at

```
look_at(Vector3) -> Quaternion
```

当該オブジェクトの注視点を、引数で与えられたVector3オブジェクトの座標に設定します。

このメソッドがよく利用されるのは、カメラオブジェクトを常に特定の対象に向け続ける用途など
が挙げられます。

例えば、「3D空間の原点（[0, 0, 0]）に配置されたオブジェクトに対し、
カメラをY軸周りに真円軌道で（一定の距離を保って）周回させ、カメラ方向を常に原点に向け続ける」
というコードは、以下のように書けます（レンダリング部分のみ抜粋）。

```ruby
distance = 5.0
x = 0
renderer.window.run do
  # カメラそのものの座標を真円軌道で動かす
  camera.position.x = distance * Math.sin(x * 0.01)
  camera.position.z = distance * Math.cos(x * 0.01)
  
  # 移動後のカメラ位置から、原点（[0, 0, 0]）を注視し直す
  camera.look_at(Mittsu::Vector3.new(0, 0, 0))

  # 1フレーム分レンダリング
  renderer.render(scene, camera)

  # カメラ位置の基本座標を移動する
  x += 1
end
```

※ 本メソッドでは回転情報を示すクォータニオン（Quaternion）オブジェクトを返してきますが、
これを何かに利用することはあまり無いと思われるので、本合宿では無視するようにしてください。

##### get_world_position

```
get_world_position -> Vector3
```

当該オブジェクトのワールド座標系における現在位置を返します。

##### get_world_rotation

```
get_world_rotation -> Euler
```

当該オブジェクトのワールド座標系における現在の回転情報を返します。

##### get_world_scale

```
get_world_scale -> Vector3
```

当該オブジェクトのワールド座標系における現在のスケール情報を返します。

##### get_world_direction

```
get_world_direction -> Vector3
```

当該オブジェクトのワールド座標系における現在の注視点（向いている方向）を返します。

##### traverse

```
traverse(&block) -> nil
```

当該オブジェクトの全ての子孫オブジェクトをフラット化して列挙し、与えられたブロックに代入して実行します。

注意点としては、当該オブジェクトそのものも列挙の対象になるという点になります。

また、全ての「子孫」が列挙されるので、当該オブジェクトXの直接の子オブジェクトAが更に子オブジェクトB, Cを持っていた
場合、列挙されるオブジェクトは「X, A, B, C」の4つになります。


##### clone

```
clone -> Object3D
```

当該オブジェクトを複製し、複製された新しいオブジェクトを返します。

同じ形のオブジェクトを大量生産する場合などに便利です。
