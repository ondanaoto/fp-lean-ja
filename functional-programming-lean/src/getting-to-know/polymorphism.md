# 多相性

Leanにおいてもほとんどの言語と同様に、型は引数を取ることができます。
例えば、型 `List Nat` は自然数のリストを、`List String` は文字列のリストを、`List (List Point)` は点のリストのリストを表しています。
これはC#やJavaのような言語での `List<Nat>`、`List<String>`、あるいは `List<List<Point>>` によく似ています。
Leanでは関数に引数を渡すのに空白を使うのと同様に、型に引数を渡すのにも空白を使います。

関数型プログラミングにおいて、「_多相性（polymorphism）_」という用語は通常、型を引数として取るデータ型や定義を指します。
これはオブジェクト指向プログラミングコミュニティにおける用語の使い方とは異なります。オブジェクト指向プログラミングでは、スーパークラスの一部の振る舞いをオーバーライドするサブクラスを指しています。
この書籍では、「多相性」という言葉は常に最初の意味で使います。
これらの型引数は、データ型または定義内で使用され、引数の名前を他の型に置換することで、同じデータ型や定義を任意の型で使用することを可能にします。

`Point` 構造体は `x` および `y` のフィールドが `Float` であることを要求します。
しかし、各座標に特定の表現を要求するものは何もありません。
型を引数として取る多相的な `Point` バージョン、それを `PPoint` と呼びますが、これはその両方のフィールドの型として引数の型を使用することができます。
```lean
{{#example_decl Examples/Intro.lean PPoint}}
```
関数定義の引数が定義された直後に書かれるのと同様に、構造体の引数は構造体の名前の直後に書かれます。
型引数の名前としてギリシャ文字を使用するのがLeanでは慣例ですが、それ以上に具体的な名前が思いつかない場合に限ります。
`Type` は他の型を記述する型ですので、`Nat`、`List String`、および `PPoint Int` はすべて `Type` 型を持っています。

`List` と同様に、`PPoint` は特定の型を引数として提供することで使用できます。
```lean
{{#example_decl Examples/Intro.lean natPoint}}
```
この例では、両方のフィールドが `Nat` であることが期待されます。
関数がその引数変数を引数値に置き換えて呼び出されるように、`PPoint` に型 `Nat` を引数として提供すると、そのフィールド `x` と `y` の型が `Nat` になる構造体が得られます。これは引数名 `α` が引数型 `Nat` に置き換えられるからです。
型はLeanにおける普通の式なので、多相的型（`PPoint` のような）に引数を渡す際に特殊な構文は必要ありません。

定義もまた、型を引数として取ることができ、多相的になります。
`replaceX` 関数は `PPoint` の `x` フィールドを新しい値に置き換えます。
`replaceX` が _任意_ の多相的な点で機能するためには、それ自体が多相的でなければなりません。
これは最初の引数が点のフィールドの型であり、後続の引数が最初の引数の名前を参照することで達成されます。
```lean
{{#example_decl Examples/Intro.lean replaceX}}
```
言い換えれば、`point` と `newX` の型の引数が `α` を述べるとき、それらは _最初に提供された型であるどの型も指している_ のです。
これは関数引数の名前がその関数の本体中に提供されたとき値を参照する方法と似ています。

これは `replaceX` の型をLeanにチェックさせた後、`replaceX Nat` の型をチェックさせることで確認できます。
```lean
{{#example_in Examples/Intro.lean replaceXT}}
```
```output info
{{#example_out Examples/Intro.lean replaceXT}}
```
この関数型は最初の引数の _名前_ を含み、型内の後続の引数はこの名前を参照します。
関数適用の値は、関数の本体内で提供される引数値に引数名を置き換えて見つかるように、関数適用の型は、関数の戻り型で提供される引数値に引数名を置き換えて見つかります。
最初の引数 `Nat` を提供することで、型の残りの部分にあるすべての `α` の出現が `Nat` に置き換えられます：
```lean
{{#example_in Examples/Intro.lean replaceXNatT}}
```
```output info
{{#example_out Examples/Intro.lean replaceXNatT}}
```
残りの引数が明示的に名付けられていないため、追加の引数が提供されてもさらなる置換は発生しません。
```lean
{{#example_in Examples/Intro.lean replaceXNatOriginT}}
```
```output info
{{#example_out Examples/Intro.lean replaceXNatOriginT}}
```
```lean
{{#example_in Examples/Intro.lean replaceXNatOriginFiveT}}
```
```output info
{{#example_out Examples/Intro.lean replaceXNatOriginFiveT}}
```
型を引数として渡したという事実は、それを評価する能力には何の影響も与えません。
```lean
{{#example_in Examples/Intro.lean replaceXNatOriginFiveV}}
```
```output info
{{#example_out Examples/Intro.lean replaceXNatOriginFiveV}}
```

