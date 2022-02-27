---
title: 'イベント'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 8000
summary: Mittsuの世界で発生する各種イベントとその扱い方について説明します。
---

## イベントとは

本稿における「イベント」とは、利用者がキーボードやマウスなどの「入力デバイス」を用いて、
プログラムに対して利用者の意思を伝える行為と定義します。

例えば、「利用者がキーボードの左矢印キーを押下した」とか「利用者がマウスを〇〇だけ動かした」
といったような出来事が「イベント」です。

## イベントハンドラ

要するに人間が入力デバイスを介して行った「操作」がイベントということですが、ではその操作を
受けたプログラム側（我々が作っている方）で、そのイベントが届いた時どのように挙動すべきかを
定義するメソッドのことを「イベントハンドラ」と呼びましょう。

「イベント」を「ハンドリング（取り扱う）」するメソッドなので「イベントハンドラ」です。

Mittsuにおけるイベントハンドラは、レンダラーが提供する「Mittsu::GLFW::Window」クラスの
インスタンスに定義されています。

#### 具体例

例えば、「キーボードのSPACEキーが押下される都度、ワールド座標原点から-Z方向に向けて球体を
撃ち出す」というプログラムを記述してみると、以下のようになります。

```ruby
require 'mittsu'

scene = Mittsu::Scene.new
camera = Mittsu::PerspectiveCamera.new(75.0, 1.333, 0.1, 1000.0)
renderer = Mittsu::OpenGLRenderer.new width: 800, height: 600, title: 'RubyCamp 2022'

# 弾のメッシュの参照を保持するための配列
bullets = []

# キーボードの任意のキーが押下された際に自動的に呼び出される「on_key_pressed」メソッド
# の内容を定義する
# 引数「glfw_key」には、GLFW（OpenGLを扱うためのライブラリ）で定義された定数が自動的に
# セットされる
renderer.window.on_key_pressed do |glfw_key|
  # 押下されたキーがスペースキーであれば、新規の弾丸メッシュを生成してシーンと配列に
  # 格納する
  # ※ 簡単のためライトをセットしていないので、MeshBasicMaterialを使っている
  if glfw_key == GLFW_KEY_SPACE
    geometry = Mittsu::SphereGeometry.new(0.5, 8, 8)
    material = Mittsu::MeshBasicMaterial.new(color: 0xff0000)
    bullet = Mittsu::Mesh.new(geometry, material)
    scene.add(bullet)
    bullets << bullet
  end
end

# 連射してる様子を見やすくするため、若干カメラを右に移動させ、
# 弾の進行方向やや前方に向ける
camera.position.x = 3
camera.look_at(Mittsu::Vector3.new(0, 0, -3))

renderer.window.run do
  # 弾配列に含まれている全ての弾についてZ座標を１フレーム分進める
  bullets.each do |bullet|
    bullet.position.z -= 0.5
  end

  renderer.render(scene, camera)
end
```

このプログラムを実行した様子は以下のようになります。

{{<
    figure src="/images/8000_events/fig_1.gif"
    class="center" width="640" height="480"
>}}

画像では分かりにくいですが、スペースキーを押す都度新しい球（弾丸）
が画面に増えていく様子が見て取れますね。

なお、画像はGIFアニメーションなので途中で切れて頭に戻っていますが、このプログラムは
本当に最小限のものなので、発射した弾丸は無限に3D空間を-Z方向に進み続けます。

これだと、スペースキーを押す都度メモリ上のオブジェクトが増える一方になるので、実際
のプログラムでは見えない領域まで進んだら球（弾丸）を削除するような処理も必要になる
でしょう。

## イベントハンドラの種類

Mittsu::GLFW::Windowクラスでは、上記の「on_key_pressed」の他にも複数のイベントハンドラ
が用意されています。

以下、代表的な（Ruby合宿2022で使用を想定する）イベントハンドラを紹介していきます。

いずれも、「on_key_pressed」と同じくブロックを取って処理を記述するタイプですので、それぞれ
についてブロック引数の数と意味についても記載します。

※ ブロック引数の名称は自由に付けられますが、説明の便宜上決め打ちで名前を付けています。

##### on_mouse_button_pressed

