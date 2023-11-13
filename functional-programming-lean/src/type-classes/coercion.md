```markdown
# 強制変換 (Coercions)

数学では、異なる文脈において同じ記号が、ある対象の異なる側面を表すために使用されることが一般的です。
例えば、環が集合が期待される文脈で参照される場合、それは環の根底にある集合が意図されていると理解されます。
プログラミング言語では、一つの型の値を別の型の値に自動的に変換するための規則を持つことが一般的です。
例えば、Javaでは `byte` 型から `int` 型への自動昇格が許され、Kotlinでは null でない型が null を許容する型の文脈で使用されることを許します。

Leanでは、これらの目的は _強制変換 (coercions)_ と呼ばれるメカニズムによって充足されます。
Leanは、ある型の式が別の型が期待される文脈で見出された場合、型エラーを報告する前に式を強制変換しようとします。
JavaやC、Kotlinとは異なり、強制変換は型クラスのインスタンスを定義することで拡張可能です。

## 正の数 (Positive Numbers)

例えば、すべての正の数には自然数が対応します。
以前定義された関数 `Pos.toNat` は、`Pos` を対応する `Nat` に変換します：
```lean
{{#example_decl Examples/Classes.lean posToNat}}
```
型 `{{#example_out Examples/Classes.lean drop}}` の関数 `List.drop` は、リストのプレフィックスを削除します。
しかし、`List.drop` を `Pos` に適用すると型エラーが生じます：
```lean
{{#example_in Examples/Classes.lean dropPos}}
```
```output error
{{#example_out Examples/Classes.lean dropPos}}
```
`List.drop` の著者はそれを型クラスのメソッドにしなかったため、新しいインスタンスを定義することで上書きすることはできません。

型クラス `Coe` は、一つの型から別の型への強制変換の方法を多重に記述します：
```lean
{{#example_decl Examples/Classes.lean Coe}}
```
`Coe Pos Nat` のインスタンスがあれば、先ほどのコードが動作するのに十分です：
```lean
{{#example_decl Examples/Classes.lean CoePosNat}}

{{#example_in Examples/Classes.lean dropPosCoe}}
```
```output info
{{#example_out Examples/Classes.lean dropPosCoe}}
```
`#check` を使用すると、舞台裏で使用されたインスタンス検索の結果が表示されます：
```lean
{{#example_in Examples/Classes.lean checkDropPosCoe}}
```
```output info
{{#example_out Examples/Classes.lean checkDropPosCoe}}
```

## 連鎖強制変換 (Chaining Coercions)

強制変換を検索する際、Leanは小さな強制変換の連鎖から強制変換を組み立てようとします。
例えば、すでに `Nat` から `Int` への強制変換が存在します。
そのインスタンスと `Coe Pos Nat` インスタンスを組み合わせることで、以下のコードが承認されます：
```lean
{{#example_decl Examples/Classes.lean posInt}}
```
この定義は、`Pos` から `Nat` へ、そして `Nat` から `Int` への二つの強制変換を使用しています。

左括弧の両側 `()` は `Unit.unit` のコンストラクタの略です。
`Repr B` インスタンスを導出した後、
```lean
{{#example_in Examples/Classes.lean coercedToBEval}}
```
の結果は次のようになります：
```output info
{{#example_out Examples/Classes.lean coercedToBEval}}
```

`Option` 型は、C#やKotlinのnull許容型と同様に使用できます：`none` コンストラクタは値の不在を表します。
Lean標準ライブラリでは、任意の型 `α` から `Option α` への強制変換を定義しており、その値を `some` でラッピングします。
これにより、`Option` 型はnull許容型と同じように使用できるようになりますが、`some` を省略することができます。
例えば、リストの最後のエントリを見つける関数 `List.getLast?` は、返り値 `x` の周りに `some` を付けずに書くことができます：
```lean
{{#example_decl Examples/Classes.lean lastHuh}}
```
インスタンス検索は強制変換を見つけ、引数を `some` でラッピングする `coe` の呼び出しを挿入します。
これらの強制変換は連鎖することができるため、`Option` のネストされた使用ではネストされた `some` コンストラクタが必要になることはありません：
```lean
{{#example_decl Examples/Classes.lean perhapsPerhapsPerhaps}}
```

強制変換は、プログラムの残りの部分から課せられた型と推論された型の間に不一致が生じたときにのみ自動的にアクティブになります。
他のエラーがある場合には、強制変換はアクティブになりません。
例えば、エラーがインスタンスが不足していることである場合、強制変換は使用されません：
```lean
{{#example_in Examples/Classes.lean ofNatBeforeCoe}}
```
```output error
{{#example_out Examples/Classes.lean ofNatBeforeCoe}}
```

これは、`OfNat` のために使用したい型を手動で示すことで回避できます：
```lean
{{#example_decl Examples/Classes.lean perhapsPerhapsPerhapsNat}}
```
さらに、強制変換は上矢印を使用することで手動で挿入することができます：
```lean
{{#example_decl Examples/Classes.lean perhapsPerhapsPerhapsNatUp}}
```
場合によっては、これを使用することでLeanが正しいインスタンスを見つけることを保証できます。
また、プログラマの意図をより明確にすることもできます。

## 非空リストと依存性のある強制変換 (Non-Empty Lists and Dependent Coercions)

`Coe α β` のインスタンスは、型 `β` が型 `α` からのすべての値を表すことができる値を持っているときに意味をなします。
`Nat` から `Int` への強制変換は理にかなっています。なぜなら型 `Int` はすべての自然数を含んでいるからです。
同様に、非空リストから通常のリストへの強制変換は理にかなっています。なぜなら `List` 型はすべての非空リストを表すことができるからです：
```lean
{{#example_decl Examples/Classes.lean CoeNEList}}
```
これにより、非空リストは `List` API全体で使用できます。

一方で、`List α` から `NonEmptyList α` への `Coe` のインスタンスを書くことは不可能です。なぜなら、空リストを表す非空リストはないからです。
この制限は、_依存性のある強制変換 (dependent coercions)_ と呼ばれる別のバージョンの強制変換を使用することで回避できます。
依存性のある強制変換は、一つの型から別の型への強制変換が、強制される特定の値に依存するときに使用できます。`OfNat`型クラスがオーバーロードされる具体的な`Nat`をパラメータとして取るのと同じように、依存性のある強制変換は、それをパラメータとして取ります：
```lean
{{#example_decl Examples/Classes.lean CoeDep}}
```
これは、特定の値を選択するチャンスです。型クラスの制約を値にさらに課すか、特定のコンストラクタを直接記述するかしてです。
例えば、実際には空でない `List` を `NonEmptyList` に強制変換することができます：
```lean
{{#example_decl Examples/Classes.lean CoeDepListNEList}}
```

## 型への強制変換 (Coercing to Types)

数学では、集合と追加の構造を備えた概念を持つのが一般的です。
例えば、モノイドは集合 _S_ 、_S_ の要素 _s_ 、および _S_ 上の結合的な二項演算子からなります。ここで、_s_ は演算子の左右で中立です。
_S_ はモノイドの「担体集合 (carrier set)」として参照されます。
0と加算を伴う自然数はモノイドを形成します。なぜなら加算は結合的で、任意の数に0を加えると恒等になるからです。
同様に、1と乗算を伴う自然数もモノイドを形成します。
モノイドは関数型プログラミングでも広く使用されています。リスト、空リスト、および追加演算子はモノイドを形成し、文字列、空文字列、および文字列追加も同様です：
```lean
{{#example_decl Examples/Classes.lean Monoid}}
```
モノイドが与えられると、リストのエントリをモノイドの担体集合に変換し、次にモノイドの演算子を使用してそれらを組み合わせる `foldMap` 関数を記述することができます。
モノイドは中立元を持っているので、リストが空の場合には自然な結果が返されます。そして演算子が結合的であるため、関数のクライアントは、再帰関数が左から右、あるいは右から左に要素を組み合わせるかどうかを気にする必要がありません。
```lean
{{#example_decl Examples/Classes.lean firstFoldMap}}
```

モノイドは3つの異なる情報から成るものですが、モノイドの名前だけを使ってその集合を参照するのが一般的です。
「Aをモノイドとし、_x_ と _y_ をその担体集合の要素とする」の代わりに、「_A_ をモノイドとし、_x_ と _y_ を _A_ の要素とする」という言い方が一般的です。
この慣習をLeanでエンコードするには、モノイドからその担体集合への新しい種類の強制変換を定義します。

`CoeSort`クラスは`Coe`クラスと似ていますが、強制変換の対象が _sort_ （すなわち、`Type` または `Prop`）であるという例外があります。
Leanにおける用語_sort_は、これらデータを分類する型—`Type`はデータを分類する型を分類する型で、`Prop`はその真実の証拠を分類する命題を分類する型。
`Coe`が型の不一致が発生したときにチェックされるのと同様に、ソート以外の何かがソートを期待する文脈で提供されるときに`CoeSort`が使用されます。

モノイドからその担体集合への強制変換は担体を抽出します：
```lean
{{#example_decl Examples/Classes.lean CoeMonoid}}
```
この強制変換を使用すると、型シグネチャが負担の少ないものになります：
```lean
{{#example_decl Examples/Classes.lean foldMap}}
```

`CoeSort`のもう一つの有用な例は、`Bool`と`Prop`の間の隙間を埋めるために使用されます。
[等価性と順序についての章](standard-classes.md#equality-and-ordering)で議論したように、Leanの `if` 文は条件が `Bool` ではなく判定可能な命題であることを期待しています。
しかし、プログラムは通常、ブール値に基づいて分岐する必要があります。
それに対し、Lean標準ライブラリは `Bool` から問題の `Bool` が `true` に等しいという命題への強制変換を定義しています：
```lean
{{#example_decl Examples/Classes.lean CoeBoolProp}}
```
この場合、疑問に思われるソートは `Prop` ではなく `Type` です。

## 関数への強制変換 (Coercing to Functions)

プログラミングで定期的に出現する多くのデータ型は、その関数についていくつかの追加情報とともに構成されます。
例えば、名前はログに表示するためのものやいくつかの設定データが関数に伴って提供されることがあります。
さらに、`Monoid` の例のように、構造のフィールドに型を置くことは、操作の実装方法が複数あり、型クラスを使用するよりも手動での制御が必要な文脈で意味を成す場合があります。
例えば、JSONシリアライザによって発行される値の特定の詳細が重要であるかもしれません。これは、別のアプリケーションが特定の形式を期待しているためです。
場合によっては、構成データ自体から関数を導出できることもあります。

非関数型の値を関数型へと変換する型クラスとして `CoeFun` があります。
`CoeFun` には二つのパラメータがあります：一つ目は関数に変換されるべき型で、二つ目はどの関数型をターゲットにするかを決定する出力パラメータです。
```lean
{{#example_decl Examples/Classes.lean CoeFun}}
```
二つ目のパラメータ自体は型を計算する関数です。
Leanでは、型は第一級の市民であり、他のどんなものと同じように関数に渡されたり、関数から返されたりします。

例えば、その引数に定数を加える関数は、加える量を含むラッパーとして表現できます。
そして、実際の関数を定義するのではなく、どれだけ加えるかに関するラッパーとして以下のようになります：
```lean
{{#example_decl Examples/Classes.lean Adder}}
```
その引数に5を加える関数は、`howMuch` フィールドに `5` を持っています：
```lean
{{#example_decl Examples/Classes.lean add5}}
```
この `Adder` 型は、関数ではなく、その引数に適用するとエラーが発生します。
```lean
{{#example_in Examples/Classes.lean add5notfun}}
```
```output error
{{#example_out Examples/Classes.lean add5notfun}}
```
`CoeFun` インスタンスを定義することで、Leanはadderを `Nat → Nat` 型の関数へと変換します：
```lean
{{#example_decl Examples/Classes.lean CoeFunAdder}}

{{#example_in Examples/Classes.lean add53}}
```
```output info
{{#example_out Examples/Classes.lean add53}}
```
すべての `Adder` が `Nat → Nat` 関数に変換されるべきであるため、「CoeFun」の二つ目のパラメータへの引数は無視されました。

値自体が正しい関数型を特定するために必要な場合、`CoeFun` の二つ目のパラメータはもはや無視されません。
例えば、以下のJSON値の表現を考えます：
```lean
{{#example_decl Examples/Classes.lean JSON}}
```
JSONシリアライザは、シリアライズ可能な型を追跡し、そのシリアライズ自体のコードを含む構造体です：
```lean
{{#example_decl Examples/Classes.lean Serializer}}
```
文字列のシリアライザは、提供された文字列を `JSON.string` コンストラクタに包む必要しかないです：
```lean
{{#example_decl Examples/Classes.lean StrSer}}
```
JSONシリアライザを、引数をシリアライズする関数として見るためには、シリアライズ可能なデータの内部型を抽出する必要があります：
```lean
{{#example_decl Examples/Classes.lean CoeFunSer}}
```
このインスタンスがあれば、シリアライザを直接引数に適用することができます：
```lean
{{#example_decl Examples/Classes.lean buildResponse}}
```