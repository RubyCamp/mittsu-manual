---
title: '当たり判定について'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 10000
summary: 3D空間における物体同士の当たり判定について説明します。
---

## 3D空間における当たり判定とは

ゲームなどのインタラクティブなプログラムにおいては、「当たり判定」がどうしても必須になってきます。

Mittsuにおいてオブジェクト同士が「当たっている」か否かを判定する方法は、
大きく分けて2種類存在します。

1つは、オブジェクトの座標同士の「距離」を計算して接触の有無を調べるもの。

もう1つは、「Raycast」と呼ばれる手法によって、オブジェクトから仮想的な「光線」を発射して、
その光線が当たる範囲にどのようなオブジェクトが存在しているかを調べる方法です。

多くの場合は、この2つを組み合わせて「当たり判定」を実現します。

## オブジェクト間の距離の測り方

ワールド座標系（シーン）に含まれる全ての3Dオブジェクトは、3次元の「位置」情報を持っています。

この位置情報は、Mittsu::Vector3という3次元ベクトルを表現するクラスによって保持されています。

例えば、以下のような位置関係にある2つの立方体を考えてみましょう。

※ 簡単のため、Y軸成分は0としています。

{{<
    figure src="/images/10000_collision/fig_1.png"
    class="center" width="640" height="480"
>}}

これはもうRubyの問題というよりは数学の問題に属するかも知れませんが、2つの3次元ベクトル間の
距離の公式に当てはめて解くようRuby（とMittsu）で記述すると、以下のようになります。

```ruby
v1 = Mttsu::Vector3.new(-2, 0, -2)
v2 = Mttsu::Vector3.new(2, 0, -2)
dx = v1.x - v2.x
dy = v1.y - v2.y
dz = v1.z - v2.z
puts Math.sqrt((dx * dx) + (dy * dy) + (dz * dz))  #=> 4.0
```

2つのベクトル間で、対応する成分同士の差を取って、得られた各成分毎の差分の2乗を足し合わせて
平方根を取るという式になります。

つまり、答えは「4.0」ですね。

そして、Mittsu::Vector3クラスには様々なメソッドが用意されており、その中の一つに「distance_to」
という非常に便利なメソッドが存在します。

このdistance_toメソッドを使うと、上記の2つのベクトル間の距離を一発で計算してくれるので
大変便利です。

```ruby
v1 = Mttsu::Vector3.new(-2, 0, -2)
v2 = Mttsu::Vector3.new(2, 0, -2)
puts v1.distance_to(v2)  #=> 4.0
```

また、メソッド名が適切に付けられているので、コードから何をやっている処理なのかを読み取り
易くて良いですね。

3Dオブジェクトの場合、位置を表すベクトルは「position」メソッドで一発で取れますので、
2つのメッシュ「m1」「m2」間の距離は、

```ruby
m1.position.distance_to(m2.position)
```

のように記述できます。

なお、この「position」で得られる座標は、オブジェクトのローカル座標系の原点の位置になります。

一般にローカル座標の原点はオブジェクトの「中心点」になりますので、2つのオブジェクトのposition間の距離が
「0」になるということは、2つのオブジェクトが重なり合っている状態ということになるので、判定
時には若干の調整をしないと、いわゆる「食い込み」が発生してしまうことになります。

それでは、以上を踏まえてdistance_toによる当たり判定を実装してみましょう。

以下のコードで、サイズの異なる2つの球同士の当たり判定を行い、両者が触れ合った瞬間、
片方が消える…という動きを実現してみます。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

geom1 = Mittsu::SphereGeometry.new(0.5, 8, 8)
mat1 = Mittsu::MeshBasicMaterial.new(color: 0xff0000)
mesh1 = Mittsu::Mesh.new(geom1, mat1)
scene.add(mesh1)

geom2 = Mittsu::SphereGeometry.new(1, 8, 8)
mat2 = Mittsu::MeshBasicMaterial.new(color: 0x00ff00)
mesh2 = Mittsu::Mesh.new(geom2, mat2)
scene.add(mesh2)

mesh1.position.z = -4
mesh1.position.x = 2
mesh2.position.z = -4
mesh2.position.x = -2

renderer.window.run do
  # 大きい方の球を1フレーム分小さな球に接近させる
  mesh2.position.x += 0.03
  
  # 2つの球の間の距離を計算
  distance = mesh1.position.distance_to(mesh2.position)
  
  # 得られた距離が、互いの半径の合計値（1.0 + 0.5）以下になったら触れたと判定する
  if distance <= 1.5
    # シーンから大きい方の球を除去
    scene.remove(mesh2)
  end
