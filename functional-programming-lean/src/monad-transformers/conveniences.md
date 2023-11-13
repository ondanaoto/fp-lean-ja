# 追加の便利機能

## パイプ演算子

通常、関数はその引数の前に記述されます。
左側から右側にプログラムを読み進める場合、関数の_出力_が最も重要になります。つまり、関数には達成すべき目標（つまり計算すべき値）があり、そのプロセスを支援するために引数が与えられます。
しかし、入力が順次精錬され出力が生成されるという考え方の方が理解しやすいプログラムもあります。
このような場合、LeanはF#と似た_パイプライン_演算子を提供しています。
パイプライン演算子は、Clojureのスレッドマクロが便利とされるのと同様の状況で役立ちます。

パイプライン `{{#example_in Examples/MonadTransformers/Conveniences.lean pipelineShort}}` は `{{#example_out Examples/MonadTransformers/Conveniences.lean pipelineShort}}` の省略形です。
例えば、以下を評価すると:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean some5}}
```
結果は以下のようになります:
```output info
{{#example_out Examples/MonadTransformers/Conveniences.lean some5}}
```
このフォーカスの変更は、プログラムを読む際の利便性を高めることがありますが、特に多くの要素が含まれる場合にパイプラインが真価を発揮します。

以下の定義を使用して:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean times3}}
```
次のパイプラインを使うと:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean itIsFive}}
```
以下の結果が出力されます:
```output info
{{#example_out Examples/MonadTransformers/Conveniences.lean itIsFive}}
```
一般的に、パイプラインのシリーズ `{{#example_in Examples/MonadTransformers/Conveniences.lean pipeline}}` はネストされた関数適用 `{{#example_out Examples/MonadTransformers/Conveniences.lean pipeline}}` の省略形です。

パイプラインは逆向きにも書けます。
この場合、データ変換の対象を最初に提示するわけではありませんが、読者にとってネストされた括弧が挑戦的になる時に、適用されるステップを明瞭にできます。
上記の例は、以下のように書き換えられます:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean itIsAlsoFive}}
```
これは以下の省略形です:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean itIsAlsoFiveParens}}
```

演算子の後ろでの名前空間を解決するためにドットの前にある型名を使用するLeanのメソッドドット記法は、パイプラインと同様の目的で機能します。
パイプライン演算子なしで `{{#example_in Examples/MonadTransformers/Conveniences.lean listReverse}}` の代わりに `{{#example_out Examples/MonadTransformers/Conveniences.lean listReverse}}` と書くことは可能です。
しかし、パイプライン演算子は多くの関数を使用する際に、ドット付き関数においても便利です。
`{{#example_in Examples/MonadTransformers/Conveniences.lean listReverseDropReverse}}` は `{{#example_out Examples/MonadTransformers/Conveniences.lean listReverseDropReverse}}` としても書けます。
このバージョンでは、単に引数を受け入れるために式を括弧で囲む必要がなく、KotlinやC#といった言語でのメソッドチェーンの利便性を取り戻します。
しかし、名前空間は手動で提示しなければなりません。
最後の便利機能として、Leanは「パイプラインドット」演算子を提供しています。これはパイプラインと同様に関数をグループ化しますが、名前空間を解決するために型名を使用します。
「パイプラインドット」を使うと、例は `{{#example_out Examples/MonadTransformers/Conveniences.lean listReverseDropReversePipe}}` に書き換えられます。

## 無限ループ

`do` ブロック内で、`repeat` キーワードは無限ループを生成します。
例として、文字列 `"Spam!"` を無限に表示するプログラムはそれを使って実現できます:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean spam}}
```
`repeat` ループは、`break` や `continue` をサポートする点で `for` ループと同様です。

[`feline` 実装](../hello-world/cat.md#streams)の `dump` 関数は、無限に実行される再帰関数を使用しています:
```lean
{{#include ../../../examples/feline/2/Main.lean:dump}}
```
この関数は `repeat` を利用することでより簡潔に記述することができます:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean dump}}
```

`spam` と `dump` のどちらも `partial` と宣言する必要はありません。これら自体が無限再帰ではないためです。
代わりに、`repeat` は `ForM` インスタンスが `partial` となる型を利用しています。
部分性は呼び出す側の関数に「感染」しません。

## Whileループ

局所的な可変性を伴うプログラミングにおいて、`if`-ガードされた `break` を使用する `repeat` の便利な代替として `while` ループがあります:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean dumpWhile}}
```
舞台裏では、`while` は単に `repeat` のより簡潔な書き方です。