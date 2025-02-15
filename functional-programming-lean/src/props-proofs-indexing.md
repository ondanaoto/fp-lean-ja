# 休憩：命題、証明、及び添字付け

多くの言語同様、Leanでは配列やリストへの添字付けに角括弧を使用します。
例えば、`woodlandCritters`が下記のように定義されている場合：
```lean
{{#example_decl Examples/Props.lean woodlandCritters}}
```
その個々の要素を取り出すことができます：
```lean
{{#example_decl Examples/Props.lean animals}}
```
しかし、４番目の要素を取り出そうとすると、実行時エラーではなくコンパイル時エラーが発生します：
```lean
{{#example_in Examples/Props.lean outOfBounds}}
```
```output error
{{#example_out Examples/Props.lean outOfBounds}}
```
このエラーメッセージは、Leanが`3 < List.length woodlandCritters`であることを自動的に証明しようとしたができなかったことを意味しています。つまり、インデックス参照が安全という証明です。範囲外エラーは一般的なバグの一種で、Leanはプログラミング言語及び定理証明者としての二重の性質を活かし、これらを可能な限り排除しようとしています。

これがどのように機能するかを理解するため、命題、証明、及び戦術という３つの重要な概念を理解する必要があります。

## 命題と証明

_命題_ は、真か偽かのどちらか一方である文です。
以下はすべて命題です：

* 1 + 1 = 2
* 加法は可換である
* 素数は無限に存在する
* 1 + 1 = 15
* パリはフランスの首都である
* ブエノスアイレスは韓国の首都である
* すべての鳥は飛ぶことができる

一方、意味不明の文は命題ではありません。
以下はどれも命題ではありません：

* 1 + 緑 = アイスクリーム
* すべての首都は素数である
* 少なくとも一つのゴーグがフリープである

命題には２種類あります。一つは純粋に数学的で、概念の定義のみに依存するものです。もう一つは世界についての事実に基づくものです。
Leanのような定理証明者は、前者のカテゴリに関心があり、ペンギンの飛翔能力や都市の法的地位については言及しません。

_証明_ は、ある命題が真であることを納得させる議論です。
数学的命題に対するこれらの議論は関連する概念の定義と論理的な議論のルールを使用して行われます。
ほとんどの証明は人が理解するために書かれており、多くの細かい詳細を省いています。
Leanのようなコンピュータ支援定理証明者は、数学者が証明を書く時に多くの細部を省くことを可能にするようデザインされており、そのソフトウェアは欠けている明示的なステップを補います。
これにより、見落としやミスの可能性が減ります。

Leanでは、プログラムの型は、それをどのように操作できるかを記述します。
例えば、`Nat → List String`という型のプログラムは、`Nat`の引数を取って文字列のリストを生成する関数です。
言い換えれば、それぞれの型はその型のプログラムとして何が数え上げられるかを指定します。

Leanでは、命題も実際には型です。
それらはその宣言が真であるとする証拠として何が数え上げられるかを指定します。
その証拠を提示することで命題は証明されます。
一方で、命題が偽である場合、その証拠を構築することは不可能です。

例えば、「1 + 1 = 2」という命題はLeanで直接書き表すことができます。
この命題の証拠はコンストラクタ`rfl`であり、これは_反射律_を意味します：
```lean
{{#example_decl Examples/Props.lean onePlusOneIsTwo}}
```
一方で、`rfl`は偽の命題「1 + 1 = 15」を証明することはできません：
```lean
{{#example_in Examples/Props.lean onePlusOneIsFifteen}}
```
```output error
{{#example_out Examples/Props.lean onePlusOneIsFifteen}}
```
このエラーメッセージは、「1 + 1」が直接「2」に評価されるため、`rfl`は二つの式が等しいことを証明できるのは両辺が同じ数である時だけであることを示しています。これにより、`onePlusOneIsTwo`が受け入れられることになります。
ちょうど`Type`が`Nat`、`String`、`List (Nat × String × (Int → Float))`などのデータ構造と関数を表す型を記述するように、`Prop`は命題を記述します。

命題が証明されたとき、それは_定理_と呼ばれます。
Leanでは、定理を宣言する場合、`def`の代わりに`theorem`キーワードで宣言するのが一般的です。
これにより、読者にとって、どの宣言が数学的証明で読むべきものであり、どの宣言が定義で読むべきかを識別しやすくなります。
一般に、証明では命題が真である証拠があることが重要で、提供されたのが_どの_証拠であるかはあまり重要ではありません。  
一方、定義では、明らかに間違った加法の定義は常に0を返すので、選ばれた値が非常に重要です。

上記の例は次のように書き換えることができます：
```lean
{{#example_decl Examples/Props.lean onePlusOneIsTwoProp}}
```

## 戦術

戦術は、_戦術_を使用して書かれることが一般的です。直接的な証拠を提供する代わりに戦術を使って証拠を構築します。
戦術は、証明すべき文（_目標（ゴール）_ と呼ばれる）と、それを証明するために利用可能な前提を監視する_証明状態_ で動作します。
戦術をゴールに適用すると、新しい目標（ゴール）が含まれた新たな証明状態になります。
すべての目標（ゴール）が証明された時点で証明は完成です。

戦術で証明を書くためには、定義の始めに`by`を記述します。
`by`を用いると、次のインデントされたブロックの最後までLeanは戦術モードに入ります。
戦術モードでは、Leanは現在の証明状態に関して継続的なフィードバックを提供します。
戦術を用いた`onePlusOneIsTwo`の証明は、まだ非常に短いです：