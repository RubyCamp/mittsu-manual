---
title: 'カメラ'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 7000
summary: 3D空間であるシーンを撮影し、レンダラーに表示するカットを得るためのカメラについて説明します。
---

## Mittsuにおけるカメラとは

Mittsuが提供する3D空間（シーン）に配置し、レンダラーに映像を供給するオブジェクトを
「カメラ」と呼称します。

2022年2月時点でMittsuが提供しているカメラの種類は、以下の3種類です。

| カメラの種類 | 概要 |
|--------------|------|
| Mittsu::PerspectiveCamera | 遠近感を表現でき、人間の視点に近い見え方をするカメラ |
| Mittsu::OrthographicCamera | 遠近感の無い平行投影用カメラ |
| Mittsu::CubeCamera | 環境マッピング用の特殊なカメラ |

Ruby合宿2022では、これらのカメラの内、「Mittsu::PerspectiveCamera」を使用します。

## カメラの作り方

カメラオブジェクトは、ここまでのサンプルでも常に作り続けていますが、initializeメソッドの仕様は
以下のようになります。

```ruby
Mittsu::PerspectiveCamera.new(fov = 50.0, aspect = 1.0, near = 0.1, far = 2000.0) #-> Mittsu::PerspectiveCamera
```

各パラメータの意味は以下の通りです。

| パラメータ | 意味 |
|------------|------|
| fov | Field of View（視野）の略で、文字通りカメラのレンズに収まる視界の広さを意味します |
| aspect | アスペクト比（画面の横幅と縦幅の比率）を意味します。ウィンドウサイズが「800x600」 ならば、800 / 600 = 1.333... となります |
| near | クリッピング距離（近端） |
| far | クリッピング距離（遠端） |

fovはデフォルトの50だと少々広すぎるように思う（個人の感想です）ので、75くらいで使うのが
無難ではないかと思われます。

あまり視野角を広く（値が小さいほど広い範囲を表示できます）しすぎると、
その分「歪み」も大きくなるためです。

aspectについては、ウィンドウサイズが決まれば自動的に決定可能ですので、
レンダラーの生成コードもまとめて、

```ruby
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
ASPECT = SCREEN_WIDTH / SCREEN_HEIGHT.to_f

scene = Mittsu::Scene.new
camera = Mittsu::PerspectiveCamera.new(75.0, ASPECT)
renderer = Mittsu::OpenGLRenderer.new width: SCREEN_WIDTH, height: SCREEN_HEIGHT, title: 'RubyCamp 2022'
```

のように定数化しておくと便利です。

クリッピング距離は多少直感的ではない概念かも知れませんが、カメラで映し出す範囲（距離）を制限する
機能です。

これは、無限に広がる3D空間に存在する全ての物体を全部カメラで撮影するのは、
レンダラーのパフォーマンスに大きな負担を及ぼす可能性があるため、カメラからの距離が
「near以上、far以下」であるオブジェクトしか映さないようにする機能です。

これにより、見えないほど遠距離にあるオブジェクトまで全部レンダリングしようとして無駄に
パフォーマンスを消費することが無くなり、レンダラーのパフォーマンスが向上できます。

nearを変更する必要はめったに生じないと思われますが、farは場合によっては変更した方が良い場合が
あるかもしれません。

## カメラの初期座標と向き

カメラは、

```ruby
renderer.window.run do
  renderer.render(scene, camera)
end
```

のように、renderメソッドにシーンとセットで渡されることにより、
当該シーンの1フレーム分の撮影を行います。

カメラオブジェクトが生成された時点の初期座標は、ワールド座標系の原点（[0, 0, 0]）であり、
向きは-Z方向に向いています。

カメラもまたメッシュなどと同様Object3Dクラスを継承していますので、メッシュなどと同じ感覚で
移動・回転が行えます。

例えば、以下のように3D空間で立方体が回転するアニメーションがあるとしましょう。

{{<
    figure src="/images/7000_camera/fig_1.gif"
    class="center" width="640" height="480"
>}}

