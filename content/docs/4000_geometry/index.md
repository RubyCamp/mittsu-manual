---
title: 'ジオメトリ（形状）'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 4000
summary: シーンに登場する物体の形状を決定付けるジオメトリの種類と扱い方について説明します。
---

## ジオメトリ（Geometry）とは

ジオメトリとは、3次元空間に配置する物体の「形状」を表現するデータ（頂点・辺・面など）の集まりです。

3次元空間に配置する物体の「基本形状」と表現すると分かりやすいかも知れません。

直方体・球・平面・三角錐・四角錐・トーラス・etc...と、多種多様な形状のジオメトリが、標準機能として用意されています。

ジオメトリは、物体の形状・サイズを決めるものですが、それだけでは3次元空間に配置することはできません。

物体は、形状の他にも、表面材質と位置情報が伴わないと、3次元空間に配置してカメラで撮影することができないためです。

材質と位置情報はそれぞれ後述しますが、ここでは代表的なジオメトリの種類と、その生成方法を見ていきましょう。

##### BoxGeometry（直方体）

直方体を表すジオメトリです。

initializeメソッドは以下のようになります。

```ruby
Mittsu::BoxGeometry.new(width, height, depth, segment_w = 1, segment_h = 1, segment_d = 1) #-> BoxGeometry
```

| 引数 | 意味 |
|------|------|
| width | 横幅 |
| height | 縦幅 |
| depth | 奥行 |
| segment_w | 横方向の分割数 |
| segment_h | 縦方向の分割数 |
| segment_d | 奥行方向の分割数 |

(例)

横・縦・奥行の大きさをそれぞれ1とし、各方向に4分割した立方体。

```ruby
geometry = Mittsu::BoxGeometry.new(1, 1, 1, 4, 4, 4)
```

上記で生成したジオメトリを、ワイヤーフレームマテリアルを適用の上でメッシュ化して表示したものが以下になります。

{{<
    figure src="/images/4000_geometry/fig_1.png"
    class="center" width="640" height="480"
>}}

横・縦・奥行のそれぞれの方向に4分割ずつされて、四角ポリゴンが4 * 4 * 4 = 64個で構成されていることが分かります。


##### SphereGeometry（球）

球体を表すジオメトリです。

initializeメソッドは以下のようになります。

```ruby
Mittsu::SphereGeometry.new(
  radius = 50.0, width_segments = 8, height_segments = 6,
  phi_start = 0.0, phi_length = (::Math::PI * 2.0),
  theta_start = 0.0, theta_length = ::Math::PI) #-> SphereGeometry
```

| 引数 | 意味 |
|------|------|
| radius | 球の半径 |
| width_segments | 横方向の分割数 |
| height_segments | 縦方向の分割数 |
| phi_start | 水平方向の描画開始角度(単位: rad) |
| phi_length | 水平方向の描画量(単位: rad) |
| theta_start | 垂直方向の描画開始角度(単位: rad) |
| theta_length | 垂直方向の描画量(単位: rad) |

(例)

半径を1とし、縦・横それぞれ16分割した球は以下のようになります。

```ruby
geometry = Mittsu::SphereGeometry.new(1, 16, 16)
```

上記で生成したジオメトリを、ワイヤーフレームマテリアルを適用の上でメッシュ化して表示したものが以下になります。

{{<
    figure src="/images/4000_geometry/fig_2.png"
    class="center" width="640" height="480"
>}}

###### phi_＊, theta_＊ について

これらのオプションを指定する機会はあまり無いかもしれませんが、それぞれ水平・垂直方向の描画量を決めるパラメータです。

例えば、水平方向の描画開始角度(phi_start)を0.0とし、描画量(phi_length)を、デフォルトの半分である
π(Math::PI)に設定した場合に描画されるジオメトリは以下のように縦に切られた半球となります。

```ruby
geometry = Mittsu::SphereGeometry.new(1, 16, 16, 0.0, Math::PI)
```

{{<
    figure src="/images/4000_geometry/fig_3.png"
    class="center" width="640" height="480"
>}}

theta_start, theta_lengthをそれぞれ0.0, π/2(デフォルトの半分)にすると、当然ながら横方向に切られたような半球になります。

```ruby
geometry = Mittsu::SphereGeometry.new(1, 16, 16, 0.0, Math::PI * 2, 0.0, Math::PI / 2)
```

{{<
    figure src="/images/4000_geometry/fig_4.png"
    class="center" width="640" height="480"
>}}


##### PlaneGeometry（平面）

平面を表すジオメトリです。

initializeメソッドは以下のようになります。

```ruby
Mittsu::PlaneGeometry.new(width, height, width_segments = 1, height_segments = 1) #-> PlaneGeometry
```

| 引数 | 意味 |
|------|------|
| width | 横幅 |
| height | 縦幅 |
| width_segments | 横方向の分割数 |
| height_segments | 縦方向の分割数 |

