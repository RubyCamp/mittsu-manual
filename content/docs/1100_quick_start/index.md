---
title: 'クイックスタートガイド'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 1100
summary: Mittsuの基本的な使い方を理解しよう。
---

## 最もシンプルなMittsuプログラム

以下のプログラムを、「main.rb」等の好きなファイル名で保存し、実行してみましょう。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

renderer.window.run do
	renderer.render(scene, camera)
end
```

成功した場合、以下のようなウィンドウが現れます。

{{<
  figure src="/images/1100_quick_start/hello_mittsu.png"
  title="実行結果" class="center" width="640" height="480"
>}}

真っ黒で何も表示されていませんが、この真っ暗な画面こそがMittsuのSceneクラスが
提供する3D空間をカメラ（今回は「Mittsu::PerspectiveCamera」を使用）で撮影し、
レンダラー（今回は「Mittsu::OpenGLRenderer」を使用）でディスプレイ上のウィンドウ
に投影した結果になります。

真っ暗なのは、現在このカメラが映し出す範囲のシーン（3D空間）に、何の物体も存在
していないためです。

## 3D空間にキャラクタを登場させてみよう

いわば空っぽの宇宙であるこのシーン（の、カメラが映し出している領域）に、
登場人物に当たる何等かの3Dキャラクタを登場させてみましょう。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

geometry = Mittsu::BoxGeometry.new(1.0, 1.0, 1.0)
material = Mittsu::MeshBasicMaterial.new(color: 0x00ff00)
mesh = Mittsu::Mesh.new(geometry, material)
mesh.position.z = -5
scene.add(mesh)

renderer.window.run do
  renderer.render(scene, camera)
end
```

ここで新たに「Mittsu::BoxGeometry」「Mittsu::MeshBasicMaterial」「Mittsu::Mesh」という
クラスが登場しました。
これらを本ドキュメントではそれぞれ、

* 「ジオメトリ」（Geometry: 形状）
* 「マテリアル」（Material: 質感）
* 「メッシュ」（Mesh: 物体）

と呼称します。

ざっくり説明すると、3D空間に位置情報を伴って表示されるのが「メッシュ」であり、
そのメッシュの見た目の形を決めるのが「ジオメトリ」。

そして、メッシュがどういった材質でできているか（質感）を表現するのが「マテリアル」
となります。

今回は、BoxGeometryを用いて立方体の形状をしたメッシュを、MeshBasicMaterial
（標準的な単色マテリアル）で緑色に塗り潰してメッシュ化しました。

そして、生成されたメッシュの3D空間内における位置情報（X, Y, Zの3次元座標で表される）
として、原点（[0, 0, 0]）からZ方向に-5だけ移動させて、シーンに追加しました。

この最後のシーンへの追加を行わないと、メッシュは3D空間内に出現しませんので注意が必要です。

{{<
  figure src="/images/1100_quick_start/fig_1.png"
  title="実行結果" class="center" width="640" height="480"
>}}

実行結果はこのようになります。

カメラは、追加した時点では原点から-Z方向に向いて撮影していますので、原点からZ方向に-5
移動した立方体は、御覧のように正方形のように見えます。

## キャラクタを回転させてみよう

これだと、あまり3D感がありませんので、この緑色をした立方体を回転させてみましょう。

現在、

```ruby
renderer.window.run do
  renderer.render(scene, camera)
end
```

となっている部分を、

```ruby
renderer.window.run do
  mesh.rotation.x += 0.05
  renderer.render(scene, camera)
end
```

のようにしてみましょう。

追加したコードは、メッシュに対して「X軸に沿って0.05だけ回転角を増やせ」というものです。

これを、「renderer.window.run」メソッドのブロック（do～endの間）に記述すると、
同メソッドが1フレーム描画する毎（デフォルトでは1秒間に60フレーム描画される）に
毎回実行してくれます。

つまり、このコードはX軸時計回り方向にわずかに回転する運動を1フレーム毎に繰り返すことで、
スムーズに回転しているように見えるはずです。

それでは、実行結果を見てみましょう。

{{<
  figure src="/images/1100_quick_start/fig_2.gif"
  title="実行結果" class="center" width="640" height="480"
>}}

確かに期待通りに動作していますね。

カメラは現在-Z方向を向いていますので、立方体が指定通りの向きに回転していることが分かります。

1フレームの回転角として現在は「0.05」を指定していますが、これを「-0.05」とすれば、回転
方向が反転しますし、数値を大きくすれば回転速度が増し、小さくすれば回転速度が遅くなります。

また、1フレームに複数の軸で回転を行うことで、複雑な回転も表現できます。

例えば、以下のようにY軸方向にも回すようにしてみましょう。

```ruby
renderer.window.run do
  mesh.rotation.x += 0.05
  mesh.rotation.y += 0.05
  renderer.render(scene, camera)
end
```

Y軸（縦軸）でも時計回りに回転するようになりますので、実行結果は以下のようになります。

{{<
  figure src="/images/1100_quick_start/fig_3.gif"
  title="実行結果" class="center" width="640" height="480"
>}}