多相的な関数は、名前付きの型引数を取り、後続の型が引数の名前を参照することによって機能します。
しかし、名前を付けることができるという特別なことは型引数にはありません。
前向きおよび後ろ向きのサインを示すデータ型が与えられた場合：
```lean
{{#example_decl Examples/Intro.lean Sign}}
```
サインが引数である関数を書くことができます。
引数が前向きの場合、関数は `Nat` を返し、後ろ向きの場合は `Int` を返します：
```lean
{{#example_decl Examples/Intro.lean posOrNegThree}}
```
型は第一級であり、Lean言語の通常のルールを使用して計算できるため、データ型に対するパターンマッチングによって計算することができます。
Leanがこの関数をチェックするとき、関数の本体での `match` 式を型での `match` 式に対応させることによって、`pos` の場合には `Nat` を予想される型とし、`neg` の場合には `Int` を予想される型とすることを使用します。

`posOrNegThree` を `Sign.pos` に適用すると、関数の本体と戻り型にある引数名 `s` が `Sign.pos` に置き換えられます。
式とその型の両方で評価が行われることができます：
```lean
{{#example_eval Examples/Intro.lean posOrNegThreePos}}
```

## 連結リスト

Leanの標準ライブラリには、`List` と呼ばれる正統的な連結リストのデータ型が含まれており、それを使用するのに便利な特殊な構文があります。
リストは角括弧で書かれます。
例えば、10未満の素数を含むリストは次のように書けます:
```lean
{{#example_decl Examples/Intro.lean primesUnder10}}
```

背後では、`List` は次のように定義された帰納的なデータ型です:
```lean
{{#example_decl Examples/Intro.lean List}}
```
標準ライブラリにある実際の定義は少し異なりますが、まだ紹介されていない機能を使用しているためです。しかし、本質的には非常に似ています。
この定義によれば、`List` は`PPoint` と同様に単一の型を引数として取ります。
この型はリストに格納されるエントリの型です。
コンストラクタに従って、`List α` は `nil` または `cons` で作成できます。
コンストラクタ `nil` は空のリストを表し、`cons` コンストラクタは空でないリストに使用されます。
`cons` の最初の引数はリストの頭部であり、二番目の引数はその尾部です。
\\( n \\) 個のエントリを含むリストには \\( n \\) 個の `cons` コンストラクタが含まれ、最後のものには `nil` がその尾部になります。

`primesUnder10` の例は、`List` のコンストラクタを直接使用してより明確に書くことができます:
```lean
{{#example_decl Examples/Intro.lean explicitPrimesUnder10}}
```
これら二つの定義は完全に同等ですが、`primesUnder10` の方が `explicitPrimesUnder10` より読みやすいです。

`List` を消費する関数は、`Nat` を消費する関数とほとんど同じように定義できます。
実際には、連結リストを考えるときには、それぞれの `succ` コンストラクタに余分なデータフィールドがぶら下がっている `Nat` としてそれを考えることができます。
この観点から、リストの長さを計算することは、それぞれの `cons` を `succ` に置き換え、最後の `nil` を `zero` に置き換えるプロセスです。
`replaceX` がポイントのフィールドの型を引数として取ったように、`length` はリストのエントリの型を引数として取ります。
たとえば、リストが文字列を含む場合、最初の引数は `String` です: `{{#example_eval Examples/Intro.lean length1EvalSummary 0}}`.
次のように計算すべきです:
```
{{#example_eval Examples/Intro.lean length1EvalSummary}}
```