(例)

横幅を4、縦幅を1とし、縦・横それぞれ4分割した平面は以下のようになります。

```ruby
geometry = Mittsu::PlaneGeometry.new(4, 1, 4, 4)
```

上記で生成したジオメトリを、ワイヤーフレームマテリアルを適用の上でメッシュ化して表示したものが以下になります。

{{<
    figure src="/images/4000_geometry/fig_5.png"
    class="center" width="640" height="480"
>}}

##### CylinderGeometry（円筒）

円筒を表すジオメトリです。

initializeメソッドは以下のようになります。

```ruby
Mittsu::CylinderGeometry.new(
  radius_top = 20.0, radius_bottom = 20.0, height = 100.0,
  radial_segments = 8, height_segments = 1, open_ended = false,
  theta_start = 0.0, theta_length = Math::PI * 2.0) #-> CylinderGeometry
```

| 引数 | 意味 |
|------|------|
| radius_top | 上端部の半径 |
| radius_bottom | 下端部の半径 |
| height | 円筒の高さ |
| radial_segments | 円周方向の分割数 |
| height_segments | 高さ方向の分割数 |
| open_ended | 終端部を閉じるか開くか（true: 開く＝リレーのバトンのような、末端が開いた円筒となる) |
| theta_start | 垂直方向の描画開始角度(単位: rad) |
| theta_length | 垂直方向の描画量(単位: rad) |

(例)

全てデフォルト値で構成した円筒は以下のようになります。

```ruby
geometry = Mittsu::CylinderGeometry.new
```

上記で生成したジオメトリを、ワイヤーフレームマテリアルを適用の上でメッシュ化して表示したものが以下になります。

{{<
    figure src="/images/4000_geometry/fig_6.png"
    class="center" width="640" height="480"
>}}

##### CircleGeometry（円）

平面の円を表すジオメトリです。

initializeメソッドは以下のようになります。

```ruby
Mittsu::CircleGeometry.new(
  radius = 50.0, segments = 8,
  theta_start = 0.0, theta_length = (::Math::PI * 2.0)) #-> CircleGeometry
```

| 引数 | 意味 |
|------|------|
| radius | 上端部の半径 |
| segments | 下端部の半径 |
| height | 円筒の高さ |
| theta_start | 描画開始角度(単位: rad) |
| theta_length | 描画量(単位: rad) |

(例)

半径を1、分割数を8（デフォルト）とした円は以下のようになります。

```ruby
geometry = Mittsu::CircleGeometry.new(1)
```

上記で生成したジオメトリを、ワイヤーフレームマテリアルを適用の上でメッシュ化して表示したものが以下になります。

{{<
    figure src="/images/4000_geometry/fig_7.png"
    class="center" width="640" height="480"
>}}

### その他のジオメトリ

Ruby合宿2022において使用する主なジオメトリは上記の通りです。

その他のジオメトリについては、名称とinitializeメソッドの仕様だけ以下に列挙します。

###### DodecahedronGeometry

```ruby
Mittsu::DodecahedronGeometry.new(radius = 1.0, detail = 0) #-> DodecahedronGeometry
```

###### IcosahedronGeometry

```ruby
Mittsu::IcosahedronGeometry.new(radius = 1.0, detail = 0) #-> IcosahedronGeometry
```

###### LatheGeometry

```ruby
Mittsu::LatheGeometry.new(points, segments = 12, phi_start = 0.0, phi_length = (::Math::PI * 2.0)) #-> LatheGeometry
```

###### OctahedronGeometry

```ruby
Mittsu::OctahedronGeometry.new(radius = 1.0, detail = 0) #-> OctahedronGeometry
```

###### ParametricGeometry

```ruby
Mittsu::ParametricGeometry.new(func, slices, stacks) #-> ParametricGeometry
```

###### PolyhedronGeometry

```ruby
Mittsu::PolyhedronGeometry.new(vertices, indices, radius = 1.0, detail = 0) #-> PolyhedronGeometry
```

###### RingGeometry

```ruby
Mittsu::RingGeometry.new(inner_radius = 0.0, outer_radius = 50.0, theta_segments = 8, phi_segments = 8, theta_start = 0.0, theta_length = (::Math::PI * 2.0)) #-> RingGeometry
```

###### TetrahedronGeometry

```ruby
Mittsu::TetrahedronGeometry.new(radius = 1.0, detail = 0) #-> TetrahedronGeometry
```

###### TorusGeometry

```ruby
Mittsu::TorusGeometry.new(radius = 100.0, tube = 40.0, radial_segments = 8, tubular_segments = 6, arc = (::Math::PI * 2.0)) #-> TorusGeometry
```

###### TorusKnotGeometry

```ruby
Mittsu::TorusKnotGeometry.new(radius = 100.0, tube = 40.0, radial_segments = 64, tubular_segments = 8, p_val = 2, q_val = 3) #-> TorusKnotGeometry
```
