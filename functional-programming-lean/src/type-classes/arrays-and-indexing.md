# 配列とインデックス化

[間奏曲](../props-proofs-indexing.md)では、リスト内のエントリをその位置で検索するためにインデックス表記法を使用する方法について説明しています。
この構文は型クラスによっても制御されており、さまざまな型に対して使用することができます。

## 配列
例えば、Leanの配列はほとんどの目的に対して連結リストよりもずっと効率的です。
Leanでは、`Array α`型は型`α`の値を動的にサイズ変更して保持する配列で、Javaの`ArrayList`、C++の`std::vector`、Rustの`Vec`に似ています。
`List`は`cons`コンストラクタを使用するたびにポインタの間接参照があるのに対して、配列は連続したメモリ領域を占有し、これはプロセッサキャッシュにとってはるかに良いです。
また、配列の中の値を検索するための時間は一定ですが、連結リスト内での検索時間はアクセスされるインデックスに比例します。

純粋関数型言語のLeanでは、データ構造の特定の位置を変更することはできません。
その代わりに、望んだ変更を含むコピーが作成されます。
配列を使用するとき、Leanコンパイラとランタイムには、配列への唯一無二の参照しかない場合に、変更を内部でミューテーションとして実装する最適化が含まれています。

配列はリストに似ていますが、先頭に`#`が付けられています：
```lean
{{#example_decl Examples/Classes.lean northernTrees}}
```
配列の値の数は`Array.size`を使用して見つけることができます。
例えば、`{{#example_in Examples/Classes.lean northernTreesSize}}`は`{{#example_out Examples/Classes.lean northernTreesSize}}`と評価されます。
配列のサイズよりも小さいインデックスについては、リストと同じようにインデックス記法を使用して対応する値を見つけることができます。
つまり、`{{#example_in Examples/Classes.lean northernTreesTwo}}`は`{{#example_out Examples/Classes.lean northernTreesTwo}}`と評価されます。
同様に、コンパイラはインデックスが範囲内であることを証明する必要があり、配列の範囲外の値を検索しようとすると、リストと同じくコンパイル時エラーが発生します。
例えば、`{{#example_in Examples/Classes.lean northernTreesEight}}`は以下の結果になります：
```output error
{{#example_out Examples/Classes.lean northernTreesEight}}
```

## 空でないリスト

空でないリストを表すデータ型は、リストの先頭のためのフィールドと、通常は空かもしれないリストのためのフィールドを持つ構造として定義することができます：
```lean
{{#example_decl Examples/Classes.lean NonEmptyList}}
```
例えば、空でないリスト`idahoSpiders`（アメリカ合衆国アイダホ州に生息するいくつかのクモの種を含む）は、`{{#example_out Examples/Classes.lean firstSpider}}`に4匹の他のクモが続き、合計で5匹のクモから構成されています：
```lean
{{#example_decl Examples/Classes.lean idahoSpiders}}
```

このリストで特定のインデックスの値を再帰的な関数で検索するときは、以下の3つの可能性を考慮する必要があります：
 1. インデックスが`0`の場合、リストの先頭を返すべきです。
 2. インデックスが`n + 1`で尾が空の場合、インデックスは範囲外です。
 3. インデックスが`n + 1`で尾が空ではない場合、関数は尾と`n`に対して再帰的に呼び出すことができます。

例えば、`Option`を返すルックアップ関数は次のように記述することができます：
```lean
{{#example_decl Examples/Classes.lean NEListGetHuh}}
```
パターンマッチの各ケースは、上記の可能性に対応します。
`get?`への再帰呼び出しは定義のネームスペース内で暗黙的に行われるため、`NonEmptyList`のネームスペース修飾子は必要ありません。
この関数を書く別の方法は、インデックスがゼロよりも大きいとき`get?`をリストに対して使うことです：
```lean
{{#example_decl Examples/Classes.lean NEListGetHuhList}}
```

