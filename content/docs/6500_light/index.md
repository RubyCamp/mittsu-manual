---
title: 'ライト'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 6500
summary: 3D空間であるシーンを照らすライト（光源）について説明します。
---

## 3D空間と光源（ライト）

何も追加しなければ、3D空間（シーン）は実は真っ暗闇です。

物体を追加しても、光源（ライト）が無ければレンダリング結果は真っ暗になります。

一部、Mittsu::MeshBasicMaterialのように、光源の影響を受けないため光源無しでも見えるマテリアルもありますが、
基本的には物体は光源無しにはレンダリングされません。

陰影の付いた3D画像は、物体と光源が合わさることで初めてレンダリングされるのです。

## ライトの種類

Mittsuでは、3D空間を照らす光源として、6種類のライトが用意されています。

本節では、その内Ruby合宿2022で使用する主な4種類について説明します。

代表的なライトであるMittsu::DirectionalLightを用いて、平面と立方体を組み合わせた構成で光源の
見え方を確認するプログラムを以下のように作成します。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

light = Mittsu::DirectionalLight.new(0xffffff)
light.position.set(1, 2, 1)
scene.add(light)

geom_box = Mittsu::BoxGeometry.new(0.5, 0.5, 0.5, 4, 4, 4)
mat_box = Mittsu::MeshPhongMaterial.new(color: 0x00ff00)
mesh_box = Mittsu::Mesh.new(geom_box, mat_box)
mesh_box.position.z = -1.5
scene.add(mesh_box)

geom_plane = Mittsu::PlaneGeometry.new(3, 3)
mat_plane = Mittsu::MeshPhongMaterial.new(color: 0x00ff00)
mesh_plane = Mittsu::Mesh.new(geom_plane, mat_plane)
mesh_plane.position.z = -3
mesh_plane.position.y = -1
mesh_plane.rotation.x = -Math::PI / 2
scene.add(mesh_plane)


renderer.window.run do
	mesh_box.rotation.x += 0.05
	mesh_box.rotation.y += 0.05
	renderer.render(scene, camera)
end
```

これを実行すると、以下のようになります。

{{<
    figure src="/images/6500_light/fig_1.gif"
    class="center" width="640" height="480"
>}}


以下、それぞれのライトについて使い方を見ていきましょう。

##### DirectionalLight

特定の方向からシーンを照らす平行光源です。

無限遠から照らしているものと扱われるため、物体に当たる光線は平行に当たります。

initializeメソッドは以下のようになります。

```ruby
Mittsu::DirectionalLight.new(color = nil, intensity = 1.0) #-> DirectionalLight
```

| 引数 | 意味 |
|------|------|
| color | 光源の色（光の色） |
| intensity | 光源の強さ |

前掲のサンプルのような見え方をします。

無限遠からの平行光源ですので、陰影を伴ってシーン全体を照らしたりする用途に向いていると思います。

##### AmbientLight

環境光（拡散環境光）を発する光源です。

シーン全体を均等に照らすような用途に用いると良いでしょう。

ただし、陰影を付けたりする用途にはあまり向かないかもしれません。

initializeメソッドは以下のようになります。

```ruby
Mittsu::AmbientLight.new(color) #-> AmbientLight
```

| 引数 | 意味 |
|------|------|
| color | 光源の色（光の色） |

サンプルのライトをAmbientLightに変更すると、以下のように見えます。

{{<
figure src="/images/6500_light/fig_2.gif"
class="center" width="640" height="480"
>}}

##### SpotLight

その名の通り「スポットライト」として用いる光源です。

スポットライトは、ある一定方向にしか光を発しませんので、向きや照らす距離、
ボカシや減衰率など細かなパラメータ調整ができる半面、思った通りの場所を照らすのに
多少慣れが必要なライトですが、その分リアルな陰影を表現しやすいライトでもあります。

initializeメソッドは以下のようになります。

```ruby
Mittsu::SpotLight.new(
  color = nil, intensity = 1.0, distance = 0.0,
  angle = (::Math::PI / 3.0), exponent = 10.0, decay = 1.0) #-> SpotLight
```

| 引数 | 意味 |
|------|------|
| color | 光源の色（光の色） |
| intensity | 光源の強さ |
| distance | ライトが照らす距離 |
| angle | 光源の角度 |
| exponent | ボカシ具合 |
| decay | 減衰率 |

サンプルのライトをSpotLightに変更すると、以下のように見えます。

{{<
figure src="/images/6500_light/fig_3.gif"
class="center" width="640" height="480"
>}}

##### PointLight

ポイントライトは、スポットライトと似ていますが、設置した位置から全方位に対して
光を発する点がスポットライトと異なります。

全ての方位に光を放ってくれるので、扱いやすい反面絵としてはのっぺりとした印象に
なりやすい特徴があります。

initializeメソッドは以下のようになります。

```ruby
Mittsu::PointLight.new(
	color = nil, intensity = 1.0,
	distance = 0.0, decay = 1.0) #-> PointLight
```

| 引数 | 意味 |
|------|------|
| color | 光源の色（光の色） |
| intensity | 光源の強さ |
| distance | ライトが照らす距離 |
| decay | 減衰率 |

サンプルのライトをPointLightに変更すると、以下のように見えます。

{{<
figure src="/images/6500_light/fig_4.gif"
class="center" width="640" height="480"
>}}

##### AreaLight

エリアライトは、

initializeメソッドは以下のようになります。

```ruby
Mittsu::AreaLight.new(color = nil, intensity = 1.0) #-> AreaLight
```

| 引数 | 意味 |
|------|------|
| color | 光源の色（光の色） |
| intensity | 光源の強さ |

サンプルのライトをAreaLightに変更すると、以下のように見えます。

{{<
figure src="/images/6500_light/fig_4.gif"
class="center" width="640" height="480"
>}}

##### PointLight

ポイントライトは、スポットライトと似ていますが、設置した位置から全方位に対して
光を発する点がスポットライトと異なります。

全ての方位に光を放ってくれるので、扱いやすい反面絵としてはのっぺりとした印象に
なりやすい特徴があります。

initializeメソッドは以下のようになります。

```ruby
Mittsu::PointLight.new(
	color = nil, intensity = 1.0,
    distance = 0.0, decay = 1.0) #-> PointLight