`length` の定義は、リストエントリ型を引数として取るため多相的であり（多相性）、それ自体を参照するため再帰的です（再帰性）。
一般的に、関数はデータの形に従います：再帰的なデータ型は再帰的な関数を導き、多相的なデータ型は多相的な関数を導きます。
```lean
{{#example_decl Examples/Intro.lean length1}}
```

変数の名前は `xs` や `ys` のように慣例的に使用され、不明な値のリストを意味します。
名前に `s` が付いていることで、それらが複数形であることを示し、それゆえ "exes" や "whys" と発音されるので、"x s" や "y s" ではなく発音されます。

リストの関数を読みやすくするために、`[]` のブラケット表記を `nil` に一致するパターンマッチに使用し、`cons` の代わりに接頭辞 `::` を使うことができます：
```lean
{{#example_decl Examples/Intro.lean length2}}
```

## 暗黙の引数

`replaceX` や `length` は使用するのにやや官僚的です。なぜなら、型の引数は通常、後続の値によって一意に決定されるからです。
実際、ほとんどの言語ではコンパイラが自動的に型の引数を決定することができ、時々ユーザからの助けを必要とします。
Leanでも同様です。
関数定義時に、丸括弧の代わりに中括弧で引数を宣言することで引数を _暗黙的な_ ものにすることができます。
たとえば、暗黙的な型引数を持つ `replaceX` のバージョンは以下のようになります：
```lean
{{#example_decl Examples/Intro.lean replaceXImp}}
```
それは `natOrigin` を明示的に `Nat` を提供せず使用できます。なぜならLeanは後続の引数から `α` の値を _推論_ できるからです：
```lean
{{#example_in Examples/Intro.lean replaceXImpNat}}
```
```output info
{{#example_out Examples/Intro.lean replaceXImpNat}}
```

同様に、`length` を再定義してエントリ型を暗黙的に取ることができます：
```lean
{{#example_decl Examples/Intro.lean lengthImp}}
```
この `length` 関数は `primesUnder10` に直接適用できます：
```lean
{{#example_in Examples/Intro.lean lengthImpPrimes}}
```
```output info
{{#example_out Examples/Intro.lean lengthImpPrimes}}
```
~~ ここから ~~
標準ライブラリでは、Leanはこの関数を `List.length` と呼びます。これは、構造体のフィールドアクセスに使用されるドット構文がリストの長さを見つけるためにも使用できることを意味します：
```lean
{{#example_in Examples/Intro.lean lengthDotPrimes}}
```
```output info
{{#example_out Examples/Intro.lean lengthDotPrimes}}
```

Leanが常に暗黙の引数を見つけることができるわけではないため、C#やJavaと同様に、時には型引数を明示的に指定する必要があります。これらの場合、名前を使って提供することができます。
例えば、整数リスト専用の `List.length` を使用する場合、`α` を `Int` に設定して指定することができます：
```lean
{{#example_in Examples/Intro.lean lengthExpNat}}
```
```output info
{{#example_out Examples/Intro.lean lengthExpNat}}
```

## 組み込みデータ型のさらなる活用

リストに加えて、Leanの標準ライブラリには様々なコンテキストで使用できる他の構造体や帰納的データ型が含まれています。

### `Option`
全てのリストに最初のエントリが存在するわけではありません。一部のリストは空です。
コレクション上の操作は、探しているものを見つけられないことがあります。
例えば、リスト内の最初のエントリを探す関数が、そのようなエントリを見つけられない場合があるのです。
そのため、最初のエントリがない場合の信号を送る方法が必要です。

多くの言語には `null` 値があり、これは値が存在しないことを表します。
既存の型に特別な `null` 値を装備する代わりに、Leanは `Option` というデータ型を提供しており、他の型に欠落値の指標を備え付けます。
例えば、`Option Int` は `Int` のnull許容型を表し、`Option (List String)` は文字列のリストのnull許容型を表します。
新しい型を導入してnull可能性を表すことで、型システムは `null` のチェックを忘れることがないように保証します。なぜなら、`Option Int` は `Int` が期待されるコンテキストで使用することはできないからです。

