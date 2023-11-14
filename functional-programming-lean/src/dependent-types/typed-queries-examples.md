# 実例: 型指定されたクエリ

インデックス付きファミリーは、他の言語に似せて設計されることになるAPIの構築に非常に有用です。
これらは、不正なHTMLを生成することができないHTMLコンストラクタのライブラリを書くため、設定ファイルフォーマットの特定のルールをエンコードするため、または複雑なビジネスの制約をモデル化するために使用できます。
このセクションでは、Leanを使用して関係代数のサブセットをインデックス付きファミリを使ってエンコードする方法、およびそれがより強力なデータベースクエリ言語の構築に使用することができるテクニックを簡単にデモンストレーションする内容を説明します。

このサブセットでは、フィールド名の非交差性のような要件を実施するために型システムを使用し、スキーマを値の型に反映させるために型レベルの計算を使用します。
しかしながら、このシステムは現実的ではありません - データベースはリンクリストのリンクリストとして表され、型システムはSQLのそれよりもずっと単純であり、関係代数の演算子は本当にSQLのそれには一致しません。
しかし、有用な原則やテクニックを示すのに十分な大きさです。

## データの宇宙
この関係代数では、列に含まれることができる基本データは、型 `Int`、`String`、`Bool` となり、これらは宇宙 `DBType` によって記述されます：
```lean
{{#example_decl Examples/DependentTypes/DB.lean DBType}}
```

`asType` を使用すると、これらのコードが型のために使用されることができます。
例えば：
```lean
{{#example_in Examples/DependentTypes/DB.lean mountHoodEval}}
```
```output info
{{#example_out Examples/DependentTypes/DB.lean mountHoodEval}}
```

形容されたデータベースのタイプのいずれかの値について平等を比較することが可能です。
しかし、これをLeanに説明するには少し手間がかかります。
単に `BEq` を直接使用すると失敗します：
```lean
{{#example_in Examples/DependentTypes/DB.lean dbEqNoSplit}}
```
```output info
{{#example_out Examples/DependentTypes/DB.lean dbEqNoSplit}}
```
ネストされたペアの宇宙と同様に、型クラス検索は自動的に `t` の値の各可能性をチェックしません
解決策は、`x` および `y` の型を洗練させるためにパターンマッチングを使用することです：
```lean
{{#example_decl Examples/DependentTypes/DB.lean dbEq}}
```
この関数のバージョンでは、`x` および `y` はそれぞれのケースにおいて `Int`、`String`、`Bool` の型を持ち、これらの型はすべて `BEq` インスタンスを持っています。
`dbEq` の定義は、コードによって記述された型の `BEq` インスタンスを定義するために使用することができます：
```lean
{{#example_decl Examples/DependentTypes/DB.lean BEqDBType}}
```
これはコード自体のインスタンスと同じではありません：
```lean
{{#example_decl Examples/DependentTypes/DB.lean BEqDBTypeCodes}}
```
前者のインスタンスでは、コードによって記述された型から描画された値を比較することを可能にしますが、後者はコード自体の比較を可能にします。

`Repr` インスタンスは、同じテクニックを使用して書くことができます。
`Repr` クラスのメソッドは `reprPrec` と呼ばれるのは、値を表示するときに演算子の優先順位のようなものを考慮に入れるように設計されているためです。
型を依存パターンマッチングによって洗練させることによって、`Int`、`String`、`Bool` の `Repr` インスタンスの `reprPrec` メソッドを使用することができます：
```lean
{{#example_decl Examples/DependentTypes/DB.lean ReprAsType}}
```

## スキーマとテーブル

スキーマは、データベースの各列の名前とタイプを記述します：
```lean
{{#example_decl Examples/DependentTypes/DB.lean Schema}}
```
実際には、スキーマは表の行を記述する宇宙と見なすことができます。
空のスキーマは単位型を記述し、単一列を持つスキーマはその値を単独で記述し、少なくとも二つの列を持つスキーマはタプルによって表されます：
```lean
{{#example_decl Examples/DependentTypes/DB.lean Row}}
```