```

| 引数 | 意味 |
|------|------|
| color | 光源の色（光の色） |
| intensity | 光源の強さ |
| distance | ライトが照らす距離 |
| decay | 減衰率 |

サンプルのライトをPointLightに変更すると、以下のように見えます。

{{<
    figure src="/images/6500_light/fig_4.gif"
    class="center" width="640" height="480"
>}}

## ライトの操作

ライトもまたメッシュやカメラと同じくObject3Dクラスを継承しています。

従って、メッシュなどと同様の手順で任意の位置に移動させたり回転させたりすることが可能です。

例えば、以下のようなプログラムを例にとってみましょう。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

light = Mittsu::PointLight.new(0xffffff)
scene.add(light)

geom_box = Mittsu::BoxGeometry.new(0.5, 0.5, 0.5, 4, 4, 4)
mat_box = Mittsu::MeshPhongMaterial.new(color: 0x00ff00)
mesh_box = Mittsu::Mesh.new(geom_box, mat_box)
mesh_box.position.z = -1.5
scene.add(mesh_box)

r = 1
theta = 0
renderer.window.run do
  light.position.x = r * Math.cos(theta)
  light.position.z = r * Math.sin(theta)
  renderer.render(scene, camera)
  theta += 0.05
end
```

これを実行すると、

{{<
    figure src="/images/6500_light/fig_5.gif"
    class="center" width="640" height="480"
>}}

このように、ポイントライトがX-Z平面上を半径1単位で周回する様子が見て取れます。


## 物体の影

現実の物体は、光を受けると影ができます。

しかし、これまで見てきたサンプルでは、

{{<
    figure src="/images/6500_light/fig_3.gif"
    class="center" width="640" height="480"
>}}

のように、陰影を付けることはできても、立方体の影が平面上に落ちることはありませんでした。

Mittsuでは、物体の影を別の物体に落とすためには、光源と物体、そしてレンダラーそのものに対して、
それぞれ影を生成するための設定をしてやらないと、影を作ることができません。

Ruby合宿2022では、影の生成までは範囲外としていますが、一例として以下に立方体の影を
平面に落とすためのサンプルを掲載しておきます。

興味があれば、サンプルのコメントなどを参考に自分でもやってみてください。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

# レンダラーのシャドウマップを有効にし、ソフトシャドウに設定しておく
renderer.shadow_map_enabled = true
renderer.shadow_map_type = Mittsu::PCFSoftShadowMap

light = Mittsu::SpotLight.new(0xffffff, 1.0)
light.position.set(1, 2, 1)

# ライトが影を生成するようにする
light.cast_shadow = true

# ライトが生成する影のパラメータを設定する
light.shadow_darkness = 0.5     # 影の暗さを設定（あまり暗すぎても不自然なので半分程度に）
light.shadow_map_width = 1024   # シャドウマップのサイズ（横幅）を定義
light.shadow_map_height = 1024  # シャドウマップのサイズ（縦幅）を定義
light.shadow_camera_near = 1.0  # 影とカメラのクリッピング距離（近端）を設定
light.shadow_camera_far = 100.0 # 影とカメラのクリッピング距離（遠端）を設定
light.shadow_camera_fov = 75.0  # 影の撮影画角を設定（基本的にはカメラのFOVに合わせておくのが吉）
scene.add(light)

geom_box = Mittsu::BoxGeometry.new(0.5, 0.5, 0.5, 4, 4, 4)
mat_box = Mittsu::MeshPhongMaterial.new(color: 0x00ff00)
mesh_box = Mittsu::Mesh.new(geom_box, mat_box)
mesh_box.position.z = -1.5

# ライトからの光を受けた立方体メッシュが影を生成するように設定
mesh_box.cast_shadow = true

scene.add(mesh_box)

geom_plane = Mittsu::PlaneGeometry.new(3, 3)
mat_plane = Mittsu::MeshPhongMaterial.new(color: 0x00ff00)
mesh_plane = Mittsu::Mesh.new(geom_plane, mat_plane)
mesh_plane.position.z = -3
mesh_plane.position.y = -1
mesh_plane.rotation.x = -Math::PI / 2

# 平面メッシュが影を受け取るように設定
mesh_plane.receive_shadow = true

scene.add(mesh_plane)


renderer.window.run do
  mesh_box.rotation.x += 0.05
  mesh_box.rotation.y += 0.05
  renderer.render(scene, camera)
end
```

結構色々設定が必要ですね。

このサンプルを実行すると、以下のようになります。

{{<
    figure src="/images/6500_light/fig_6.gif"
    class="center" width="640" height="480"
>}}

このように、影については関連する光源と「影を作る物体」「影を受ける物体」を設定してやる
必要があります。

なんでデフォルトで全ての物体が影を作ったり受けたりできるようになっていないのかと言えば、
一般的に3Dグラフィクスのレンダリングにおいて、影の生成というのは比較的コストの高い処理
であるため、という理由が大きいでしょう。

今回くらいのサンプル程度であれば大したコストではありませんが、3D空間に登場する物体が
増えれば増えるほど、影を生成するコストは無視できないものになっていきます。

なので、デフォルトでは影は落とさないようにしておいて、影が必要な物体だけに設定する方が
コストが抑えやすいということは言えるでしょう。
