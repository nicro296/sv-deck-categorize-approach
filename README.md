# sv-deck-categorize-approach

[English translation is here](https://github.com/nicro296/sv-deck-categorize-approach/edit/main/README.md#deck-type-evaluation-methods)

## デッキタイプの評価の方法
デッキタイプ分類済みのサンプルデッキを用いて分析対象のデッキ\[deck\](以降ターゲットデッキと呼ぶ)の分類を行う方法について、現在行っている範囲でまとめた。


#### 評価関数[V]
デッキタイプ毎にデッキを引数に持つ評価関数\[V(deck)\]を用意する。
<br>
得られる値はターゲットデッキがそのデッキタイプにどれくらい属しているかを表し、値が大きいほどそのデッキタイプである可能性が高いと判断され、以下の式で定義される。
<br>
```
V = W*S (0.0≦V≦1.0)
```
W:重み付け係数
<br>
S:デッキの一致度
<br>
それぞれ後で説明する。


##### 採用率[P]
カード[card]とその採用枚数\[n\]毎に定義される。
```
P(card,n) = サンプルデッキ群を用意して算出 (0.0≦P≦1.0)
```
###### 例 各プレイヤーのゴブリンの採用枚数が以下の時
|サンプルデッキ|ゴブリンの採用枚数|
|----|----|
|deckA|3|
|deckB|3|
|deckC|2|
|deckD|1|
|deckE|1|
|deckF|0|

P(ゴブリン,1) = 0.83;<br>
P(ゴブリン,2) = 0.50;<br>
P(ゴブリン,3) = 0.33;

##### デッキの一致度[S]
```
S(deck) = Σ[デッキ内のカード] P(card,n)
```
同名カードが複数採用され手入れも1枚目、2枚目、3枚目、それぞれのPの値を合計するためデッキのカード40枚のPの和となり、Sは0.0≦S≦40.0を満たす

##### 重み付け係数[W]
0.0≦V≦1.0を満たすようにWを指定する。
<br>
サンプルデッキの採用カードが散らばるほどSの値は小さくなりやすい。デッキタイプ間で評価値のスケールに差が生まれないように考慮し、Sが取りうる最大値\[Smax\]をもとに重み付けする。
<br>
Pの値が大きいものから順に40個足した和をSmaxとして
```
V = S/Smax
W = 1/Smax
```

#### サンプルデッキの用意について
個人でまとめをしていく上でデッキを用意することは難しいため、ターゲットデッキの未分類のデッキを手動でデッキタイプを命名してサンプルデッキとして利用する。
<br>
そしてそれをもとに分類されたデッキもサンプルデッキに追加していく、現在評価関数は0.8を閾値として分類を行い、どれにも当てはまらなかったものは未分類または、手動で命名している。

#### 不十分なサンプルデータへの配慮
サンプルデッキ数が少ない場合や偏っている場合への一策として以下のようにPの値を修正する。
<br>
<br>
```
P'(card,n) = 
  0.2(0.0 ≦ P(card,n) ≦0.2)
  0.4(0.2 ≦ P(card,n) ≦0.4)
  0.6(0.4 ≦ P(card,n) ≦0.6)
  0.8(0.6 ≦ P(card,n) ≦0.8)
  1.0(0.8 ≦ P(card,n) ≦1.0)
```

一人しか採用者がいないカードはサンプルデッキ数が増えるにつれ評価値への影響が少なくなりすぎてしまう。0.0≦P≦0.2を0.2とすることで評価を一人も採用者がいないカードと評価に差をつけることができる。
この評価関数Vはまだ未成熟である。関数の挙動を捉えやすく、扱いやすくするためにもPの取りうる値を制限することは都合がよい。
<br>

## デッキタイプの階層的な扱い
採用カードが似通ったデッキタイプ間の分類を行う際に、どうしても特定のカードに依存した分類を行う必要が出てくる。
<br>
特定カードへの依存、つまりカードのテキストへの依存は枚数分布からは得られない情報であるから根本的に別の手段で評価する必要があると考えている。条件設定が手作業な方法となってしまい避けたいところではあるがほかの方法が思いつかないので特定カードの採用枚数の条件を設定して判別する。
<br>
それに伴いデッキタイプの分類の構造を変更する。
デッキタイプはそれぞれ「カテゴリー」と「要素」を持つとする。
#### カテゴリー
> カテゴリーはそれぞれデッキを引数に持つ評価関数を持つ。つまり上記でデッキタイプとして扱っていたものをカテゴリーとして扱う。
> デッキタイプはそれぞれ1つのカテゴリーを持つ。

#### 要素
> 要素は特定カードの採用枚数を条件として判断される。
> デッキタイプは複数の要素を持つことができる。

ターゲットデッキは閾値を満たす最も評価値が高くなるカテゴリーと条件を満たす要素の情報からデッキタイプが導かれる。
全てのカテゴリーの評価値が閾値を満たさない時デッキタイプが「未分類」として扱う。

#### 例
RGW環境にいたデッキタイプ\[純共鳴ネメシス、共鳴(バハムート)ネメシス、人形共鳴ネメシス\]は以下のように定義できる。

|デッキタイプ|カテゴリー|要素|
|----|----|----|
|純共鳴Nm|共鳴|無し|
|共鳴(バハムート)Nm|共鳴|\[バハムート\]|
|人形共鳴Nm|人形共鳴|無し|

<br>
要素は以下のように定義される。

|要素名|判別カード|採用枚数の条件|
|----|----|----|
|バハムート|\[バハムート,終焉の地\]|4枚以上|

##### Q.人形共鳴ネメシスでカテゴリーを共鳴として要素に人形としていないのはなぜ?
> A.人形共鳴の評価関数が共鳴とは似ていないため一つのカテゴリーに入れる必要がないから。
> 基本的にはカテゴリーで分類を行い、評価関数が似通っているが分類を行いたい場合にカテゴリーを共通化して一方または双方に要素を追加することで差別化する。

同じカテゴリーでもデッキタイプごとにリストに異なった特徴があるためカテゴリーの評価関数はデッキタイプごとに用意する方が都合がよい。
<br>
##### Q.同カテゴリーのデッキタイプの評価関数間で閾値を満たしているものと満たしていないものが出た場合は?
> A.一つでも評価値を満たしているなら満たしているものとして扱う。
> デッキリストが似ていることよりも優先して分類に作用するカードがあるから要素で判断するので、もし共鳴(バハムート)のカテゴリー評価関数では閾値を満たさなくても純共鳴のカテゴリー評価関数で閾値を超えており、要素「バハムート」を持つデッキであれば共鳴(バハムート)に分類することになる。

##### Q.複数のデッキタイプの条件を満たしているときは?
> A.デッキタイプの条件(カテゴリーと要素)を複数満たしている時の優先順位を定義する必要がある。
> 上記の例では共鳴(バハムート)の条件を満たすデッキは純共鳴の条件を満たしているので優先順位は共鳴バハムートのほうが高く設定する。要素の条件がきついほど条件を満たしているときの優先順位は高く設定するのが基本。


#### 閾値に対するイメージ
上記の分類方法をJCGで使っている(閾値 = 0.80)イメージとして

1. リストが固まっているデッキタイプは手を加えなくても十分に分類される。
2. 大会上位デッキ(予選決勝勝ちデッキ)は26~32/32分類できている。
3. 大会エントリー全体では10~20%が未分類扱い、未分類を除いた誤った分類はないに等しい。
4. 個人的な視点だが少し見慣れないカードの積み方をしたリストをはじいているのでそういったリストが出てきていることを私自身が認知しやすい

似通っているデッキタイプが存在しているか、リストが変遷している時期か、サンプルデッキ数が確保できているか、によって精度が影響を受けやすい。
<br>
現在行っている方法の閾値は0.7から0.8あたりが考慮範囲。JCG 23rd Vol.1とVol.3をサンプルデッキとしてVol.4を分類したところ916デッキ中、未分類は81デッキ(閾値0.8)、33デッキ(閾値0.7)であった。誤分類らしきものは閾値0.7の時にベレロフォンを積んでいない3デッキが回復Bに分類されていた(Vol.1,Vol.3では未分類のまま対処していた)ことのみであった。
<br>
前期に比べて似通ったデッキタイプがない(前期の人形Nm,共鳴Nm,共鳴人形Nm)分閾値は低くても大きな問題はなさそう。
<br>
しかし閾値を下げることは似通ったデッキタイプが生まれやすくなるということであり要素による分類により注力することを意味するので目的に合わせて考える必要があると思う。





## Deck Type Evaluation Methods
Using a sample deck that has already been classified into a deck type, this section summarizes how to classify the deck [deck] to be analyzed (henceforth referred to as the target deck). This is my own method and not a completed theory.


#### Evaluation Function [V]
An evaluation function [V(deck)] is provided for each deck type, with the deck as an argument.
<br>
The obtained value indicates how much the target deck belongs to that deck type, and the higher the value, the more likely it is to be of that deck type, as defined by the following formula.
<br>
```
V = W*S (0.0≦V≦1.0)
```
W:weighted coefficient
<br>
S:Degree of deck match
<br>
Each of these is explained later.


##### Adoption rate [P].
Defined for each card [card] and its number of adopted cards [n].
```
P(card,n) = Calculated by preparing a group of sample decks (0.0≤P≤1.0)
```
###### Example When the number of goblins employed by each player is as follows
|sample deck|Number of cards(goblin) adopted|
|----|----|
|deckA|3|
|deckB|3|
|deckC|2|
|deckD|1|
|deckE|1|
|deckF|0|

P(goblin,1) = 0.83;<br>
P(goblin,2) = 0.50;<br>
P(goblin,3) = 0.33;

##### Degree of deck match [S]
```
S(deck) = Σ[cards in the deck] P(card,n)
```
Even if multiple cards with the same name are used, the P value of the first card, the second card, the third card, and each P value is summed, so the sum of the P values of the 40 cards in the deck is obtained, and S satisfies 0.0 ≤ S ≤ 40.0 .

##### weighted coefficient [W]
Specify W to satisfy 0.0 ≤ V ≤ 1.0.
<br>
The more scattered the cards in the sample decks, the smaller the value of S tends to be. To avoid differences in the scale of evaluation values among deck types, we weighted S based on the maximum possible value of S [Smax].
<br>
The sum of 40 [P] added in order from the highest value to the lowest is Smax,
```
V = S/Smax
W = 1/Smax
```

#### Preparation of Sample Decks
Since it is difficult to prepare a deck for an individual to put together, an unclassified deck from the target deck is used as a sample deck by manually naming the deck type.
<br>
The decks classified based on this are also added to the sample deck. Currently, the evaluation function uses 0.8 as the threshold for classification, and those that do not fit any of the criteria are either unclassified or named manually.

#### Consideration for Inadequate Sample Data
As a measure to deal with small or biased sample decks, the value of P is modified as follows.
<br>
<br>
```
P'(card,n) = 
  0.2(0.0 ≦ P(card,n) ≦0.2)
  0.4(0.2 ≦ P(card,n) ≦0.4)
  0.6(0.4 ≦ P(card,n) ≦0.6)
  0.8(0.6 ≦ P(card,n) ≦0.8)
  1.0(0.8 ≦ P(card,n) ≦1.0)
```

Cards with only one adopter have too little impact on the evaluation value as the number of sample decks increases. 0.0 ≤ P ≤ 0.2 can be used to differentiate the evaluation from cards with no single adopter.
This evaluation function V is still in its infancy. It is convenient to limit the possible values of P to make the behavior of the function easier to understand and handle.
<br>

## Hierarchical Treatment of Deck Types
When classifying between deck types that employ similar cards, it is inevitably necessary to rely on specific cards for classification.
<br>
Since dependence on specific cards, in other words, dependence on the text of cards, is information that cannot be obtained from the distribution of the number of cards, we believe it is necessary to use a fundamentally different method to evaluate this information. We have to set a condition for the number of specific cards to be used, which is a manual process that we would like to avoid, but we cannot think of any other method.
<br>
The structure of the deck type classification will be changed accordingly.
Each deck type shall have a "category" and an "element".
#### category
> Each category has its own evaluation function with the deck as an argument. In other words, what was treated as a deck type above is treated as a category.
> Each deck type has one category.

#### element
> Elements are determined based on the number of specific cards employed.
> A deck type can have more than one element.

The target deck type is derived from the information of the category with the highest evaluation value that satisfies the threshold value and the element that satisfies the condition.
When the evaluation values for all categories do not meet the threshold, the deck type is treated as "unclassified.

#### 例
The deck types \[pure resonance nemesis, resonance (bahamut) nemesis, puppet resonance nemesis\] that were in the RGW environment can be defined as follows.

|デッキタイプ|カテゴリー|要素|
|----|----|----|
|純共鳴Nm|共鳴|無し|
|共鳴(バハムート)Nm|共鳴|\[バハムート\]|
|人形共鳴Nm|人形共鳴|無し|

<br>
要素は以下のように定義される。

|要素名|判別カード|採用枚数の条件|
|----|----|----|
|バハムート|\[バハムート,終焉の地\]|4枚以上|

##### Q.人形共鳴ネメシスでカテゴリーを共鳴として要素に人形としていないのはなぜ?
> A.人形共鳴の評価関数が共鳴とは似ていないため一つのカテゴリーに入れる必要がないから。
> 基本的にはカテゴリーで分類を行い、評価関数が似通っているが分類を行いたい場合にカテゴリーを共通化して一方または双方に要素を追加することで差別化する。

同じカテゴリーでもデッキタイプごとにリストに異なった特徴があるためカテゴリーの評価関数はデッキタイプごとに用意する方が都合がよい。
<br>
##### Q.同カテゴリーのデッキタイプの評価関数間で閾値を満たしているものと満たしていないものが出た場合は?
> A.一つでも評価値を満たしているなら満たしているものとして扱う。
> デッキリストが似ていることよりも優先して分類に作用するカードがあるから要素で判断するので、もし共鳴(バハムート)のカテゴリー評価関数では閾値を満たさなくても純共鳴のカテゴリー評価関数で閾値を超えており、要素「バハムート」を持つデッキであれば共鳴(バハムート)に分類することになる。

##### Q.複数のデッキタイプの条件を満たしているときは?
> A.デッキタイプの条件(カテゴリーと要素)を複数満たしている時の優先順位を定義する必要がある。
> 上記の例では共鳴(バハムート)の条件を満たすデッキは純共鳴の条件を満たしているので優先順位は共鳴バハムートのほうが高く設定する。要素の条件がきついほど条件を満たしているときの優先順位は高く設定するのが基本。


#### 閾値に対するイメージ
上記の分類方法をJCGで使っている(閾値 = 0.80)イメージとして

1. リストが固まっているデッキタイプは手を加えなくても十分に分類される。
2. 大会上位デッキ(予選決勝勝ちデッキ)は26~32/32分類できている。
3. 大会エントリー全体では10~20%が未分類扱い、未分類を除いた誤った分類はないに等しい。
4. 個人的な視点だが少し見慣れないカードの積み方をしたリストをはじいているのでそういったリストが出てきていることを私自身が認知しやすい

似通っているデッキタイプが存在しているか、リストが変遷している時期か、サンプルデッキ数が確保できているか、によって精度が影響を受けやすい。
<br>
現在行っている方法の閾値は0.7から0.8あたりが考慮範囲。JCG 23rd Vol.1とVol.3をサンプルデッキとしてVol.4を分類したところ916デッキ中、未分類は81デッキ(閾値0.8)、33デッキ(閾値0.7)であった。誤分類らしきものは閾値0.7の時にベレロフォンを積んでいない3デッキが回復Bに分類されていた(Vol.1,Vol.3では未分類のまま対処していた)ことのみであった。
<br>
前期に比べて似通ったデッキタイプがない(前期の人形Nm,共鳴Nm,共鳴人形Nm)分閾値は低くても大きな問題はなさそう。
<br>
しかし閾値を下げることは似通ったデッキタイプが生まれやすくなるということであり要素による分類により注力することを意味するので目的に合わせて考える必要があると思う。