`Option` には、それぞれ非nullバージョンとnullバージョンを表す `some` および `none` という2つのコンストラクタがあります。
非nullコンストラクタである `some` には下層の値が含まれますが、`none` には引数がありません：
```lean
{{#example_decl Examples/Intro.lean Option}}
```

`Option` 型は、C# や Kotlin などの言語における nullable 型ととても似ていますが、同一ではありません。
これらの言語では、ある型（例えば`Boolean`）が常にその型の実際の値（`true` および `false`）を指す場合、`Boolean?` や `Nullable<Boolean>` の型は追加で `null` 値を許容します。
型チェッカーと他のツールがnullのチェックを覚えているのを助けるこのシステムの追跡は非常に有用です。また、型署名を介してnull可能性を明示的に記述するAPIは、そうでないものよりも情報が豊富です。
しかし、これらのnullableな型は、Leanの `Option` と一つ非常に重要な方法で異なります。それは、複数のオプションの層を許さないということです。
`{{#example_out Examples/Intro.lean nullThree}}` を `{{#example_in Examples/Intro.lean nullOne}}`、`{{#example_in Examples/Intro.lean nullTwo}}`、または `{{#example_in Examples/Intro.lean nullThree}}` を使って構築することができます。
一方、C#は `?` を非null許容型にのみ加えることができるために、複数のnull許容層を禁じていますが、Kotlinは `T??` を `T?` と同等とみなしています。
この微妙な違いは実際にはほとんど関連性がありませんが、時々重要になることがあります。

リストの最初のエントリを、存在する場合に限って探すには `List.head?` を使用します。
クエスチョンマークは名称の一部であり、C# や Kotlin での nullable 型を示すクエスチョンマークの使用とは関係ありません。
`List.head?` の定義において、リストの尾部を表すためにアンダースコアが使用されています。
パターンでは、アンダースコアは一切のものにマッチしますが、マッチしたデータを参照するための変数を導入することはありません。
アンダースコアを代わりに名前として使用することで、入力の一部が無視されることを明確に読者に伝える方法です。
```lean
{{#example_decl Examples/Intro.lean headHuh}}
```
Leanの命名規則では、失敗する可能性のある操作をグループ化して定義し、それぞれの操作に適したサフィックスを使用します。具体的には、`Option` を返すバージョンには `?` を、無効な入力が提供された場合にクラッシュするバージョンには `!` を、操作が失敗する可能性がある場合にデフォルト値を返すバージョンには `D` を使用します。例えば、`head` は呼び出し側がリストが空でないことを数学的に証明する必要があり、`head?` は `Option` を返し、`head!` は空のリストが渡されるとプログラムをクラッシュさせ、`headD` はリストが空の場合に返すデフォルト値を取ります。疑問符と感嘆符は特別な構文ではなく、名前の一部です。これは、Leanの命名規則が多くの言語よりも柔軟であることを示しています。

`head?` は `List` 名前空間で定義されているため、アクセス表記法で使用することができます。これは、`List` 型のオブジェクトに直接 `head?` 操作を適用することを意味します。例えば、`myList.head?` のように使用して、`myList` というリストの最初の要素を取得し、その要素が存在しない場合は `none` を返すことができます。これにより、コードがより読みやすく、効率的になります。
```lean
{{#example_in Examples/Intro.lean headSome}}
```
```output info
{{#example_out Examples/Intro.lean headSome}}
```
しかし、空のリストでテストを試みると、次のような二つのエラーが発生します：
```lean
{{#example_in Examples/Intro.lean headNoneBad}}
```
```output error
{{#example_out Examples/Intro.lean headNoneBad}}

{{#example_out Examples/Intro.lean headNoneBad2}}
```
これは、Leanが式の型を完全に決定することができなかったためです。特に、`List.head?` への暗黙の型引数も、`List.nil` への暗黙の型引数も見つけることができませんでした。Leanの出力において、`?m.XYZ` は推論できなかったプログラムの一部を表しています。これら未知の部分は _メタ変数_ と呼ばれ、一部のエラーメッセージに現れます。式を評価するためには、Leanはその型を見つけることができる必要がありますが、空のリストにはその型を見つけるためのエントリが存在しないため、型が利用できませんでした。型を明示的に提供することで、Leanは処理を進めることができます。
```lean
{{#example_in Examples/Intro.lean headNone}}
```
```output info
{{#example_out Examples/Intro.lean headNone}}
```
型は型アノテーションを用いて提供することもできます：
```lean
{{#example_in Examples/Intro.lean headNoneTwo}}
```
```output info
{{#example_out Examples/Intro.lean headNoneTwo}}
```
エラーメッセージは有用な手がかりを提供します。両方のメッセージで _同じ_ メタ変数が欠けている暗黙の引数を記述するために使用されており、これはLeanが二つの欠けている部分が解決策を共有することを決定していることを意味します。ただし、解決策の実際の値を決定することはできませんでした。

