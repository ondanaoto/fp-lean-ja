# 宇宙デザインパターン

Leanでは、他の型を分類する`Type`、`Type 3`、`Prop`などの型は宇宙として知られています。
しかし、_宇宙_ という用語は、Leanの型のサブセットを表すために使用されるデータ型と、そのデータ型のコンストラクタを実際の型に変換する関数を用いるデザインパターンにも使われます。
このデータ型の値は、それらの型の_コード_ と呼ばれます。

Leanの組み込みの宇宙のように、このパターンで実装された宇宙は、どのように実現されるかのメカニズムは異なるものの、使用可能な型の一部の集合を記述する型です。
Leanには、他の型を直接記述する`Type`、`Type 3`、`Prop`などの型が存在します。
この配置は _Russell流の宇宙_ と呼ばれます。
このセクションで説明されているユーザー定義の宇宙は、その型をすべて_データ_ として表現し、これらのコードを真に実際の型に解釈する明示的な関数を含みます。
この配置は _Tarski流の宇宙_ と呼ばれます。
依存型理論に基づいた言語のようにLeanの場合、通常はRussell流の宇宙が使用されますが、Tarski流の宇宙はこれらの言語でAPIを定義するための有用なパターンです。

カスタム宇宙を定義すると、APIで使用できる型の閉じたコレクションを選り分けることができます。
型のコレクションが閉じているため、コード上での再帰によりプログラムは宇宙の_任意_ の型で機能します。
カスタム宇宙の一例は、`Nat`を意味する`nat`、`Bool`を意味する`bool`といったコードがあります：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean NatOrBool}}
```
コードに対するパターンマッチングにより、型を洗練させることができます。ちょうど`Vect`のコンストラクタに対するパターンマッチングが期待される長さを洗練させるようにです。
例えば、この宇宙の型を文字列からデシリアライズするプログラムは次のように書けます：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean decode}}
```
`t`に対する依存型のパターンマッチングにより、期待される結果型`t.asType`はそれぞれ`NatOrBool.nat.asType`および`NatOrBool.bool.asType`に洗練され、これらは実際の型`Nat`と`Bool`に計算されます。

コードは他のデータと同じように再帰可能です。
型`NestedPairs`は対と自然数の型の任意の可能なネスティングを符号化します：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean NestedPairs}}
```
この場合、解釈関数`NestedPairs.asType`は再帰的です。
これは、宇宙の`BEq`を実装するためにコード上の再帰が必要であることを意味します：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean NestedPairsbeq}}
```

`NestedPairs`宇宙のすべての型にはすでに`BEq`インスタンスがありますが、インスタンス宣言のデータ型の可能性のあるすべてのケースを自動的にチェックすることはありません。なぜなら、`NestedPairs`のように無限に多くのケースがある可能性があるからです。
コード上での再帰に頼らずに直接`BEq`インスタンスに訴えようとすると、エラーが発生します：
```lean
{{#example_in Examples/DependentTypes/Finite.lean beqNoCases}}
```
```output error
{{#example_out Examples/DependentTypes/Finite.lean beqNoCases}}
```
エラーメッセージの`t`は未知の型`NestedPairs`の値を意味します。

## 型クラス vs 宇宙

型クラスでは、必要なインターフェースの実装がある限り、APIでオープンエンドの型コレクションを使用することができます。
ほとんどの場合、これが好ましいです。
APIのすべての使用事例を事前に予測することは困難であり、型クラスはライブラリーコードが元の著者が予想したよりも多くの型で利用されることを可能にする便利な方法です。

一方、Tarski流宇宙は、あらかじめ決められた型のコレクションでのみAPIを使用可能に制限します。
これはいくつかの状況で有用です：
 * どの型が渡されているかによって関数が非常に異なる動作をするべきとき - 型そのものにパターンマッチすることはできませんが、型のコードにパターンマッチングをすることは許されています
 * 外部システムが本質的に提供できるデータの型を限定し、余分な柔軟性が望まれていないとき
 * いくつかの操作の実装以上の型に関する追加の性質が必要なとき

型クラスは、JavaやC#のインターフェースと同じ状況で多くの場合に有用ですが、Tarski流宇宙は封印クラスが使用される場面で有用ですが、通常の帰納的データ型では使用できない状況で有用です。



翻訳している間に、コードブロックとして示されている具体的なLeanのコード例（`{{#example_decl ...}}`、`{{#example_in ...}}`、`{{#example_out ...}}`）、`{{#include ...}}`を含むテキストの部分と実際の出力（`{{#example_eval ...}}`）はダイナミックな内容を示しており、こちらではそれらを具体化または翻訳できませんので、一部の翻訳を省略し、なおかつそれらコード部分の説明は一般的な状態で翻訳します。



