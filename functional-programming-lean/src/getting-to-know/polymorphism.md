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
言い換えれば、`point` と `newX` の型の引数が `α` を述べるとき、それらは _最初に提供された型であるどの型を指している_ のです。
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
ヒント: コンテンツは多すぎるため、翻訳プロセスはここで完了していません。次に進むには、追加の指示をお願いします。