マウスボタンの押下を検知した際に呼ばれます。

###### 例

```ruby
@renderer.window.on_mouse_button_pressed do |glfw_button, position|
	# イベント発火時の挙動を記述する
end
```

| ブロック引数 | 意味 |
| glfw_button | GLFW定数（マウスボタンの種別）|
| position | Mittsu::Vector2インスタンス（position.x, position.y で座標値が得られる）


##### on_mouse_button_released

マウスボタンが離されたことを検知した際に呼ばれます。

###### 例

```ruby
@renderer.window.on_mouse_button_released do |glfw_button, position| 
    # イベント発火時の挙動を記述する
end
```

| ブロック引数 | 意味 |
| glfw_button | GLFW定数（マウスボタンの種別）|
| position | Mittsu::Vector2インスタンス |

##### on_mouse_move

マウスの位置が移動したことを検知した際に呼ばれます。

###### 例

```ruby
@renderer.window.on_mouse_move do |position|
    # イベント発火時の挙動を記述する
end
```

| ブロック引数 | 意味 |
| position | Mittsu::Vector2インスタンス |

##### on_scroll

マウスホイールが回転したことを検知した際に呼ばれます。

###### 例

```ruby
@renderer.window.on_scroll do |offset|
    # イベント発火時の挙動を記述する
end
```

| ブロック引数 | 意味 |
| offset | Mittsu::Vector2インスタンス（offset.x, offset.y で各座標軸毎の回転量が得られる） |

※ なお、一般的なマウスではホイールは1軸であるためX成分は基本0になります。

##### on_key_released

キーボードのキーが押下された後、キーが離されたことを検知した際に呼ばれます。

###### 例

```ruby
@renderer.window.on_key_released do |glfw_key|
    # イベント発火時の挙動を記述する
end
```

| ブロック引数 | 意味 |
| glfw_key | GLFW定数（キーの種別） |

##### on_key_typed

キーボードのキーが押し続けられている（いわゆる「押しっ放し状態」）ことを検知した際に呼ばれます。

###### 例

```ruby
@renderer.window.on_key_typed do |glfw_key|
    # イベント発火時の挙動を記述する
end
```

| ブロック引数 | 意味 |
| glfw_key | GLFW定数（キーの種別） |

##### on_character_input

キーボードのキー（文字としてディスプレイに表示可能なものに限る）が押下された
ことを検知した際に呼ばれ、押下された文字そのものを返します。

###### 例

```ruby
@renderer.window.on_character_input do |char|
    # イベント発火時の挙動を記述する
end
```

| ブロック引数 | 意味 |
| char | 押下された文字そのもの |


## GLFW定数

キーボードのキーやマウスのボタンなどの識別用に、GLFWが用意している定数のことを、
本マニュアルでは「GLFW定数」と呼びます。

GLFW定数はそのまま利用してももちろん問題はありませんが、将来的にライブラリ側で変更される
可能性を考慮すると、以下のように自前のハッシュとしてワンクッション噛ませておくと、GLFWの
バージョンアップなどの際に定数が変化しても（あまりそのようなケースは無いかも知れませんが）
プログラム側を変更せずにこのハッシュだけアップデートすれば良い状態にできます。

以下、Ruby合宿2022の範囲内で使用する可能性のあるGLFW定数の一覧を、上述のハッシュ形式で紹介します。

