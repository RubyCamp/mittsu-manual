---
title: 'マテリアル（質感）'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 5000
summary: シーンに登場する物体の質感や表面の色などを決定付けるマテリアルの種類と扱い方について説明します。
---

## マテリアル（Material）とは

マテリアルとは、3D空間における物体の見え方を決定付ける「材質」を意味します。

例えば、以下の3つは同じ立方体(BoxGeometry)ですが、それぞれ表面の色をマテリアルの属性によって変更して、別の色の球体にしています。

{{<
    figure src="/images/5000_material/fig_1.png"
    class="center" width="640" height="480"
>}}

※ 上記サンプルは、後述する「ライト」（平行光源）をシーンに1つだけ追加してライトアップしてあります。

## マテリアルの種類

例として、1つの球体を赤色で表示するプログラムを使い、マテリアルの種類による違いを見てみましょう。

ソースコード全文は(少々長いですが)以下のようになります。

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
material = Mittsu::MeshBasicMaterial.new(color: 0xff0000)
mesh = Mittsu::Mesh.new(geometry, material)
mesh.position.z = -3
scene.add(mesh)

renderer.window.run do
	renderer.render(scene, camera)
end
```

この内、マテリアルを作成しているのは、

```ruby
material = Mittsu::MeshBasicMaterial.new(color: 0xff0000)
```

の部分になります。

このマテリアルを、

```ruby
mesh = Mittsu::Mesh.new(geometry, material)
```

のように、マテリアルを適用する対象ジオメトリ「g」とペアにしてメッシュ（後述）化することで、
赤い表面を持つ球体を作り出しています。

そして、その球体に対して、

```ruby
directionalLight = Mittsu::DirectionalLight.new(0xffffff);
directionalLight.position.set(1, 1, 1)
```

で作成し、原点の若干斜め上に配置した平行光源で照らしています。

このサンプルを用いて、マテリアルの種類を確認していきましょう。

マテリアルの種類は、以下に紹介したもの以外にも存在しますが、
本マニュアルではRuby合宿2022において使用するものに限定して紹介します。

##### MeshBasicMaterial

最もシンプルなマテリアルです。

光源による陰影を考慮しないフラットな表現になるため、色による塗り潰しが均一になるのが特徴です。

サンプルそのままで実行可能ですので、実行してみると以下のようになります。

{{<
    figure src="/images/5000_material/fig_2.png"
    class="center" width="640" height="480"
>}}

##### MeshLambertMaterial

クラス名の通り、ランバートシェーディングという技法で材質を表現するマテリアルです。

MeshBasicMaterialと違い、光源による陰影を反映しますので、立体感が出ます。

質感としては光沢に欠ける均質な質感となります。

{{<
figure src="/images/5000_material/fig_3.png"
class="center" width="640" height="480"
>}}

##### MeshPhongMaterial

フォンシェーディングという技法で材質を表現するマテリアルです。

MeshLambertMaterialと同様に、光源による陰影を反映しますので、立体感が出ます。

質感としては光沢のあるやや金属的な質感となります。

{{<
    figure src="/images/5000_material/fig_4.png"
    class="center" width="640" height="480"
>}}


## マテリアルのオプション

各マテリアルについて、initializeメソッドで受け渡せる主なオプションは以下のようになります（Ruby合宿2022において使用を想定するものに限定しています）。

| オプション名 | 意味 |
|--------------|------|
| color        | マテリアルの色を16進数6桁で表現します。RGBそれぞれ2桁ずつとなります |
| map      | テクスチャマップ（後述）を指定します（Mittsu::Texture） |
| normalMap | ノーマルマップ（後述）を指定します（Mittsu::Texture） |
| wireframe | ワイヤーフレーム表示のON/OFFを制御します（boolean） |

## テクスチャマッピング

マテリアルは、colorオプションで色を指定する以外にも、テクスチャマッピングという技法により
任意の画像を貼り付けることもできます。

テクスチャは「Mittsu::Texture」クラスのインスタンスとして、
画像ファイル（PNGなどのフォーマット）を読み込んで、マテリアルの「map」オプションなどに
指定して反映させます。

たとえば、以下の画像はMittsuのサンプルに同梱されている地球の表面を表すテクスチャです。

{{<
    figure src="/images/5000_material/earth.png"
    class="center" width="640" height="480"
>}}

この画像を、サンプルのマテリアルに適用し、言わば画像で球体を「包み込む」ように貼り付けてみましょう。

ソースコードは以下のようになります(立体感が無いと寂しいので、マテリアルは「MeshLambertMaterial」を使います)。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

directionalLight = Mittsu::DirectionalLight.new(0xffffff);
directionalLight.position.set(1, 1, 1)
scene.add(directionalLight)

texture_map = Mittsu::ImageUtils.load_texture("./earth.png")

geometry = Mittsu::SphereGeometry.new(1, 32, 32)
material = Mittsu::MeshLambertMaterial.new(map: texture_map)
mesh = Mittsu::Mesh.new(geometry, material)
mesh.position.z = -2
scene.add(mesh)

renderer.window.run do
  renderer.render(scene, camera)
end
```