## 有限型の宇宙

APIで使用できる型をあらかじめ決められたコレクションに制限することで、オープンエンドのAPIでは不可能な操作が可能になることがあります。例えば。通常は関数を等価比較することはできません。関数は同じ入力に対して同じ出力をマッピングするときに等しいとみなされるべきです。これを確認するには無限の時間がかかる場合があります。なぜなら型`Nat → Bool`の二つの関数を比較するには、それぞれの`Nat`に対して関数が同じ`Bool`を返すかどうかを確認する必要があるからです。

つまり、無限の型からの関数はそれ自体が無限です。関数をテーブルと見なすことができ、無限の引数型を持つ関数には囲む場合が示される各ケースを表すために無限に多くの行が必要です。しかし、有限の型からの関数にはテーブルに必要な行が有限になります。これにより、その関数を有限とすることができます。有限の引数型を持つ二つの関数は、すべての可能な引数を列挙して、それぞれの引数に対して関数を呼び出し、次いで結果を比較することにより、等価性をチェックすることができます。高階関数の等価性をチェックするには、与えられた型のすべての可能な関数を生成する必要があり、さらに各引数型の要素を戻り値型の各要素に写像できるように戻り値型も有限でなければならないことが必要です。これは速い方法ではありませんが、有限の時間で完了します。

一つの方法として有限型を宇宙で表すことができます：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean Finite}}
```
この宇宙では、コンストラクタ`arr`が関数型に対応し、`arr`ow（矢印）で書かれます。

この宇宙の二つの値を等価比較することは、`NestedPairs`宇宙とほぼ同じです。唯一重要な違いは`arr`のケースの追加で、これはヘルパー`Finite.enumerate`を使用して`t1`によってコード化された型からすべての値を生成し、二つの関数がすべての可能な入力に対して等しい結果を返すかどうかをチェックします：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean FiniteBeq}}
```
標準ライブラリ関数`List.all`は、リストのすべてのエントリで提供された関数が`true`を返すかどうかをチェックします。この関数は、ブーリアンの関数を等価比較するために使用することができます：
```lean
{{#example_in Examples/DependentTypes/Finite.lean arrBoolBoolEq}}
```
```output info
{{#example_out Examples/DependentTypes/Finite.lean arrBoolBoolEq}}
```
また標準ライブラリの関数を比較するために使用することができます：
```lean
{{#example_in Examples/DependentTypes/Finite.lean arrBoolBoolEq2}}
```
```output info
{{#example_out Examples/DependentTypes/Finite.lean arrBoolBoolEq2}}
```
関数合成などのツールを使って作成された関数を比較することもできます：
```lean
{{#example_in Examples/DependentTypes/Finite.lean arrBoolBoolEq3}}
```
```output info
{{#example_out Examples/DependentTypes/Finite.lean arrBoolBoolEq3}}
```
これは、`Finite`宇宙がライブラリーで作成された特別なアナローグでなく、Leanの_実際_ の関数型を符号化しているからです。

`enumerate`の実装も`Finite`からのコードに対する再帰によります。
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:FiniteAll}}
```
`Unit`のケースでは、単一の値があります。
`Bool`のケースでは、返す値が二つあります（`true`と`false`）。
ペアのケースでは、結果は`t1`によってコード化される型の値と`t2`によってコード化される型の値のデカルト積になるべきです。
言い換えると、`t1`からのすべての値は`t2`からのすべての値と組にされるべきです。
ヘルパー関数`List.product`は通常の再帰関数で確かに書けますが、ここではアイデンティティモナドでの`for`を使って定義されています：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean ListProduct}}
```
最後に、関数のケースの`Finite.enumerate`は、引数としてターゲットするすべての戻り値のリストを取るヘルパー「`Finite.functions`」へと移譲します。

一般に、ある有限型から結果値のコレクションへのすべての関数を生成することは、その関数のテーブルを生成することと見なすことができます。
各関数は各入力に出力を割り当てるため、与えられた関数は\\(k\\)の可能な引数があるときにそのテーブルに\\(k\\)行があります。
各行が\\(n\\)の可能な出力のいずれかを選択できるため、\\(n^k\\)の潜在的な関数を生成することがあります。