renderer.render(scene, camera)
end
```

実行結果は以下のようになります。

{{<
    figure src="/images/10000_collision/fig_2.gif"
    class="center" width="640" height="480"
>}}

これは、説明を簡単にするために極力シンプルに実装したパターンですので、このまま他の
あらゆるケースに応用できるかというと難しいです。

なぜなら、球であれば中心点（オブジェクトの「position」）から全ての表面が等距離（半径r）
ですので、どこで接触しても同じ距離で判定できますが、立方体など球ではない形状の場合、
そうはいかないためです。

※ 球以外のジオメトリの当たり判定を厳密に行うには、もう少し複雑な処理が必要になるのですが、
ここでは最小限の利用方法の説明にとどめたいと思います。

## Raycasterによる当たり判定対象の絞り込み

distance_toだけでもかなりの部分当たり判定はこなすことが可能です。

しかし、3D空間内にオブジェクトの数が増えてくると、当たり判定を行う「組み合わせ」の数が
どんどん膨らんでいくという問題に直面してしまいます。

例えば、シーン内に半径1の球が10個配置され、互いにランダムに運動しているとしましょう。

これらの球同士が接触した場合、互いに3D空間から消滅する…というプログラムを考えてみます。

この場合、distance_toによる当たり判定は、素朴に実装した場合1フレーム（1/60秒）毎に
10 ＊ 9 ＝ 90回必要になってきますね。

100個だと、100 ＊ 99 = 9900回です。

実際はもうちょっと減らすことはできますが、しかしたった100個のオブジェクトでこの増え方では、
なかなか性能がスケールしてくれません。

何とかして計算回数を減らしたいものですね。

そんな場合に使えるのが、Raycasterです。

前述の通り、Raycasterはオブジェクトから進行方向を決めて仮想的な光線を発射し、その光線と
「交差」したオブジェクトだけを選択してくれます。

これは、RubyレベルではなくOpenGL（GLFW）のdll内部で計算してくれるため、極めて高速に処理
されます。

これによって得られるオブジェクトの配列は、

* 光線と交差したオブジェクトのみが
* 光線を発したオブジェクトから見て近い順に

並べられて格納されてきます。

ですので、一般的にはRaycasterで得られた配列の先頭要素のオブジェクト（が存在すれば）とだけ
distance_toによって距離を測定すれば、100個のオブジェクトがあっても、100回のRaycaster実行で
当たり判定が終了するということになります。

#### Raycasterの使い方

Raycasterは、Mittsu::Raycasterクラスのインスタンスとして利用します。

initializeメソッドは以下のような仕様になります。

```ruby
Mittsu::Raycaster.new(origin = Vector3.new, direction = Vector3.new, near = 0.0, far = Float::INFINITY) #=> Raycaster 
```

| 引数 | 意味 |
|------|------|
| origin | 光線の発射位置 |
| direction | 光線の向き |
| near | 光線の交差判定の近端 |
| far | 光線の交差判定の遠端 |

originは、そのままRaycasterの光線を飛ばす元、つまり当たり判定の基準オブジェクトの位置です。

directionが少し分かりにくいですが、これは、originの位置から光線を飛ばす「方向」を定義する
単位ベクトルです。

ここで言う単位ベクトルとは、簡単に言えば3次元ベクトルの「長さ」（norm: ノルム）が1となる、
各成分が0～1の範囲で決定付けられるベクトル…という程度の意味と理解しておいてください。

任意のベクトルを単位ベクトル化するのはやや面倒な計算が必要ですが、Mittsu::Vector3クラスには
これを簡単に実行してくれる「normalize」という便利なメソッドが用意されています。

例えば、原点[0, 0, 0] から、方向[1, 1, 1] を通って、nearからfarまでの区間
（デフォルトは原点から無限遠まで）を対象に交差判定を行うことを考えると、光線の向きは
以下のようになります。

{{<
    figure src="/images/10000_collision/fig_2.png"
    class="center" width="640" height="480"
>}}

この場合の単位ベクトルは、手で計算して作ると以下のような値を取ります。

```ruby
Mittsu::Vector3.new(0.5773502691896258, 0.5773502691896258, 0.5773502691896258)
```

計算式は割愛しますが、いちいちこんな値を計算したくはないですね。

なお、normalizeしなくてもdirectionとして使えるベクトルも存在します。

それは、X・Y・Zのいずれか1つの成分のみが1または-1となるベクトルです。

これは、当然ながら長さ（ノルム）は1ですので、計算するまでもなくそのまま単位ベクトルとして
通用するわけです。

しかし、いちいちそのような使い分けをするのも面倒な話ですので、directionを決める場合は
かならずMittsu::Vector3クラスにnormalizeメソッドを当てて単位ベクトルを作るもの、と整理
しておく方が良いでしょう。

それでは、先ほどの2つの球が接近する当たり判定について、Raycasterを交えて
実装するとどうなるかを見てみましょう。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

geom1 = Mittsu::SphereGeometry.new(0.5, 8, 8)
mat1 = Mittsu::MeshBasicMaterial.new(color: 0xff0000)
mesh1 = Mittsu::Mesh.new(geom1, mat1)
scene.add(mesh1)

geom2 = Mittsu::SphereGeometry.new(1, 8, 8)
mat2 = Mittsu::MeshBasicMaterial.new(color: 0x00ff00)
mesh2 = Mittsu::Mesh.new(geom2, mat2)
scene.add(mesh2)

mesh1.position.z = -4
mesh1.position.x = 2
mesh2.position.z = -4
mesh2.position.x = -2

# 交差判定用Raycasterの向きを決定する単位ベクトルを生成する
norm_vector = Mittsu::Vector3.new(-1, 0, 0).normalize

# 交差判定用のRaycasterオブジェクトを生成する
raycaster = Mittsu::Raycaster.new

renderer.window.run do
	mesh2.position.x += 0.03

	# raycasterの発射点を更新する
	# 原点はmesh1（画面右側の赤く小さい球）の中心座標とし、そこから-X方向
	# （画面左。緑の大きい球の方向）に向けて光線を飛ばす
	raycaster.set(mesh1.position, norm_vector)

	# Mittsu::Raycaster#intersect_objectsで、交差判定を実行
	# 引数は、交差判定対象となるオブジェクトの配列となる
	# （今回はサンプルなので要素1つのみだが、任意の個数判定対象にできる）
	collisions = raycaster.intersect_objects([mesh2])

	# 配列として返ってくる交差判定結果の件数が1件以上あれば当たっている可能性のある
	# オブジェクトが存在するということになる
	if collisions.size > 0
		# 最も近距離にあるオブジェクトを得る
		obj = collisions.first[:object]
		# 当該オブジェクトと、当たり判定元オブジェクトの位置との距離を測る
		distance = mesh1.position.distance_to(obj.position)
		# 後は同じ
		if distance <= 1.5
			scene.remove(obj)
		end
	end
	renderer.render(scene, camera)
end
```

