---
title: 'メッシュ（物体）'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 6000
summary: ジオメトリとマテリアルから3D空間内に登録されるメッシュ（物体）を作ります。
---

## メッシュ（Mesh）とは

メッシュとは、これまで説明してきた「ジオメトリ」（形状）と「マテリアル」（質感）
に基づいて、3次元空間に実際に「位置」を伴って配置されるオブジェクトになります。

メッシュ（Mesh）クラスはObject3dクラスの子クラスであり、Object3dが持つ全てのメソッドが利用可能です。

基本的なメッシュの作り方は、これまでも何度か出てきたように、

```ruby
geometry = Mittsu::SphereGeometry.new(1, 32, 32)
material = Mittsu::MeshBasicMaterial.new(color: 0xff0000)
mesh = Mittsu::Mesh.new(geometry, material)
```

という形式が一般的です。
ジオメトリ・マテリアルの生成はそれぞれ作りたい物体に応じて適宜変更されますが、

```ruby
mesh = Mittsu::Mesh.new(geometry, material)
```

の部分が変わることは無いでしょう。

上記のように作成されたメッシュは、最終的に、

```ruby
scene.add(mesh)
```

のように、シーンオブジェクトに追加されることで、3次元空間に登録されカメラによる撮影対象に加わります。

なお、生成されたばかりのメッシュ（カメラもですが）オブジェクトの初期位置は、
ワールド座標系の原点（[0, 0, 0]）になります。

すなわち、単純に生成して追加しただけだと、カメラもメッシュも同一座標に重なって登録されてしまう
ことになります。

これでは実行しても何も表示されませんので、カメラかメッシュのどちらかの位置を変更しなければなりません。

カメラの初期状態における向きは、-Z方向になります。

つまり、カメラは-Z方向に向いてワールド座標の原点に位置しているわけですので、
最も少ないコードでメッシュをカメラの撮影範囲に移動するとすれば、ワールド座標系において、-Z方向に
数単位ほど移動させればよさそうですね。

{{<
    figure src="/images/2000_scene/fig_2.png"
    class="center" width="640" height="480"
>}}

上図のように、-Z方向に5単位ほど移動させるコードは、以下のように記述できます。

```ruby
mesh.position.z = -5
```

このコードは、前述の「scene.add(mesh)」の前に書いても後に書いても、どちらでも問題ありません。

実行結果は以下のようになります。

{{<
    figure src="/images/6000_mesh/fig_1.png"
    class="center" width="640" height="480"
>}}

後は、同じ理屈でX軸方向やY軸方向も含めて、好きな位置にメッシュを移動させることができます。

##### アニメーション

アニメーションといっても、キャラクタが生き物のように動くようなものは現時点（2022/02月）の
Mittsuのバージョンでは非常に困難です。

そのため、Ruby合宿2022では、いわゆる「ボーン・アニメーション」のような複雑なアニメーションは
扱いません。

ここまで紹介してきた基本的なジオメトリ・マテリアルを用いたメッシュ全体の位置や回転・サイズ
などを変化させることでアニメーションを表現していきましょう。

例えば、以下のように、X-Z平面上で真円軌道を描いて運動する球体を表現する
アニメーションはどのように実装すればよいでしょうか？

{{<
    figure src="/images/6000_mesh/fig_2.gif"
    class="center" width="640" height="480"
>}}

実装例の全文は次のようになります。

※ 見た目に陰影がついていた方が分かりやすいので、MeshPhongMaterialを用いて陰影を付けています。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

directionalLight = Mittsu::DirectionalLight.new(0xffffff);
directionalLight.position.set(1, 1, 1)
scene.add(directionalLight)

geometry = Mittsu::SphereGeometry.new(1, 32, 32)
material = Mittsu::MeshPhongMaterial.new(color: 0xff0000)
mesh = Mittsu::Mesh.new(geometry, material)
scene.add(mesh)

r = 5
theta = 0

renderer.window.run do
  mesh.position.x = r * Math.cos(theta)
  mesh.position.z = r * Math.sin(theta) - 7
  renderer.render(scene, camera)
  theta += 0.05