再度、有限型からあるリストの値への関数を生成することは、有限型を記述するコードに対する再帰に依存しています：
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:FiniteFunctionSigStart}}
```

`Unit`からの関数のテーブルには一行含まれます。それは関数が提供される入力に基づいて異なる結果を選択できないからです。
これは各潜在的な入力に対して一つの関数を生成することを意味します。
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:FiniteFunctionUnit}}
```
結果値が\\(n\\)個あるとき、`Bool`からの関数には\\(n^2\\)種類があります。なぜなら各個別の`Bool → α`型の関数は、`Bool`を使って特定の`α`の二つのうちの一つを選択するからです：
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:FiniteFunctionBool}}
```
ペアからの関数を生成することはカリー化を活用することで達成できます。
ペアからの関数は、ペアの最初の要素を受け取り、ペアの二番目の要素を待っている関数を返す関数に変換することができます。
これにより、`Finite.functions`をこの場合に再帰的に使用することを可能にします：
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:FiniteFunctionPair}}
```

高階関数を生成することは少し頭の体操になります。各高階関数は、その引数として関数を取ります。この引数関数は入出力の振る舞いに基づいて他の関数から区別することができます。一般的に、高階関数は引数関数を可能な引数すべてに適用し、その結果に基づいたあらゆる可能な振る舞いを実行できます。これにより高階関数を構築する手段が示唆されます：
 * 引数関数自体のすべての可能な引数のリストから開始します。
 * 可能な各引数について、引数関数を可能な引数に適用した結果の観察に基づいて発生する可能なすべての振る舞いを構築します。これは`Finite.functions`および残りの可能な引数に対する再帰を使用して行うことができます。なぜなら再帰の結果は、残りの可能な引数の観察に基づいた振る舞いを基にした機能を表しているからです。`Finite.functions`は現在の引数に対する観察に基づくこれらの実現方法をすべて構築します。
 * これらの観察に対する潜在的な振る舞いごとに、引数関数を現在の可能な引数に適用する高階関数を構築します。その結果は観察振る舞いに渡されます。
 * 再帰の基底ケースは、各結果値に対して何も観察しない高階関数です―それは引数関数を無視し、単に結果値を返します。

この再帰関数を直接定義することは、Leanが全体の関数が終了することを証明するのが不可能となる原因となります。しかし、より単純な形の再帰である_右折り畳み_を使うことによって、関数が終了することが明確となり終了チェッカーにそれを伝えることができます。右折り畳みは3つの引数を取ります：リストの頭をリストの尾に対する再帰の結果と結びつけるステップ関数、リストが空の時に返すデフォルト値、および処理するリスト。そしてそれはリストを解析し、リスト内の各`::`をステップ関数への呼び出しで置き換え、`[]`をデフォルト値で置き換えることで、基本的には：
```lean
{{#example_decl Examples/DependentTypes/Finite.lean foldr}}
```
リスト内の`Nat`の合計は`foldr`を使って計算できます：
```lean
{{#example_eval Examples/DependentTypes/Finite.lean foldrSum}}
```

`foldr`を使って、高階関数は以下のように作成されます：
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:FiniteFunctionArr}}
```
`Finite.Functions`の完全な定義は次のとおりです：
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:FiniteFunctions}}
```

`Finite.enumerate`と`Finite.functions`は互いに呼び出されるので、`mutual`ブロック内で定義されなければなりません。言い換えれば、`Finite.enumerate`の定義の直前に`mutual`キーワードがあります：
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:MutualStart}}
```
そして`Finite.functions`の定義の直後に`end`キーワードがあります：
```lean
{{#include ../../../examples/Examples/DependentTypes/Finite.lean:MutualEnd}}
```

このアルゴリズムは機能の比較には特に実用的ではありません。チェックするケースの数は指数関数的に増加し、`((Bool × Bool) → Bool) → Bool`のような単純な型でさえ{{#example_out Examples/DependentTypes/Finite.lean nestedFunLength}}個の異なる関数を説明します。なぜこんなに多くありますか？上述の理由に基づいて、型\\( T \\)によって記述される値の数を表すために\\( \\left| T \\right| \\)を使うと、
\\[ \\left| \\left( \\left( \\mathtt{Bool} \\times \\mathtt{Bool} \\right) \\rightarrow \\mathtt{Bool} \\right) \\rightarrow \\mathtt{Bool} \\right| \\]
は
\\[ \\left|\\mathrm{Bool}\\right|^{\\left| \\left( \\mathtt{Bool} \\times \\mathtt{Bool} \\right) \\rightarrow \\mathtt{Bool} \\right| }, \\]
であり、それは
\\[ 2^{2^{\\left| \\mathtt{Bool} \\times \\mathtt{Bool} \\right| }}, \\]
つまり
\\[ 2^{2^4} \\]
または65536です。
ネストされた指数関数は急速に増加し、多くの高階関数が存在します。

## 練習問題

 * `Finite`によって記述される型にある任意の値を文字列に変換する関数を書いてください。関数はそのテーブルとして表されるべきです。
 * `Finite`と`Finite.beq`に空の型`Empty`を追加してください。
 * `Finite`と`Finite.beq`に`Option`を追加してください。