リストが1つのエントリを含んでいる場合、有効なインデックスは`0`のみです。
リストが2つのエントリを含んでいる場合、`0`と`1`の両方が有効なインデックスです。
リストが3つのエントリを含んでいる場合`, `0`, `1`, `2`の全てが有効なインデックスです。
言い換えれば、空でないリストへの有効なインデックスは、リストの長さより厳密に小さい自然数であり、尾の長さ以下です。

インデックスが範囲内であるという定義は、戦術が許容可能なインデックスの証拠を見つけるために数値の不等式を解決できるが、`NonEmptyList.inBounds`という名前については何も知らないので、`abbrev`として記述されるべきです：
```lean
{{#example_decl Examples/Classes.lean inBoundsNEList}}
```
この関数は真または偽であるかもしれない命題を返します。
例えば、`2`は`idahoSpiders`で範囲内ですが、`5`は範囲外です：
```leantac
{{#example_decl Examples/Classes.lean spiderBoundsChecks}}
```
論理否定演算子の優先順位が非常に低いため、`¬idahoSpiders.inBounds 5`は`¬(idahoSpiders.inBounds 5)`と同等です。

この事実は、インデックスが有効であるという証拠が必要であり、そのために`Option`を返す必要がないルックアップ関数を書くために使用できます。これは標準ライブラリ関数がコンパイル時に証拠をチェックするために委任できるバージョンです：
```lean
{{#example_decl Examples/Classes.lean NEListGet}}
```
もちろん、この関数を書くためには、同じ証拠を使用できる標準ライブラリ関数に委任するのではなく、証拠を直接使用するという手法が必要です。
これには、本書の後の部分で説明される証明や命題を扱うための技術が必要です。

## インデックス表記のオーバーロード

コレクション型のインデックス表記は、`GetElem`型クラスのインスタンスを定義することによってオーバーロードすることができます。
柔軟性のため、`GetElem`は4つのパラメータを持っています：
 * コレクションの型
 * インデックスの型
 * コレクションから取り出される要素の型
 * インデックスが範囲内であるとする証拠を決定する関数

要素の型と証拠関数は両方とも出力パラメータです。
`GetElem`には単一のメソッド`getElem`があり、コレクション値、インデックス値、インデックスが範囲内であるという証拠を引数として取り、要素を返します：
```lean
{{#example_decl Examples/Classes.lean GetElem}}
```
 
`NonEmptyList α`の場合、これらのパラメータは以下のとおりです：
 * コレクションは`NonEmptyList α`
 * インデックスは`Nat`型
 * 要素の型は`α`
 * インデックスが範囲内である場合、それは尾の長さ以下である

実際、`GetElem`のインスタンスは直接`NonEmptyList.get`に委任できます：
```lean
{{#example_decl Examples/Classes.lean GetElemNEList}}
```
このインスタンスがあると`NonEmptyList`は`List`と同じくらい便利になります。
`{{#example_in Examples/Classes.lean firstSpider}}`を評価すると、`{{#example_out Examples/Classes.lean firstSpider}}`が得られ、`{{#example_in Examples/Classes.lean tenthSpider}}`はコンパイル時のエラーにつながります：
```output error
{{#example_out Examples/Classes.lean tenthSpider}}
```

`GetElem`型クラスのコレクション型およびインデックス型は入力パラメータであるため、新しい型は既存のコレクションにインデックスを付けるために使用することができます。
正数型`Pos`は、リストにインデックスを付けるのに完全に合理的ですが、最初のエントリを指すことはできません。
以下の`GetElem`インスタンスにより、`Pos`は`Nat`と同じように、リストエントリを見つけるのに便利に使用できます：
```lean
{{#example_decl Examples/Classes.lean ListPosElem}}
```

インデックスが非数値の場合でも、インデックス表記は意味を持ち得ます。
例えば、`Bool`はポイントのフィールドでの選択に使用され、`false`は`x`に、`true`は`y`に対応します：
```lean
{{#example_decl Examples/Classes.lean PPointBoolGetElem}}
```
この場合、すべての`Bool`は有効なインデックスです。
すべての可能な`Bool`が範囲内であるため、証拠は単に真の命題`True`です。