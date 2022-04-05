---
marp: true
header: 深層学習 系列データのためのネットワーク
footer: "© 2020-2020, H.Takahashi" 
paginate: true
---

# 深層学習
## 系列データのためのネットワーク
#### 会津大学 コンピュータ理工学研究科 コンピュータ情報システム学専攻 髙橋輝
---

# 系列データ
個々の要素が順序付きの集まり
$$
    \textbf{x}^1, \textbf{x}^2, \textbf{x}^3, \cdots, \textbf{x}^T
$$
として与えられるデータを系列データと呼ぶ

#### 約束
<!-- 
BaseColor: #FF0088
AccentColor: #22FF00
-->
- 要素の並びをインデックス$1, 2, 3,\cdots, T$で表し, $t$を時刻と呼ぶ.
    - 時刻は物理的な時間と対応するとは<span style="color: #FF0088;">限らない</span>
- 個々のデータが順序を持っている, つまり, 並びに意味を持っていればよい.
---

# 今回扱う問題
 ### 1. テキスト to 多クラス (Text to Multi-class)
- レストランの利用客の感想を3段階で評価.
- 文を構成する各単語をベクトルで表現. (下に例示)
    $\textbf{x}^1 = `\texttt{They}`, \textbf{x}^2=`\texttt{have}`, \cdots, \textbf{x}^{15} = `\texttt{better}`$
- データの最小単位は一つの文$(\textbf{X}_n =(\textbf{x}^1, \textbf{x}^2, \cdots, \textbf{x}^{T_n})).$
- 単語数は自由なので, 系列長$T_n$も自由.

---
## 今回扱う問題
### 2. 音声認識(Speech Recognition)
- 発話を記録した時間信号から発話内容を推定する.
- 信号は<span style="color: #FF0088">一定の周期で標本化され, 
        量子化されたデジタルデータ</span>(=一般的な音声データ)
- 前処理
    - 方法の例: $10ms$間隔で$25ms$幅の窓で切り出し, 周波数スペクトルの分布情報を取り出して, 特徴ベクトルの系列$(\bf x \rm^1, \bf x \rm^2, \cdots,)$を得る.
- 入力に前処理を行ったデータを取り, 発話を構成する<span style="color: #ff0088">音素(phoneme)</span> or 
<span style="color: #ff0088">発話内容を直接表す文字列</span>を推定する.
### 1, 2共に, 出力は入力と異なる長さの系列を出力できる必要がある
---

# リカレントニューラルネットワーク　<!-- fit -->

---

# 1. リカレントニューラルネットワーク(RNN)とはなんぞや？
A. リカレントニューラルネットワーク(Recurrent Neural Network)とは, 
<span style="color: #FF0088; padding:0 3pc 0 10pc;">内部に(有向)閉路を持つニューラルネットワーク</span> の総称である.

- 例として,
    - Elman Network
    - Jordan Network
    - Time Delay Network 
    - Echo State Network など様々なものがあるが, 始めは単純なものを考える.


---

<h2 style="position: absolute; top:80px; left:100px;">シンプルなRNN</h2>

![bg 80%](./figure/Figure6.2(a).svg)
![bg 80%](./figure/Figure6.2(b).svg)

---


前図のように,中間層のユニットの出力が自分自身に<span style="color: #FF0088">重み付きで</span>戻されるRNNを考える.
この自分自身に戻ってくるパスを<span style="color: #FF0088">帰還路</span>と呼ぶ.

この構造により, 中間層のユニットは, ひとつ前の状態を**覚える**ことができる.

また, このユニットは, ひとつ前の出力と, 現在の入力の両方を考慮して状態が変わるため, 振る舞いを**動的に変化させる**ことができる. 

この二つの特性により, この単純なRNNは系列データ中の"文脈"を捉えることが期待される.

---

RNNは各時刻tにつき1つの入力$\textbf{x}^t$を受け取り, 出力$\textbf{y}^t$を出力する.
これは, <span style="color: #ff0088">入力と同じ長さ</span>の系列を出力することを意味する.

過去に受け取った入力(理論上はすべて)が<span style="color: #ff0088">帰還路</span>を通して出力に影響を与える. 

#### 順伝播型ネットワークとの比較

| | 順伝播型ネットワーク | RNN |
| --- |-------------------- | --- |
| 帰還路 | なし | あり |
| 写像 | $\textbf{x}^{t} \mapsto y$ | $\bf (x^0, x^1, \cdots, x^{t}) \mapsto y$ |

このRNNは, 系列データについて, 順伝播型ネットワークと同じ<span style="color: #ff0088">万能性</span>を持つ.
<!-- 十分な個数の隠れ層ユニットがあれば任意の系列から系列への写像を任意の精度で近似できる. -->
---

### RNNの順伝播

系列$(\bf x^0, x^1, \cdots, x^{t})$を順に入力すると, 系列$(\bf y^0, y^1, \cdots, y^{t})$を出力する.
$$
    y : (\textbf{x}^0, \textbf{x}^1, \cdots, \textbf{x}^{t}) \mapsto (\textbf {y}^0, \textbf{y}^1, \cdots, \textbf{y}^{t})
$$

この計算の詳細を後のスライドで説明するが, その前に定義を行う.
#### 定義
- $\textbf{x}^t = (x_i^t)$ : ネットワークの入力
- $\textbf{u}^t = (u_j^t)$, $\textbf{z}^t = (z_j^t)$ : 中間層ユニットの入出力
- $\textbf{v}^t = (v_k^t)$, $\textbf{y}^t = (y_k^t)$ : 出力層ユニットの入出力
- $\textbf{d}^t = (d_k^t)$: 目標とする出力

---

#### 続・定義

- $\textbf{W}^{(in)} = (w_{ji}^{(in)})$: 入力層と中間層のユニット間の重み
- $\textbf{W} = (w_{j'j})$: 帰還路の結合重み
- $\textbf{W}^{(out)} = (w_{kj}^{(out)})$: 中間層と出力層のユニット間の重み
- $x_0^t = 1, z_0^t = 1$: 各層の$0$番目はバイアスを表現するため, 常に$1$を出力するユニットを配置する.
    - つまり, 中間層のバイアスは, $w_{j0}^{(in)}$, $w_{k0}^{(out)}$によって表現される.

---

### RNNの順伝播
#### 

---


---

## References 
1. https://qiita.com/mochimochidog/items/ca04bf3df7071041561a
1. Hornik, K., Stinchcombe, M., & White, H. (1989). Multilayer feedforward networks are universal approximators. Neural networks, 2(5), 359-366.