画像ファイル「earth.png」は、上記ソースコードと同じフォルダに保存しておいてください。

実行すると、以下のようになります。

{{<
    figure src="/images/5000_material/fig_5.png"
    class="center" width="640" height="480"
>}}

テクスチャマッピング用の画像は、

```ruby
texture_map = Mittsu::ImageUtils.load_texture("./earth.png")
```

として読み込んでいます。

Mittsu::ImageUtilsは、Mittsuが用意している画像関係のユーティリティクラスであり、
load_textureメソッドは指定した画像を読み込んでMittsu::Textureオブジェクトにして返してくれます。

後は、このMittsu::Textureオブジェクトを、

```ruby
material = Mittsu::MeshLambertMaterial.new(map: texture_map)
```

のように、マテリアル生成時のオプション「map」に与えることで、
マテリアルにテクスチャマップを設定できます。

## ノーマルマッピング

テクスチャマップはあくまでも画像を貼り付けているだけなので、どうしても平面的な見た目になります。

マテリアル種別を「MeshPhongMaterial」にすると、「ノーマルマップ」というテクスチャを使って、
テクスチャマップに凹凸があるように見せることが可能になります。

先ほどの地球のテクスチャマップに対応するノーマルマップの画像は以下のようになります。

{{<
    figure src="/images/5000_material/earth_normal.png"
    class="center" width="640" height="480"
>}}

ノーマルマップを貼って、凹凸感を出した場合のソースコード全体が以下になります。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
scene.fog = Mittsu::Uniform.new(:float, 2000.0)
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

directionalLight = Mittsu::DirectionalLight.new(0xffffff);
directionalLight.position.set(1, 1, 1)
scene.add(directionalLight)

texture_map = Mittsu::ImageUtils.load_texture("./earth.png")
normal_map = Mittsu::ImageUtils.load_texture("./earth_normal.png")

geometry = Mittsu::SphereGeometry.new(1, 32, 32)
material = Mittsu::MeshPhongMaterial.new(map: texture_map, normal_map: normal_map)
mesh = Mittsu::Mesh.new(geometry, material)
mesh.position.z = -2
scene.add(mesh)

renderer.window.run do
  renderer.render(scene, camera)
end
```

これを実行すると、

{{<
    figure src="/images/5000_material/fig_6.png"
    class="center" width="640" height="480"
>}}

のように、ノーマルマップの色調差に基づいて、テクスチャマップに凹凸感が出るようになります。

###### ノーマルマップ(normal_map)とバンプマップ(bump_map)の違い

ノーマルマップと同じように、テクスチャに凹凸感を出す技術として「バンプマッピング(Bump Mapping)」があります。

Mittsu::MeshPhongMaterialでもオプション「bump_map」としてサポートされていますが、
Ruby合宿2022では基本的にノーマルマップのみを使うようにしてください。

※ 2022/02現在のMittsuのバージョンでは動作確認ができなかったため。

ノーマルマップとバンプマップの違いは、凹凸感を出す元情報が、モノクロ画像の濃淡差で陰影を付ける
方法（バンプマップ）か、ピクセル単位の色の違いによって法線ベクトルの向きを表現し、
その法線ベクトルに基づいて凹凸を付ける方法（ノーマルマップ）かの違いになります。