実行結果は先のサンプルと同じになります。

ここで、Mittsu::Raycaster#set, Mittsu::Raycaster#intersect_objectsという
メソッドが登場しました。


Mittsu::Raycaster#setで、光線を発射する原点と向きを決定し、交差判定を実施します。

setの引数は、光線発射位置となるMittsu::Vector3オブジェクト（多くの場合、メッシュの
positionになります）と、光線の向きを表す単位ベクトル（Mittsu::Vector3）です。

そして、setで飛ばした光線とオブジェクトの交点検出を行うのが、Mittsu::Raycaster#intersect_objectsです。

引数には、当該Raycasterで交差判定を行う全ての対象オブジェクトが入った配列を指定します。

これは、事前にプログラマー自身で用意しておくか、オブジェクト間に親子関係が設定されているならば、
「obj.children」のようにして子オブジェクトの集合を得て使ってもよいでしょう。

※ こういう時に、Mittsu::Groupクラスが役に立ちます。

戻り値は、交差判定結果を格納したハッシュの配列になります。

前述の通り、交差判定の結果として距離が近いオブジェクトから順に格納されていますので、
「collisions.first」で最も近いオブジェクトの交差判定結果（ハッシュ）を得ることができます。

この交差判定結果ハッシュは、以下のようなキーを持っています。

| キー名 | 意味 |
|--------|------|
| object | 対象オブジェクト |
| distance | 判定で得られた距離 |
| point | ワールド座標系上での交点 |
| face | 光線と交差したオブジェクト上の面（face） |

distanceキーにも距離は入っていますので、「collisions.first[:distance]」として距離を得てもよい
のですが、厳密な当たり判定距離に変換するのには若干加工が必要ですので、シンプルに「:object」キー
で対象オブジェクトを取得し、distance_toで判定した方が楽であると思われます。

point, faceについては、Ruby合宿2022の範囲では扱いませんが、複雑な形状においてより厳密な当たり
判定を行う場合などに使用できる値です。

## ウィンドウ上の任意の二次元座標からRaycasterを作る

言葉にすると分かりにくいかも知れませんが、要はマウス等でウィンドウ上の任意の座標をクリック
した際、そのクリックされた座標（2次元座標： ウィンドウ座標系）と交錯する3D空間内のオブジェクト
を得る、というシチュエーションです。

3DCGを作成するソフトウェアなどでは一般的な「3Dオブジェクトをクリックして選択する」といった
ような挙動を実現する際に不可欠な機能です。


