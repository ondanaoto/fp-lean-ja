# 追加の便利機能

## パイプ演算子

通常、関数はその引数の前に記述されます。
左から右にプログラムを読んでいくと、関数の_出力_が最も重要視されるようになります。つまり、関数には達成すべき目標（つまり計算すべき値）があり、そのプロセスを支援するために引数が与えられます。
しかし、入力が順次洗練されて出力が生成されることについて考えるほうが理解しやすいプログラムもあります。
このような場合には、LeanはF#に似た_パイプライン_演算子を提供します。
パイプライン演算子は、Clojureのスレッドマクロと同じ状況で便利です。

パイプライン `{{#example_in Examples/MonadTransformers/Conveniences.lean pipelineShort}}` は `{{#example_out Examples/MonadTransformers/Conveniences.lean pipelineShort}}` の省略形です。
例えば、以下を評価すると:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean some5}}
```
結果は:
```output info
{{#example_out Examples/MonadTransformers/Conveniences.lean some5}}
```
この重点の変更により、プログラムを読む際にさらに便利になることがありますが、パイプラインは多くの構成要素を含むときに真価を発揮します。

以下の定義を使って:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean times3}}
```
次のパイプライン:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean itIsFive}}
```
は以下の結果を出力します:
```output info
{{#example_out Examples/MonadTransformers/Conveniences.lean itIsFive}}
```
一般的に、パイプラインのシリーズ `{{#example_in Examples/MonadTransformers/Conveniences.lean pipeline}}` はネストされた関数適用 `{{#example_out Examples/MonadTransformers/Conveniences.lean pipeline}}` の省略形です。

パイプラインは逆向きにも書けます。
この場合、データ変換の対象を最初に提示するわけではありませんが、読者にとって多くのネストされた括弧が挑戦的である場合には、適用のステップを明確にすることができます。
前の例は、以下のように等価に書けます:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean itIsAlsoFive}}
```
これは以下の省略形です:
```lean
{{#example_in Examples/MonadTransformers/Conveniences.lean itIsAlsoFiveParens}}
```

演算子の後の名前空間を解決するためにドットの前の型の名前を使用するLeanのメソッドドット記法はパイプラインと同様の目的で機能します。
パイプライン演算子がなくても `{{#example_in Examples/MonadTransformers/Conveniences.lean listReverse}}` の代わりに `{{#example_out Examples/MonadTransformers/Conveniences.lean listReverse}}` と書くことは可能です。
しかしながら、パイプライン演算子は多くのものを使用する際に、ドット付きの関数でも便利です。
`{{#example_in Examples/MonadTransformers/Conveniences.lean listReverseDropReverse}}` は `{{#example_out Examples/MonadTransformers/Conveniences.lean listReverseDropReverse}}` としても書けます。
このバージョンは、単に引数を受け入れるために式を括弧で囲む必要がなく、KotlinやC#といった言語でのメソッド呼び出しの鎖の利便性を取り戻します。
しかし、名前空間は手動で提供する必要があります。
最後の便利機能として、Leanは「パイプラインドット」演算子を提供しており、これはパイプラインと同様に関数をグループ化しますが、名前空間を解決するために型の名前を使用します。
「パイプラインドット」を使うと、例は `{{#example_out Examples/MonadTransformers/Conveniences.lean listReverseDropReversePipe}}` に書き換えられます。

## 無限ループ

`do`ブロック内で、`repeat`キーワードは無限ループを導入します。
例えば、文字列 `"Spam!"` をスパムするプログラムはそれを使用できます:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean spam}}
```
`repeat`ループは、`for`ループのように `break` と `continue` をサポートしています。

[`feline`の実装](../hello-world/cat.md#streams)の `dump` 関数は、永遠に実行される再帰関数を使用しています:
```lean
{{#include ../../../examples/feline/2/Main.lean:dump}}
```
この関数は `repeat` を使用することで大幅に短縮できます:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean dump}}
```

`spam` と `dump` のどちらも `partial` として宣言する必要はありません。これらはそれ自体が無限再帰ではないためです。
代わりに、`repeat` は `ForM` インスタンスが `partial` である型を利用しています。
部分性は呼び出し関数に「感染」しません。

## Whileループ

局所的な可変性を伴うプログラミングにおいて、`while`ループは `if`-ガードされた `break` を伴う `repeat` の便利な代替手段です:
```lean
{{#example_decl Examples/MonadTransformers/Conveniences.lean dumpWhile}}
```
舞台裏では、`while` は単に `repeat` のより簡単な表記に過ぎません。