この挙動を実現するには、2つの方法があり得ます。

1つ目は、メッシュを回転させる方法です。

全文を掲載すると以下のようになります。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

directionalLight = Mittsu::DirectionalLight.new(0xffffff);
directionalLight.position.set(1, 1, 1)
scene.add(directionalLight)

geometry = Mittsu::BoxGeometry.new(1, 1, 1, 4, 4, 4)
material = Mittsu::MeshLambertMaterial.new(color: 0x00ff00)
mesh = Mittsu::Mesh.new(geometry, material)
mesh.position.z = -2
scene.add(mesh)

renderer.window.run do
  mesh.rotate_z(0.05)
  renderer.render(scene, camera)
end
```

このプログラムでは、

```ruby
renderer.window.run do
	mesh.rotate_z(0.05)
	renderer.render(scene, camera)
end
```

のように、1フレーム毎にメッシュの方をZ軸に沿って回転させています。

これとは反対の発想で実現するもう1つのパターンが、「カメラの方を回転させる」というものです。

```ruby
renderer.window.run do
  camera.rotate_z(-0.05)
  renderer.render(scene, camera)
end
```

※ カメラの方が回ることになるので、同じ向きに回すためにはメッシュの場合と回転角が逆になります。

これでも同じ挙動が実現できます。

メッシュに対して行える操作は、基本的に全てカメラに対しても行えます。

この特徴を上手く活用することで、思わぬ効果を得ることもできるかも知れません。

## カメラの移動と向きの制御

カメラは、前述の通り回転もできれば移動もできます。

例えば、ワールド座標系の原点に立方体を配置し、立方体は動かさずにカメラの方を原点中心に半径rの
真円軌道で移動させると、立方体が回転しているように見えることになります。

しかし、その時1点問題があります。

それは、カメラの位置を変えても、カメラの向いている方向は変わらないということです。

そのため、カメラが真円軌道を角度Δθずつ移動する都度、カメラの向いている方向も微調整しなくては
なりません。

今回の場合は、立方体を回転しているように見せたいわけですから、カメラは常に原点方向を向き続ける
必要があります。

もちろん、Object3d#rotate_＊メソッドでカメラを正しい角度で回転させれば目的は達成できますが、
計算は少々面倒です。

そこで役に立つのが「Object3d#look_at」メソッドです。

look_atメソッドは、引数で与えられたMittsu::Vector3オブジェクトが示す座標にそのオブジェクトの
向きを向けてくれるメソッドですので、この場合は注視点となる原点（[0, 0, 0]）を「look_at」し
続ければよいわけです。

この考えに沿って実装したコード全文は以下になります。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

directionalLight1 = Mittsu::DirectionalLight.new(0xffffff);
directionalLight1.position.set(1, 1, 1)
scene.add(directionalLight1)

directionalLight2 = Mittsu::DirectionalLight.new(0xffffff);
directionalLight2.position.set(-1, -1, -1)
scene.add(directionalLight2)

geometry = Mittsu::BoxGeometry.new(1, 1, 1, 4, 4, 4)
material = Mittsu::MeshPhongMaterial.new(color: 0x00ff00)
mesh = Mittsu::Mesh.new(geometry, material)
scene.add(mesh)

r = 3
theta = 0
renderer.window.run do
  camera.position.x = r * Math.cos(theta)
  camera.position.z = r * Math.sin(theta)
  camera.look_at(Mittsu::Vector3.new(0, 0, 0))
  renderer.render(scene, camera)
  theta += 0.05
end
```

※ 回転する裏側も照らす必要があるため、平行光源を2つ追加しています。

実行結果は以下のようになります。

{{<
    figure src="/images/7000_camera/fig_2.gif"
    class="center" width="640" height="480"
>}}

オブジェクトを動かすか、カメラを動かすか。

あるいは、その両方を動かすか。

作るアプリケーションの性質に応じて検討するのが良いでしょう。