### `Prod`

`Prod` 構造体は、「積」を意味するもので、二つの値を一緒に結合するための一般的な方法です。例えば、`Prod Nat String` は `Nat` と `String` を含んでいます。言い換えれば、`PPoint Nat` は `Prod Nat Nat` に置き換えることができます。`Prod` は、C#のタプルやKotlinの `Pair` および `Triple` 型、C++の `tuple` に非常に似ています。`Point` のような単純なケースでさえ、独自の構造を定義することで、ドメイン用語を使用するとコードの読みやすさが向上します。さらに、構造型を定義することで、異なるドメインの概念に異なる型を割り当てることにより、より多くのエラーを捕捉し、それらが混同されるのを防ぐことができます。

一方で、新しい型を定義するためのオーバーヘッドが価値のないケースもあります。さらに、いくつかのライブラリは十分に一般的で、「ペア」以上の具体的な概念がない場合もあります。最後に、標準ライブラリには、組み込みのペア型で作業するのを容易にするためのさまざまな便利な機能が含まれています。

標準のペア構造は `Prod` と呼ばれます。
```lean
{{#example_decl Examples/Intro.lean Prod}}
```
リストは頻繁に使用されるため、より読みやすくするための特別な構文があります。同様の理由で、製品型とそのコンストラクタにも特別な構文があります。型 `Prod α β` は通常 `α × β` と記述され、集合のデカルト積の通常の表記法を反映しています。同様に、`Prod` にはペアのための通常の数学的表記法が利用可能です。つまり、次のように書く代わりに：
```lean
{{#example_decl Examples/Intro.lean fivesStruct}}
```
このように書くだけで十分です：
```lean
{{#example_decl Examples/Intro.lean fives}}
```

両方の表記は右結合です。これは、以下の定義が同等であることを意味します：
```lean
{{#example_decl Examples/Intro.lean sevens}}

{{#example_decl Examples/Intro.lean sevensNested}}
```
つまり、二つ以上の型のすべての積とそれに対応するコンストラクタは、実際には裏で入れ子になった積と入れ子になったペアです。

### `Sum`

`Sum` データ型は、二つの異なる型の値の間で選択を可能にする一般的な方法です。例えば、`Sum String Int` は `String` または `Int` のどちらかです。`Prod` のように、`Sum` は非常に一般的なコードを書く場合や、特定のドメインに特化した型がない非常に小さなコードのセクション、または標準ライブラリに有用な機能が含まれている場合に使用するべきです。ほとんどの状況では、カスタム帰納型を使用する方が読みやすく、保守しやすいです。

型 `Sum α β` の値は、型 `α` の値に適用されるコンストラクタ `inl` または型 `β` の値に適用されるコンストラクタ `inr` のいずれかです：
```lean
{{#example_decl Examples/Intro.lean Sum}}
```
これらの名前はそれぞれ「左注入」と「右注入」の略称です。`Prod` に対するデカルト積の表記法と同様に、`Sum` には「円にプラス」の表記法が使用され、`α ⊕ β` は `Sum α β` を書く別の方法です。`Sum.inl` と `Sum.inr` に特別な構文はありません。