end
```

これは非常に簡単ですね。

3次元空間を真上から眺めた座標系（X-Z平面）を考え、
以下のように原点を中心とした半径rの時計回り円運動を考えれば、
任意の角度θにおける（メッシュが取るべき）X座標・Z座標の値「x」「z」は、それぞれ、

```azure
x = r * cos(θ)
z = r * sin(θ)
```

となるので、後はカメラにちょうどいい感じに収まるよう、-Z方向に半径rより若干大き目にオフセットしてやればよいということになります。

{{<
    figure src="/images/6000_mesh/fig_3.png"
    class="center" width="800" height="600"
>}}

このように、

```ruby
renderer.window.run do
  # この辺りでシーン内のオブジェクトの状態を1フレーム分変化させる
  renderer.render(scene, camera)
  # この辺りでも可。先にフレームを描画してから変化させる場合はこちら。
end
```

というメインループの中で、1フレーム毎にシーン内のオブジェクトの状態を微妙に変えていくことで、
アニメーションが表現可能となります。

## メッシュのグルーピング

複数のメッシュを言わば「くっつけ」て、一つのメッシュのように扱うことも可能です。

例えば、以下のように立方体に円錐を横方向に結合させた物体を作ってみましょう。

{{<
    figure src="/images/6000_mesh/fig_4.png"
    class="center" width="800" height="600"
>}}

###### 親オブジェクトの作成

まず、シンプルに緑色をした立方体のメッシュを作成します。

このメッシュを「mesh_body」と名付けます。

```ruby
geom_body = Mittsu::BoxGeometry.new(1, 1, 1, 4, 4, 4)
mat_body = Mittsu::MeshLambertMaterial.new(color: 0x00ff00)
mesh_body = Mittsu::Mesh.new(geom_body, mat_body)
```

###### 子オブジェクトの作成

そして、続いて青色をした円錐のメッシュを作成します。

円錐は、立方体の1面の中に収めたいので、半径を0.5としています
（長さについては適当ですが、とりあえず0.5としましょう）。

このメッシュを「mesh_lenz」と名付けましょう（何となく、ビデオカメラのような感じの見た目を作るため）。

```ruby
geom_lenz = Mittsu::CylinderGeometry.new(0.5, 0.5, 0.5)
mat_lenz = Mittsu::MeshLambertMaterial.new(color: 0x0000ff)
mesh_lenz = Mittsu::Mesh.new(geom_lenz, mat_lenz)
```

###### オブジェクトの親子関係の設定

こうして作成した2つのメッシュを、立方体の方を親として、入れ子関係にします。

```ruby
mesh_body.add(mesh_lenz)
```

これにより、mesh_lenzはmesh_bodyの子オブジェクト（child）となり、
親のワールド座標系の変化に追随するようになります。

###### 親オブジェクトをシーンに登録

そして、親子関係を設定したmesh_bodyを、シーンに追加します。

ついでに、原点に置いたカメラに映るよう、mesh_bodyの位置を-Z方向に若干ずらしておきましょう。

```ruby
scene.add(mesh_body)
mesh_body.position.z = -5
```

###### 子オブジェクトが従う座標系

しかし、これだけでは実行しても立方体が表示されるのみです。

これは、少々直感的に把握しづらいかも知れませんが、mesh_bodyのchildになったmesh_lenzが、
mesh_bodyのローカル座標系における原点に位置している…つまり、mesh_bodyと完全に重なっているためです。

mesh_lenzはmesh_bodyより小さく作っていますので、これではmesh_bodyの中に埋もれてしまいます。

以下のようにして、mesh_bodyの一面に（横向きに）張り付くように移動させてみましょう。

```ruby
mesh_lenz.position.x = -0.5          # -X方向に0.5移動（高さを0.5にしているため）
mesh_lenz.rotation.z = Math::PI / 2  # Z軸を中心にπ/2（90度）回転
```

これで、前述の図のような形状に変化します。

ここでポイントとなるのは、mesh_bodyの子（child）となったmesh_lenzが属している座標系です。

mesh_lenzは、上記の通りシーン全体のワールド座標ではなく、mesh_bodyのローカル座標に基づいて移動や回転していることが見て取れます。

このように、あるオブジェクトに入れ子になった子オブジェクトは、親のローカル座標系に基づいて移動・回転・スケールするようになります。

そして、親であるmesh_bodyが何故ワールド座標に従うかと言えば、これも同じ理屈で、mesh_bodyから見た親オブジェクトが
シーン（scene）そのものになるためです。

つまり、ワールド座標とは、シーンオブジェクトのローカル座標系と言い換えることもできるわけです。

###### 結合したまま扱えることの確認

最後に、親オブジェクトであるmesh_bodyだけを移動させ、子オブジェクトが相対的な位置関係を保ったまま追随することを確認しましょう。

以下のコードで、mesh_bodyを円周軌道で周回させつつ、X・Y軸に沿って回転させてみます。

```ruby
r = 5
theta = 0
renderer.window.run do
  mesh_body.position.x = r * Math.cos(theta)
  mesh_body.position.z = r * Math.sin(theta) - 7
  mesh_body.rotate_x(0.05)
  mesh_body.rotate_y(0.05)
  renderer.render(scene, camera)
  theta += 0.03