```ruby
# GLFW定数へのマッピング
GLFW_CONSTS = {
    m_left: GLFW_MOUSE_BUTTON_LEFT,   # マウス左ボタン
    m_right: GLFW_MOUSE_BUTTON_RIGHT, # マウス右ボタン
    k_up: GLFW_KEY_UP,                # ↑
    k_down: GLFW_KEY_DOWN,            # ↓
    k_left: GLFW_KEY_LEFT,            # ←
    k_right: GLFW_KEY_RIGHT,          # →
    k_0: GLFW_KEY_0,                  # 0
    k_1: GLFW_KEY_1,                  # 1
    k_2: GLFW_KEY_2,                  # 2
    k_3: GLFW_KEY_3,                  # 3
    k_4: GLFW_KEY_4,                  # 4
    k_5: GLFW_KEY_5,                  # 5
    k_6: GLFW_KEY_6,                  # 6
    k_7: GLFW_KEY_7,                  # 7
    k_8: GLFW_KEY_8,                  # 8
    k_9: GLFW_KEY_9,                  # 9
    k_num0: GLFW_KEY_KP_0,            # テンキー0
    k_num1: GLFW_KEY_KP_1,            # テンキー1
    k_num2: GLFW_KEY_KP_2,            # テンキー2
    k_num3: GLFW_KEY_KP_3,            # テンキー3
    k_num4: GLFW_KEY_KP_4,            # テンキー4
    k_num5: GLFW_KEY_KP_5,            # テンキー5
    k_num6: GLFW_KEY_KP_6,            # テンキー6
    k_num7: GLFW_KEY_KP_7,            # テンキー7
    k_num8: GLFW_KEY_KP_8,            # テンキー8
    k_num9: GLFW_KEY_KP_9,            # テンキー9
    k_add: GLFW_KEY_KP_ADD,           # +
    k_sub: GLFW_KEY_KP_SUBTRACT,      # -
    k_multi: GLFW_KEY_KP_MULTIPLY,    # *
    k_divide: GLFW_KEY_KP_DIVIDE,     # /
    k_decimal: GLFW_KEY_KP_DECIMAL,   # .
    k_a: GLFW_KEY_A,                  # A
    k_b: GLFW_KEY_B,                  # B
    k_c: GLFW_KEY_C,                  # C
    k_d: GLFW_KEY_D,                  # D
    k_e: GLFW_KEY_E,                  # E
    k_f: GLFW_KEY_F,                  # F
    k_g: GLFW_KEY_G,                  # G
    k_h: GLFW_KEY_H,                  # H
    k_i: GLFW_KEY_I,                  # I
    k_j: GLFW_KEY_J,                  # J
    k_k: GLFW_KEY_K,                  # K
    k_l: GLFW_KEY_L,                  # L
    k_m: GLFW_KEY_M,                  # M
    k_n: GLFW_KEY_N,                  # N
    k_o: GLFW_KEY_O,                  # O
    k_p: GLFW_KEY_P,                  # P
    k_q: GLFW_KEY_Q,                  # Q
    k_r: GLFW_KEY_R,                  # R
    k_s: GLFW_KEY_S,                  # S
    k_t: GLFW_KEY_T,                  # T
    k_u: GLFW_KEY_U,                  # U
    k_v: GLFW_KEY_V,                  # V
    k_w: GLFW_KEY_W,                  # W
    k_x: GLFW_KEY_X,                  # X
    k_y: GLFW_KEY_Y,                  # Y
    k_z: GLFW_KEY_Z,                  # Z
    k_esc: GLFW_KEY_ESCAPE,           # ESC
    k_space: GLFW_KEY_SPACE,          # SPACE
    k_enter: GLFW_KEY_ENTER,          # ENTER
    k_rshift: GLFW_KEY_RIGHT_SHIFT,   # 右SHIFT
    k_rctrl: GLFW_KEY_RIGHT_CONTROL,  # 右CTRL
    k_lshift: GLFW_KEY_LEFT_SHIFT,    # 左SHIFT
    k_lctrl: GLFW_KEY_LEFT_CONTROL,   # 左CTRL
    k_tab: GLFW_KEY_TAB,              # TAB
    k_bs: GLFW_KEY_BACKSPACE,         # BS
    k_del: GLFW_KEY_DELETE,           # DEL
    k_insert: GLFW_KEY_INSERT,        # INSERT
    k_home: GLFW_KEY_HOME,            # HOME
    k_end: GLFW_KEY_END,              # END
    k_pup: GLFW_KEY_PAGE_UP,          # PAGE UP
    k_pdown: GLFW_KEY_PAGE_DOWN,      # PAGE DOWN
}
```
上記ハッシュと、このハッシュのキーと値を入れ替えたハッシュを用意しておくと便利です。

キーと値を入れ替えたハッシュは以下のようにして簡単に定義できます。

※ 値に重複が無いことが前提となります。

```ruby
GLFW_CONSTS_INVERTED = GLFW_CONSTS.invert
```