例えば、ペットの名前が犬の名前または猫の名前のいずれかである場合、それらに対する型を文字列の合計として導入することができます：
```lean
{{#example_decl Examples/Intro.lean PetName}}
```
実際のプログラムでは、この目的のために情報豊かなコンストラクタ名を持つカスタム帰納型を定義する方が通常は良いでしょう。ここでは、犬の名前には `Sum.inl` を、猫の名前には `Sum.inr` を使用します。これらのコンストラクタは動物の名前のリストを作成するために使用することができます：
```lean
{{#example_decl Examples/Intro.lean animals}}
```
パターンマッチングは、二つのコンストラクタの間を区別するために使用できます。例えば、動物の名前のリストの中で犬の数（つまり `Sum.inl` コンストラクタの数）を数える関数は、以下のようになります：
```lean
{{#example_decl Examples/Intro.lean howManyDogs}}
```
関数呼び出しは中置演算子よりも先に評価されるため、`howManyDogs morePets + 1` は `(howManyDogs morePets) + 1` と同じです。予想通り、`{{#example_in Examples/Intro.lean dogCount}}` は `{{#example_out Examples/Intro.lean dogCount}}` を生成します。

### `Unit`

`Unit` は引数のないコンストラクタを一つだけ持つ型で、そのコンストラクタは `unit` と呼ばれます。言い換えると、`Unit` はこのコンストラクタが何の引数も取らずに適用される単一の値のみを記述します。`Unit` は以下のように定義されます：
```lean
{{#example_decl Examples/Intro.lean Unit}}
```

単独で、`Unit` は特に有用ではありません。しかし、多態性のあるコードでは、欠けているデータのプレースホルダーとして使用できます。例えば、以下の帰納的データ型は算術式を表します：
```lean
{{#example_decl Examples/Intro.lean ArithExpr}}
```
型引数 `ann` はアノテーションを意味し、各コンストラクタは注釈されます。パーサーから来る式にはソースの位置が注釈されることがあるため、`ArithExpr SourcePos` の返り値は、パーサーが各サブ式に `SourcePos` を配置したことを保証します。しかし、パーサーから来ない式にはソースの位置がないため、その型は `ArithExpr Unit` になります。

さらに、Leanのすべての関数には引数があるため、他の言語の引数がない関数は `Unit` 引数を取る関数として表現できます。返り値の位置では、`Unit` 型はCから派生した言語の `void` に似ています。C系の言語では、`void` を返す関数は呼び出し元に制御を戻しますが、興味深い値は返しません。意図的に興味のない値として、`Unit` は型システムに特別な `void` 機能を必要とせずにこれを表現することができます。Unitのコンストラクタは空の括弧として書くことができます：`{{#example_in Examples/Intro.lean unitParens}} : {{#example_out Examples/Intro.lean unitParens}}`。

### `Empty`

`Empty` データ型は、いかなるコンストラクタも持ちません。そのため、到達不可能なコードを示します。なぜなら、`Empty` 型の値で終了するような一連の呼び出しは決して発生しないからです。

`Empty` は `Unit` ほど頻繁には使用されません。しかし、特定の専門的なコンテキストで有用です。多くの多態性データ型は、そのコンストラクタのすべてで型引数を使用しません。例えば、`Sum.inl` と `Sum.inr` はそれぞれ `Sum` の型引数のうちの1つだけを使用します。`Sum` への型引数として `Empty` を使用することで、プログラムの特定のポイントでの1つのコンストラクタを除外することができます。これにより、追加の制約があるコンテキストで汎用コードを使用することが可能になります。

### 命名：合計、積、および単位

一般的に、複数のコンストラクタを提供する型は _合計型（sum types）_ と呼ばれ、単一のコンストラクタが複数の引数を取る型は _積型（product types）_ と呼ばれます。これらの用語は、通常の算術で使用される合計と積に関連しています。関与する型が有限数の値を含む場合に、この関係は最も理解しやすいです。例えば `α` と `β` がそれぞれ \\( n \\) 個と \\( k \\) 個の異なる値を含む型であれば、`α ⊕ β` は \\( n + k \\) 個の異なる値を含み、`α × β` は \\( n \times k \\) 個の異なる値を含みます。例えば、`Bool` は2つの値 `true` と `false` を持ち、`Unit` は1つの値 `Unit.unit` を持ちます。積 `Bool × Unit` は2つの値 `(true, Unit.unit)` と `(false, Unit.unit)` を持ち、合計 `Bool ⊕ Unit` は3つの値 `Sum.inl true`、`Sum.inl false`、`Sum.inr unit` を持ちます。同様に、\\( 2 \times 1 = 2 \\)、\\( 2 + 1 = 3 \\) です。