end
```

実行すると以下のように挙動します。

{{<
    figure src="/images/6000_mesh/fig_5.gif"
    class="center" width="800" height="600"
>}}

親オブジェクトであるmesh_bodyの位置・回転情報が変化しても、子であるmesh_lenzは何もしていないのにちゃんと追随して位置関係を保っていますね。

もちろん、親オブジェクトを任意に動かしながら、子オブジェクトも動かすこともできます。

この入れ子構造を活用することで、基本的な形状だけでも結構複雑な物体を表現することができるようになります。


##### オブジェクトグループ(Mittsu::Group)について

前述の例では、普通のメッシュであるmesh_bodyを親として直接シーンに追加しましたが、
複数のオブジェクトをグルーピングする場合、不可視の親オブジェクトを作って、
それに子として関連オブジェクトを紐づけるような使い方をしたいこともあります。

そんな時に使うのが、Mittsu::Groupクラスになります。

Mittsu::Groupを使って、本体（mesh_body）とレンズ（mesh_lenz）を備えた「video_camera」オブジェクトを
定義するよう、前記の例を少し書き換えてみましょう。

以下がその全文になります。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

directionalLight = Mittsu::DirectionalLight.new(0xffffff);
directionalLight.position.set(1, 1, 1)
scene.add(directionalLight)

geom_body = Mittsu::BoxGeometry.new(1, 1, 1, 4, 4, 4)
mat_body = Mittsu::MeshLambertMaterial.new(color: 0x00ff00)
mesh_body = Mittsu::Mesh.new(geom_body, mat_body)

geom_lenz = Mittsu::CylinderGeometry.new(0.5, 0.5, 0.5)
mat_lenz = Mittsu::MeshLambertMaterial.new(color: 0x0000ff)
mesh_lenz = Mittsu::Mesh.new(geom_lenz, mat_lenz)

video_camera = Mittsu::Group.new
video_camera.add(mesh_body)
video_camera.add(mesh_lenz)

scene.add(video_camera)

video_camera.position.z = -5
mesh_lenz.position.x = -0.5
mesh_lenz.rotation.z = Math::PI / 2

r = 5
theta = 0
renderer.window.run do
  video_camera.position.x = r * Math.cos(theta)
  video_camera.position.z = r * Math.sin(theta) - 7
  video_camera.rotate_x(0.05)
  video_camera.rotate_y(0.05)
  renderer.render(scene, camera)
  theta += 0.03
end
```

mesh_bodyを直接シーンに追加するよりも、やや可読性が上がったような気がしますね。

このように、Mittsu::Groupのインスタンスそのものはレンダリング対象になりませんので、
複数のオブジェクトを束ねる用途に使うと便利です。

Mittsu::Groupでは、Object3Dクラスを継承しているクラスのインスタンスであれば何でも
追加できますので、メッシュだけではなく光源（ライト）などもまとめることも可能です。