では、2次元平面であるウィンドウ座標系から、どうやって奥行のある3次元座標系に配置されている
3Dオブジェクトを特定するかということになりますが、これは、

「カメラが存在している座標が、そのままウィンドウが3D空間内で見ている座標そのもの」

という大前提に基づいて、カメラの現在位置をoriginとし、originからマウスでクリックされた座標
に向かう単位ベクトルを生成してdirectionとすることで実現します。

これもまた計算式は少々ややこしくなるのですが、やはりMittsu::Raycasterにはこれを簡単に
計算してくれる素晴らしいメソッドが用意されています。

それが「Mittsu::Raycaster#set_from_camera」です。

具体的な使い方は以下のようになります。

```ruby
raycaster.set_from_camera(mouse_position, camera)
```

ここで、「mouse_position」は、2次元ベクトルを表すMittsu::Vector2クラスのインスタンスです。

重要な点として、この引数で渡す値は、ウィンドウ座標系そのままではダメで、単位ベクトル化しない
といけない点にありますが、Mittsu::Vector3の場合と異なり今回はウィンドウ座標系からの変換になるので、
少々複雑な変換式になります。

本マニュアルのサンプルでは、ウィンドウのサイズは800x600で統一しています。

このウィンドウ上のちょうど真ん中あたり（x: 400, y: 300）をクリックしたとしましょう。

すると、この座標からset_from_cameraメソッドに渡すMittsu::Vector2インスタンスは次のように生成します。

```ruby
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
position = Mittsu::Vector2.new(400, 300)
mouse_position = Mittsu::Vector2.new
mouse_position.x = ((position.x / SCREEN_WIDTH) * 2.0 - 1.0)
mouse_position.y = ((position.y / SCREEN_HEIGHT) * -2.0 + 1.0)
```

※ 上記の式は、Mittsu公式サンプル集から引用し、見やすいように若干体裁を整えたものになります。

実際の使用例として、3D空間内に20個の球をランダム配置し、それらをマウスクリックで選択（色を変える）
できるようにするサンプルを掲載します。

※ 陰影感を付けるため、DirectionalLightとMeshPhongMaterialを使っています。

```ruby
require 'mittsu'

SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
ASPECT = SCREEN_WIDTH / SCREEN_HEIGHT.to_f

scene = Mittsu::Scene.new
camera = Mittsu::PerspectiveCamera.new(75.0, ASPECT, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: SCREEN_WIDTH, height: SCREEN_HEIGHT, title: 'RubyCamp 2022'

# 陰影をつけるためのライトを配置
light = Mittsu::DirectionalLight.new(0xffffff, 1.0)
light.position.set(1, 2, 1)
scene.add(light)

# 球オブジェクトをグルーピングするための親オブジェクトを生成
spheres = Mittsu::Group.new
scene.add(spheres)

# 20個の球オブジェクトを生成して、spheresオブジェクトの子として登録
20.times do
  geometry = Mittsu::SphereGeometry.new(0.5, 8, 8)
  material = Mittsu::MeshPhongMaterial.new(color: 0x00ff00)
  mesh = Mittsu::Mesh.new(geometry, material)
  mesh.position.x = rand(10.0) - 5.0
  mesh.position.y = rand(10.0) - 5.0
  mesh.position.z = rand(10.0) - 15.0
  spheres.add(mesh)
end

# Raycasterとマウス位置の単位ベクトルを収めるオブジェクトを生成
raycaster = Mittsu::Raycaster.new
mouse_position = Mittsu::Vector2.new

# マウスクリックイベントハンドラを設定する
renderer.window.on_mouse_button_pressed do |glfw_button, position|
  case glfw_button
  # クリックされたボタンが左クリックである場合
  when GLFW_MOUSE_BUTTON_LEFT
    # ウィンドウ座標から必要な単位ベクトルを生成
    mouse_position.x = ((position.x / SCREEN_WIDTH) * 2.0 - 1.0)
    mouse_position.y = ((position.y / SCREEN_HEIGHT) * -2.0 + 1.0)

    # 当たり判定実行
    raycaster.set_from_camera(mouse_position, camera)
    intersects = raycaster.intersect_objects(spheres.children)
    # 交差判定を得られたオブジェクト全ての色を赤に変更する
    intersects.each do |intersect|
        intersect[:object].material.color.set(0xff0000)
    end
  end
end

renderer.window.run do
  renderer.render(scene, camera)
end
```

{{<
    figure src="/images/10000_collision/fig_3.gif"
    class="center" width="640" height="480"
>}}

これらの機能と、イベントハンドラを組み合わせることで、思った以上に複雑な処理も実現できますので、
是非チャレンジしてみてください。