## 遭遇する可能性のあるメッセージ

すべての定義可能な構造や帰納型が `Type` 型を持つわけではありません。特に、コンストラクタが任意の型を引数として取る場合、その帰納型は異なる型を持つ必要があります。これらのエラーは通常、"宇宙レベル"について何かを述べています。例えば、この帰納型の場合：
```lean
{{#example_in Examples/Intro.lean TypeInType}}
```
Leanは以下のエラーを出力します：
```output error
{{#example_out Examples/Intro.lean TypeInType}}
```
後の章で、なぜこのようなことが起こるのか、そして定義をどのように修正して機能させるかが説明されます。現在は、コンストラクタではなく帰納型全体に型を引数として渡すように試してみてください。

同様に、コンストラクタの引数が定義されているデータ型を引数として取る関数である場合、その定義は拒否されます。例えば：
```lean
{{#example_in Examples/Intro.lean Positivity}}
```
は以下のメッセージを引き起こします：
```output error
{{#example_out Examples/Intro.lean Positivity}}
```
技術的な理由から、これらのデータ型を許可することは、Leanの内部論理を損なう可能性があり、定理証明者としての使用に適さなくなる可能性があります。

帰納型への引数を忘れると、混乱するメッセージが表示されることもあります。例えば、`ctor` の型で `MyType` に引数 `α` が渡されていない場合：
```lean
{{#example_in Examples/Intro.lean MissingTypeArg}}
```
Leanは以下のエラーを返します：
```output error
{{#example_out Examples/Intro.lean MissingTypeArg}}
```
このエラーメッセージは、`MyType` の型は `Type → Type` であり、それ自体が型を記述していないことを示しています。`MyType` は実際の型になるためには引数が必要です。

同じメッセージは、他のコンテキストで型引数が省略された場合にも表示されることがあります。例えば、定義のための型シグネチャで：
```lean
{{#example_decl Examples/Intro.lean MyTypeDef}}

{{#example_in Examples/Intro.lean MissingTypeArg2}}
```

## 練習問題

* リストの最後のエントリを見つける関数を書きます。これは `Option` を返すべきです。
* 与えられた述語を満たすリスト内の最初のエントリを見つける関数を書きます。定義を `def List.findFirst? {α : Type} (xs : List α) (predicate : α → Bool) : Option α :=` で始めます。
* ペア内の二つのフィールドを交換する `Prod.swap` 関数を書きます。定義を `def Prod.swap {α β : Type} (pair : α × β) : β × α :=` で始めます。
* `PetName` の例をカスタムデータ型を使用するように書き換え、`Sum` を使用するバージョンと比較します。
* 二つのリストをペアのリストに結合する `zip` 関数を書きます。結果のリストは入力リストの最短のものと同じ長さであるべきです。定義を `def zip {α β : Type} (xs : List α) (ys : List β) : List (α × β) :=` で始めます。
* リストの最初の \\( n \\) エントリを返す多態性のある `take` 関数を書きます。ここで \\( n \\) は `Nat` です。リストに `n` より少ないエントリが含まれている場合、結果のリストは入力リストであるべきです。`{{#example_in Examples/Intro.lean takeThree}}` は `{{#example_out Examples/Intro.lean takeThree}}` を生成し、`{{#example_in Examples/Intro.lean takeOne}}` は `{{#example_out Examples/Intro.lean takeOne}}` を生成するはずです。
* 型と算術の類推を使用して、積を合計に分配する関数を書きます。言い換えると、それは `α × (β ⊕ γ) → (α × β) ⊕ (α × γ)` の型を持つべきです。
* 型と算術の類推を使用して、二倍の乗算を合計に変換する関数を書きます。言い換えると、それは `Bool × α → α ⊕ α` の型を持つべきです。