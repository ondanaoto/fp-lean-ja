```markdown
# 構造体

プログラムを書く際の最初のステップは、問題領域の概念を特定し、それらをコードで適切に表現することです。
時には、領域の概念が他の、よりシンプルな概念の集まりである場合があります。
その場合、これらのシンプルなコンポーネントを一つの"パッケージ"にまとめて、意味のある名前を付けることが便利です。
Leanでは、C言語やRustの`struct`、C#の`record`に相当する _構造体_ を使ってこれを実現します。

構造体を定義すると、Leanに他のどのタイプにも還元できない完全に新しいタイプが導入されます。
これは便利です。なぜなら、同じデータを含んでいても異なる概念を表す複数の構造体が存在するかもしれないからです。
例えば、ポイントはカルテシアン座標または極座標を使って表されるかもしれません。どちらも浮動小数点数のペアで表されます。
別々の構造体を定義することで、APIの利用者が一方を他方と混同することを防ぐことができます。

Leanの浮動小数点数の型は`Float`と呼ばれ、浮動小数点数は通常の表記法で記述されます。
```lean
{{#example_in Examples/Intro.lean onePointTwo}}
```
```output info
{{#example_out Examples/Intro.lean onePointTwo}}
```
```lean
{{#example_in Examples/Intro.lean negativeLots}}
```
```output info
{{#example_out Examples/Intro.lean negativeLots}}
```
```lean
{{#example_in Examples/Intro.lean zeroPointZero}}
```
```output info
{{#example_out Examples/Intro.lean zeroPointZero}}
```
小数点を含む浮動小数点数を書くとき、Leanは型`Float`を推論します。小数点を含まない場合、型注釈が必要になるかもしれません。
```lean
{{#example_in Examples/Intro.lean zeroNat}}
```
```output info
{{#example_out Examples/Intro.lean zeroNat}}
```

```lean
{{#example_in Examples/Intro.lean zeroFloat}}
```
```output info
{{#example_out Examples/Intro.lean zeroFloat}}
```


カルテシアンポイントは、`x`と`y`という名前の2つの`Float`フィールドを持つ構造体です。
これは`structure`キーワードを使って宣言されます。

```lean
{{#example_decl Examples/Intro.lean Point}}
```

この宣言の後、`Point`は新しい構造体タイプとなります。
最後の行は`deriving Repr`と記述されており、型`Point`の値を表示するためのコードの生成をLeanに依頼しています。
このコードは、プログラマーによる評価結果のレンダリングに`#eval`によって使われます。これはPythonの`repr`関数に類似しています。
また、コンパイラが生成した表示コードをオーバーライドすることも可能です。

構造体タイプの値を作成する典型的な方法は、中括弧内にすべてのフィールドの値を指定することです。
カルテシアン平面の原点は`x`と`y`が共にゼロのところです：

```lean
{{#example_decl Examples/Intro.lean origin}}
```

もし`Point`の定義で`deriving Repr`行が省略されたら、`{{#example_in Examples/Intro.lean PointNoRepr}}`を試みると、関数の引数を省略した場合に発生するエラーに似たエラーが発生します：
```output error
{{#example_out Examples/Intro.lean PointNoRepr}}
```
そのメッセージは、評価機構が評価結果をユーザーに伝える方法を知らないことを意味しています。

幸いにも、`deriving Repr`により、`{{#example_in Examples/Intro.lean originEval}}`の結果は`origin`の定義に非常によく似ています。
```output info
{{#example_out Examples/Intro.lean originEval}}
```

構造体はデータの集まりを"束ねて"、名前を付けて一つの単位として扱うために存在するので、構造体から個々のフィールドを抽出することも重要です。
これはC言語、Python、Rustのようなドット記法を使用して行われます。

```lean
{{#example_in Examples/Intro.lean originx}}
```
```output info
{{#example_out Examples/Intro.lean originx}}
```

```lean
{{#example_in Examples/Intro.lean originy}}
```
```output info
{{#example_out Examples/Intro.lean originy}}
```

これを使用して、構造体を引数として取る関数を定義することができます。
たとえば、ポイントの加算は、基礎となる座標値を加えることによって行われます。
`{{#example_in Examples/Intro.lean addPointsEx}}`は
```output info
{{#example_out Examples/Intro.lean addPointsEx}}
```
となるべきです。
関数自体は、`p1`と`p2`と呼ばれる2つの`Points`を引数として取ります。
結果として得られるポイントは、`p1`と`p2`の`x`と`y`フィールドに基づいています：
```lean
{{#example_decl Examples/Intro.lean addPoints}}
```

同様に、2つのポイント間の距離は、それぞれの`x`と`y`成分の差の二乗の合計の平方根で表されます。これは以下のように記述されます：
```lean
{{#example_decl Examples/Intro.lean distance}}
```
たとえば、(1, 2)と(5, -1)の距離は5です：
```lean
{{#example_in Examples/Intro.lean evalDistance}}
```
```output info
{{#example_out Examples/Intro.lean evalDistance}}
```


複数の構造体は、同じ名前のフィールドを持つことがあります。
たとえば、3次元ポイントのデータ型は`x`と`y`のフィールドを共通して持ち、同じフィールド名でインスタンス化されるかもしれません：
```lean
{{#example_decl Examples/Intro.lean Point3D}}

{{#example_decl Examples/Intro.lean origin3D}}
```
つまり、中括弧構文を使用するためには構造体の期待されるタイプがわかっている必要があります。
タイプが不明な場合、Leanは構造体をインスタンス化することができません。
たとえば、
```lean
{{#example_in Examples/Intro.lean originNoType}}
```
は次のエラーにつながります
```output error
{{#example_out Examples/Intro.lean originNoType}}
```

通常どおり、状況は型注釈を提供することで解決されます。
```lean
{{#example_in Examples/Intro.lean originWithAnnot}}
```
```output info
{{#example_out Examples/Intro.lean originWithAnnot}}
```

プログラムをより簡潔にするために、Leanは中括弧の中で構造体の型注釈を許可しています。
```lean
{{#example_in Examples/Intro.lean originWithAnnot2}}
```
```output info
{{#example_out Examples/Intro.lean originWithAnnot2}}
```

## 構造体の更新

`Point`の`x`フィールドを`0.0`に置き換える関数`zeroX`を想像してください。
ほとんどのプログラミング言語コミュニティでは、この文章は`x`を指しているメモリ位置が新しい値に上書きされることを意味します。
しかし、Leanには変更可能な状態がありません。
関数型プログラミングコミュニティでは、この種の表現によってほとんど常に意味されることは、新しい`Point`がアロケートされて`x`フィールドが新しい値を指し、他のすべてのフィールドが入力からの元の値を指すことです。
`zeroX`を文字通りに記述する一つの方法は、`x`に新しい値を記入し、`y`を手動で転送することです：
```lean
{{#example_decl Examples/Intro.lean zeroXBad}}
```
しかし、このスタイルのプログラミングには欠点があります。
まず、構造体に新しいフィールドが追加された場合、どのフィールドを更新する場所でも更新する必要があり、メンテナンスが難しくなります。
第二に、構造体が同じタイプの複数のフィールドを含んでいる場合、コピーアンドペーストのコーディングによってフィールドの内容が複製されたり切り替えられたりするリスクがあります。
最後に、プログラムが長くて官僚的になります。

Leanは構造体のいくつかのフィールドを置き換える一方で他をそのままにしておくための便利な構文を提供しています。
これは構造体の初期化で`with`キーワードを使用することにより行われます。
変更されないフィールドのソースは`with`の前に、新しいフィールドがその後に配置されます。
例えば、`zeroX`は新しい`x`値だけで記述することができます：

```lean
{{#example_decl Examples/Intro.lean zeroX}}
```

この構造体の更新構文は既存の値を変更しないことを忘れないでください。それは新しい値を作り出し、古い値といくつかのフィールドを共有します。
例えば、ポイント`fourAndThree`があるとします：
```lean
{{#example_decl Examples/Intro.lean fourAndThree}}
```
それを評価してから、`zeroX`を使ってそれを更新してから、もう一度評価すると、元の値が返されます：
```lean
{{#example_in Examples/Intro.lean fourAndThreeEval}}
```
```output info
{{#example_out Examples/Intro.lean fourAndThreeEval}}
```
```lean
{{#example_in Examples/Intro.lean zeroXFourAndThreeEval}}
```
```output info
{{#example_out Examples/Intro.lean zeroXFourAndThreeEval}}
```
```lean
{{#example_in Examples/Intro.lean fourAndThreeEval}}
```
```output info
{{#example_out Examples/Intro.lean fourAndThreeEval}}
```

構造体の更新が元の構造を変更しないという事実は、古いものから新しい値を計算する場合に推論を容易にします。
古い構造体を参照するすべての参照は、提供された新しい値のすべてで同じフィールド値を示し続けます。

## 裏側での動作

すべての構造体には _コンストラクタ_ があります。
ここで、「コンストラクタ」という用語は混乱の元になるかもしれません。
JavaやPythonなどの言語でのコンストラクタとは異なり、Leanのコンストラクタはデータタイプが初期化される際に実行される任意のコードではありません。
コンストラクタは単に新しくアロケートされたデータ構造に格納されるデータを集めるだけです。
データを前処理したり無効な引数を拒否するカスタムコンストラクタを提供することはできません。
これは本当に、二つの文脈で「コンストラクタ」という語が異なるが関連する意味を持つというケースです。

デフォルトでは、`S`という名前の構造体のコンストラクタは`S.mk`という名前になります。
ここで、`S`は名前空間の修飾子であり、`mk`はコンストラクタ自体の名前です。
中括弧初期化構文の代わりに、コンストラクタは直接適用することができます。
```lean
{{#example_in Examples/Intro.lean checkPointMk}}
```
しかし、これは一般的に良いLeanのスタイルとは考えられておらず、Leanは標準の構造体初期化構文を使用してフィードバックを返すことさえあります。
```output info
{{#example_out Examples/Intro.lean checkPointMk}}
```

コンストラクタには関数タイプがあるため、関数が期待されるどこでも使用することができます。
例えば、`Point.mk`は2つの`Float`（それぞれ`x`と`y`に対応する）を受け取り、新しい`Point`を返します。
```lean
{{#example_in Examples/Intro.lean Pointmk}}
```
```output info
{{#example_out Examples/Intro.lean Pointmk}}
```
構造体のコンストラクタ名を上書きするには、冒頭に二つのコロンを使って書きます。
たとえば、`Point.mk`の代わりに`Point.point`を使用するには、次のように書きます：
```lean
{{#example_decl Examples/Intro.lean PointCtorName}}
```

コンストラクタに加えて、構造体の各フィールドに対してアクセサ関数が定義されます。
これらは構造体の名前空間の中でフィールドと同じ名前を持っています。
`Point`については、アクセサ関数`Point.x`と`Point.y`が生成されます。
```lean
{{#example_in Examples/Intro.lean Pointx}}
```
```output info
{{#example_out Examples/Intro.lean Pointx}}
```

```lean
{{#example_in Examples/Intro.lean Pointy}}
```
```output info
{{#example_out Examples/Intro.leanPointy}}
```

ここで、実際には、構造体の構築構文で使用される中括弧構文は、背後では構造体のコンストラクターを呼び出す形に変換されます。先の`addPoints`の定義における`p1.x`という構文も、`Point.x`アクセス関数を呼び出す形に変換されます。つまり、`{{#example_in Examples/Intro.lean originx}}`と`{{#example_in Examples/Intro.lean originx1}}`の両方が同じ結果を生成します。
```output info
{{#example_out Examples/Intro.lean originx1}}
```

アクセスドット記法は、単純な構造体のフィールド以外にも使えます。任意の引数を取る関数にも使えるのです。より一般的には、アクセス記法は`TARGET.f ARG1 ARG2 ...`の形を取ります。`TARGET`が`T`型の場合、名前が`T.f`の関数が呼び出されます。`TARGET`は`T`型の最も左の引数になりますが、これは必ずしも最初の引数とは限りません。そして、残りの引数`ARG1 ARG2 ...`は順に提供されます。例えば、`String.append`は、`String`が`append`フィールドを有する構造体ではなくても、ドット記法を用いて文字列から呼び出すことができます。
```lean
{{#example_in Examples/Intro.lean stringAppendDot}}
```
```output info
{{#example_out Examples/Intro.lean stringAppendDot}}
```
その例では、`TARGET`は`"one string"`を表し、`ARG1`は`" and another"`を表します。

関数`Point.modifyBoth`（つまり`Point`名前空間で定義された`modifyBoth`）は`Point`の両方のフィールドに関数を適用します。
```lean
{{#example_decl Examples/Intro.lean modifyBoth}}
```
`Point`引数が関数引数の後に来ていても、ドット記法で使用することが可能です。
```lean
{{#example_in Examples/Intro.lean modifyBothTest}}
```
```output info
{{#example_out Examples/Intro.lean modifyBothTest}}
```
このケースでは、`TARGET`は`fourAndThree`を表し、`ARG1`は`Float.floor`です。アクセスドット記法のターゲットは必ずしも最初の引数ではなく、型が一致する最初の引数として使われます。

## 練習問題

 * `Float`型の高さ、幅、奥行きを持つ直方体を表す`RectangularPrism`という名前の構造体を定義してください。
 * 直方体の体積を計算する`volume : RectangularPrism → Float`という名前の関数を定義してください。
 * 端点によって線分を表す`Segment`という名前の構造体を定義し、線分の長さを計算する`length : Segment → Float`という関数を定義してください。`Segment`には最大で二つのフィールドを持つべきです。
 * `RectangularPrism`の宣言によって導入される名前は何ですか？
 * 次の`Hamster`および`Book`の宣言によって導入される名前は何ですか？それぞれの型は何ですか？

```lean
{{#example_decl Examples/Intro.lean Hamster}}
```

```lean
{{#example_decl Examples/Intro.lean Book}}
```