[多様体型についての初期セクション](../getting-to-know/polymorphism.md#Prod)で説明されているように、Leanの積型とタプルは右結合です。
これはネストされたペアが普通のフラットタプルと等価であることを意味します。

テーブルは、共有スキーマを持つ行のリストです：
```lean
{{#example_decl Examples/DependentTypes/DB.lean Table}}
```
例えば、山の頂上への訪問の日記は、スキーマ `peak` を用いて表されます：
```lean
{{#example_decl Examples/DependentTypes/DB.lean peak}}
```
この書籍の著者による、訪問された頂上の選抜は通常のタプルリストとして現れます：
```lean
{{#example_decl Examples/DependentTypes/DB.lean mountainDiary}}
```
別の例は、滝とそれらへの訪問の日記で構成されています：
```lean
{{#example_decl Examples/DependentTypes/DB.lean waterfall}}

{{#example_decl Examples/DependentTypes/DB.lean waterfallDiary}}
```

### 再帰と宇宙、再訪

行をタプルとして便利に構成するという事実には、コストが伴います：二つのベースケースを別々に扱うという `Row` の事実は、`Row` を使用する型で定義され、コード（つまり、スキーマ）上で再帰的に定義された関数は同じ区別を行う必要があります。
ここで問題になるケースの一つは、スキーマ上での再帰を使用して行の平等性を検証する関数を定義する平等チェックです。
この例はLeanの型チェッカーを通過しません：
```lean
{{#example_in Examples/DependentTypes/DB.lean RowBEqRecursion}}
```
```output error
{{#example_out Examples/DependentTypes/DB.lean RowBEqRecursion}}
```
問題は、パターン `col :: cols` が行の型を十分に洗練していないためです。
これは、Leanがまだ `Row` の定義の `[col]` シングルトンパターンまたは `col1 :: col2 :: cols` パターンのどちらが一致したかを判断できないため、`Row` への呼び出しはペア型に縮小しないためです。
解決策は、`Row` の構造を `Row.bEq` の定義に反映することです：
```lean
{{#example_decl Examples/DependentTypes/DB.lean RowBEq}}
```

他のコンテキストとは異なり、型に出現する関数は、その入力/出力動作のみの観点からのみ考慮されるわけではありません。
これらの型を使用するプログラムは、その構造が型レベル関数で使用されるパターンマッチングと再帰的行動と一致するように強制されます。
依存型プログラミングの大きな部分は、適切な型レベルの関数を選択し、それに正しい計算動作をすることです。

### 列ポインタ

いくつかのクエリは、ある特定の列がスキーマに含まれている場合にのみ意味があります。
例えば、高度が1000メートル以上の山を返すクエリは、整数を含む `"elevation"` 列があるスキーマのコンテキストでのみ意味があります。
スキーマが特定の列を含むことを示す一つの方法は、それに直接ポインタを提供することであり、ポインタをインデックス付きファミリとして定義することで無効なポインタを除外することが可能です。

列がスキーマに存在するには二つの方法があります：それはスキーマの始まりにあるか、それ以降のスキーマのどこかにあります。
最終的に、列がスキーマの後半にある場合、それはスキーマのいくつかの尾部の始まりになります。

インデックス付きファミリ `HasCol` は、仕様をLeanコードに翻訳したものです：
```lean
{{#example_decl Examples/DependentTypes/DB.lean HasCol}}
```
ファミリーの三つの引数はスキーマ、列名、およびそのタイプです。
すべてがインデックスですが、列名とタイプの後にスキーマを置くために引数を並べ替えることで、名前とタイプをパラメータにすることができます。
コンストラクタ `here` は、スキーマが望ましい名前とタイプを持つ列 `⟨name, t⟩` で始まるときに使用することができます。これはスキーマの最初の列へのポインタであり、最初の列が望ましい名前とタイプを持っているときにのみ使用できます。
コンストラクタ `there` は、より小さいスキーマへのポインタを一つの列をそれに付け加えたスキーマへのポインタに変換します。

`"elevation"` が `peak` の三番目の列であるため、`there` を使って最初の二つの列を過ぎて探すことができます。その後、それは最初の列です。

Continue translating...