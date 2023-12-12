# ポリモーフィズム

ほとんどの言語と同様に、Leanの型は引数を取ることができます。
例えば、型 `List Nat` は自然数のリストを、`List String` は文字列のリストを、`List (List Point)` は点のリストのリストを表します。
これは、C#やJavaのような言語での `List<Nat>`、`List<String>`、`List<List<Point>>` と非常に似ています。
Leanは関数に引数を渡すときにスペースを使用するのと同様に、型に引数を渡すときにもスペースを使用します。

関数型プログラミングでは、_ポリモーフィズム_ という用語は通常、型を引数として取るデータ型や定義を指します。
これは、オブジェクト指向プログラミングコミュニティでの用語の使い方とは異なり、そこでは通常、サブクラスがスーパークラスのいくつかの振る舞いをオーバーライドすることを指します。
この本では、「ポリモーフィズム」という言葉は常に最初の意味で使用されます。
これらの型引数はデータ型や定義で使用され、引数の名前を他の型に置き換えることで、任意の型で同じデータ型や定義を使用することができます。

`Point` 構造体は、`x` と `y` の両方のフィールドが `Float` であることを要求します。
しかし、各座標の具体的な表現が点に必要なわけではありません。
`Point` のポリモーフィックバージョンである `PPoint` は、型を引数として取り、その型を両方のフィールドに使用することができます：
```lean
{{#example_decl Examples/Intro.lean PPoint}}
```
関数定義の引数が定義される名前の直後に書かれるのと同様に、構造体の引数は構造体の名前の直後に書かれます。
Leanでは、特に名前が思い浮かばない場合に型引数にギリシャ文字を使用するのが慣例です。
`Type` は他の型を記述する型なので、`Nat`、`List String`、`PPoint Int` はすべて `Type` 型を持ちます。

`List` のように、`PPoint` も特定の型を引数として提供することで使用できます：
```lean
{{#example_decl Examples/Intro.lean natPoint}}
```
この例では、両方のフィールドが `Nat` であることが期待されます。
関数がその引数変数を引数値で置き換えることで呼び出されるのと同様に、`PPoint` に型 `Nat` を引数として提供することで、引数名 `α` が引数型 `Nat` に置き換えられた結果、フィールド `x` と `y` が型 `Nat` を持つ構造体が得られます。
Leanでは型は通常の式なので、ポリモーフィック型（`PPoint` のような）に引数を渡す際に特別な構文は必要ありません。

定義もまた型を引数として取ることができ、これによりポリモーフィックになります。
関数 `replaceX` は `PPoint` の `x` フィールドを新しい値で置き換えます。
`replaceX` が _任意の_ ポリモーフィックな点で動作するためには、それ自体がポリモーフィックでなければなりません。
これは、最初の引数が点のフィールドの型であり、後続の引数が最初の引数の名前を参照することで実現されます。
```lean
{{#example_decl Examples/Intro.lean replaceX}}
```
言い換えれば、引数 `point` と `newX` の型が `α` を言及しているとき、それらは _最初の引数として提供された型を指しています_ 。
これは、関数の引数名が関数の本体で提供された値を参照する方法と似ています。

Leanに `replaceX` の型をチェックさせ、その後 `replaceX Nat` の型をチェックさせることでこれを確認できます。
```lean
{{#example_in Examples/Intro.lean replaceXT}}
```
```output info
{{#example_out Examples/Intro.lean replaceXT}}
```
この関数型には最初の引数の _名前_ が含まれ、型の後続の引数はこの名前を参照しています。
関数適用の値が関数の本体で提供された引数値で引数名を置き換えることによって見つかるのと同様に、関数適用の型は関数の戻り値の型で提供された値で引数の名前を置き換えることによって見つかります。
最初の引数 `Nat` を提供することで、型の残りの部分にある `α` のすべての出現が `Nat` に置き換えられます：
```lean
{{#example_in Examples/Intro.lean replaceXNatT}}
```
```output info
{{#example_out Examples/Intro.lean replaceXNatT}}
```
残りの引数が明示的に名前付けされていないため、さらに引数が提供されてもこれ以上の置換は発生しません：
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
型に引数として型を渡すことによって全体の関数適用式の型が決定されたとしても、それを評価する能力には影響しません。
```lean
{{#example_in Examples/Intro.lean replaceXNatOriginFiveV}}
```
```output info
{{#example_out Examples/Intro.lean replaceXNatOriginFiveV}}
```

ポリモーフィック関数は、名前付きの型引数を取り、後続の型が引数の名前を参照することで機能します。
しかし、型引数が名前付けされることを可能にする特別なことは何もありません。
正または負の符号を表すデータ型が与えられた場合：
```lean
{{#example_decl Examples/Intro.lean Sign}}
```
引数が符号である関数を書くことができます。
引数が正の場合、関数は `Nat` を返し、負の場合は `Int` を返します：
```lean
{{#example_decl Examples/Intro.lean posOrNegThree}}
```
型は第一級であり、Leanの通常のルールを使用して計算できるため、データ型に対するパターンマッチングを使用して計算することができます。
Leanがこの関数をチェックするとき、関数の本体の `match` 式が型の `match` 式に対応しているという事実を使用して、`pos` の場合には `Nat` を期待される型とし、`neg` の場合には `Int` を期待される型とします。

`posOrNegThree` を `Sign.pos` に適用すると、関数の本体とその戻り値の型の両方で引数名 `s` が `Sign.pos` に置き換えられます。
式とその型の両方で評価が行われることがあります：
```lean
{{#example_eval Examples/Intro.lean posOrNegThreePos}}
```

## リンクリスト

Leanの標準ライブラリには、`List` と呼ばれる標準的なリンクリストデータ型が含まれており、それをより便利に使用するための特別な構文があります。
リストは角括弧で書かれます。
例えば、10未満の素数を含むリストは次のように書かれます：
```lean
{{#example_decl Examples/Intro.lean primesUnder10}}
```
舞台裏では、`List`は帰納的データ型で、以下のように定義されています：

```lean
{{#example_decl Examples/Intro.lean List}}
```

標準ライブラリにおける実際の定義は、まだ紹介されていない機能を使用しているため、若干異なりますが、本質的には非常に似ています。
この定義によると、`List`は`PPoint`がそうであったように、単一の型を引数として取ります。
この型はリストに格納されるエントリの型です。
コンストラクタによると、`List α`は`nil`または`cons`を使って構築することができます。
コンストラクタ`nil`は空のリストを表し、`cons`コンストラクタは非空のリストに使用されます。
`cons`の最初の引数はリストの先頭であり、2番目の引数はその尾です。
\\( n \\)個のエントリを含むリストは、最後のものが尾に`nil`を持つ\\( n \\)個の`cons`コンストラクタを含んでいます。

`primesUnder10`の例は、`List`のコンストラクタを直接使用することで、より明示的に書くことができます：

```lean
{{#example_decl Examples/Intro.lean explicitPrimesUnder10}}
```

これら2つの定義は完全に同等ですが、`primesUnder10`は`explicitPrimesUnder10`よりも読みやすいです。

`List`を消費する関数は、`Nat`を消費する関数とほぼ同じ方法で定義することができます。
実際、リンクリストを考える一つの方法は、各`succ`コンストラクタに余分なデータフィールドが付いている`Nat`として考えることです。
この観点から、リストの長さを計算することは、各`cons`を`succ`に置き換え、最後の`nil`を`zero`に置き換えるプロセスです。
`replaceX`がポイントのフィールドの型を引数として取ったように、`length`はリストのエントリの型を引数として取ります。
例えば、リストが文字列を含む場合、最初の引数は`String`です：`{{#example_eval Examples/Intro.lean length1EvalSummary 0}}`。
計算は以下のようになるはずです：

```
{{#example_eval Examples/Intro.lean length1EvalSummary}}
```

`length`の定義は、リストエントリの型を引数として取るため多相的（ポリモーフィック）であり、自身を参照するため再帰的です。
一般的に、関数はデータの形状に従います：再帰的なデータ型は再帰的な関数に、多相的なデータ型は多相的な関数につながります。

```lean
{{#example_decl Examples/Intro.lean length1}}
```

`xs`や`ys`といった名前は、未知の値のリストを表すために慣習的に使用されます。
名前にある`s`は複数形を示しているため、「エックスエス」や「ワイエス」とは発音せず、「エックスズ」と「ワイズ」と発音します。

リスト上の関数を読みやすくするために、`nil`に対してパターンマッチするためのブラケット記法`[]`を使用したり、`cons`の代わりに中置の`::`を使用することができます：

```lean
{{#example_decl Examples/Intro.lean length2}}
```

## 暗黙の引数

`replaceX`と`length`は、型引数が後の値によって一意に決定されるため、使用するのがやや煩雑です。
実際、ほとんどの言語では、コンパイラは自身で型引数を決定することができ、ユーザーからの助けをたまに必要とします。
Leanでも同様です。
関数を定義する際に、括弧の代わりに中括弧で引数を宣言することにより、_暗黙的_にすることができます。
たとえば、暗黙の型引数を持つ`replaceX`のバージョンは次のようになります：

```lean
{{#example_decl Examples/Intro.lean replaceXImp}}
```

後の引数から`α`の値を_Leanが_推測できるため、`natOrigin`を使用する際に`Nat`を明示的に提供する必要はありません：

```lean
{{#example_in Examples/Intro.lean replaceXImpNat}}
```
```output info
{{#example_out Examples/Intro.lean replaceXImpNat}}
```

同様に、`length`はエントリタイプを暗黙的に取るように再定義することができます：

```lean
{{#example_decl Examples/Intro.lean lengthImp}}
```

この`length`関数は`primesUnder10`に直接適用することができます：

```lean
{{#example_in Examples/Intro.lean lengthImpPrimes}}
```
```output info
{{#example_out Examples/Intro.lean lengthImpPrimes}}
```

標準ライブラリでは、Leanはこの関数を`List.length`と呼び、構造体のフィールドアクセスに使用されるドット構文をリストの長さを見つけるためにも使用することができます：

```lean
{{#example_in Examples/Intro.lean lengthDotPrimes}}
```
```output info
{{#example_out Examples/Intro.lean lengthDotPrimes}}
```

C#やJavaが時々型引数を明示的に提供することを要求するように、Leanも常に暗黙の引数を見つけることができるわけではありません。
これらの場合、名前を使って提供することができます。
たとえば、整数のリストのみを対象とする`List.length`のバージョンは、`α`を`Int`に設定することで指定することができます：

```lean
{{#example_in Examples/Intro.lean lengthExpNat}}
```
```output info
{{#example_out Examples/Intro.lean lengthExpNat}}
```

## その他の組み込みデータ型

リストに加えて、Leanの標準ライブラリには、さまざまなコンテキストで使用できる他の構造や帰納的データ型が含まれています。

### `Option`
すべてのリストに最初のエントリがあるわけではありません。一部のリストは空です。
コレクションの操作は、探しているものを見つけられないことがあります。
たとえば、リストの最初のエントリを見つける関数は、そのようなエントリを見つけられないかもしれません。
したがって、最初のエントリがなかったことを示す方法が必要です。

多くの言語には、値がないことを表す`null`値があります。
既存の型に特別な`null`値を装備する代わりに、Leanは`Option`と呼ばれるデータ型を提供し、欠落している値のインジケータを他の型に装備します。
たとえば、null許容の`Int`は`Option Int`によって表され、null許容の文字列リストは`Option (List String)`の型で表されます。
null許容性を表すために新しい型を導入することで、`Option Int`は`Int`が期待されるコンテキストで使用できないため、`null`のチェックを忘れることがないように型システムが保証します。

`Option`には、それぞれ基礎となる型の非nullバージョンとnullバージョンを表す`some`と`none`という2つのコンストラクタがあります。
非nullコンストラクタである`some`は基礎となる値を含み、`none`は引数を取りません：

```lean
{{#example_decl Examples/Intro.lean Option}}
```

`Option`型はC#やKotlinのような言語のnull許容型に非常に似ていますが、同一ではありません。
これらの言語では、型（例えば`Boolean`）が常にその型の実際の値（`true`と`false`）を指す場合、型`Boolean?`や`Nullable<Boolean>`は追加で`null`値を許容します。
これを型システムで追跡することは非常に有用です：型チェッカーやその他のツールは、プログラマーがnullをチェックすることを思い出させるのに役立ち、null許容性を型シグネチャを通じて明示的に記述するAPIは、そうでないものよりも情報が豊富です。
しかし、これらのnull許容型はLeanの`Option`と1つ非常に重要な点で異なります。それは、複数のオプショナリティの層を許容しないということです。
`{{#example_out Examples/Intro.lean nullThree}}`は`{{#example_in Examples/Intro.lean nullOne}}`、`{{#example_in Examples/Intro.lean nullTwo}}`、または`{{#example_in Examples/Intro.lean nullThree}}`を使用して構築することができます。
一方、C#は`?`をnull非許容型にのみ追加することを許可し、Kotlinは`T??`を`T?`と同等とみなします。
この微妙な違いは実際にはあまり関係ありませんが、時々重要になることがあります。

リストの最初のエントリを、存在する場合に見つけるには`List.head?`を使用します。
疑問符は名前の一部であり、C#やKotlinでnull許容型を示すために使用される疑問符とは関係ありません。
`List.head?`の定義では、リストの残りの部分を表すためにアンダースコアが使用されます。
パターンでは、アンダースコアは何にでも一致しますが、一致したデータを参照する変数を導入しません。
名前の代わりにアンダースコアを使用することは、入力の一部が無視されることを読者に明確に伝える方法です。
```lean
{{#example_decl Examples/Intro.lean headHuh}}
```
Leanの命名規則では、失敗する可能性のある操作を、`Option`を返すバージョンには`?`、無効な入力が提供された場合にプログラムをクラッシュさせるバージョンには`!`、操作が失敗する場合にデフォルト値を返すバージョンには`D`という接尾辞を使用してグループで定義します。
例えば、`head`はリストが空でないことを証明する数学的証拠を呼び出し元に提供することを要求し、`head?`は`Option`を返し、`head!`は空のリストが渡された場合にプログラムをクラッシュさせ、`headD`はリストが空の場合に返すデフォルト値を取ります。
疑問符と感嘆符は名前の一部であり、特別な構文ではありません。Leanの命名規則は多くの言語よりも自由です。

`head?`は`List`名前空間で定義されているため、アクセサ記法で使用できます：
```lean
{{#example_in Examples/Intro.lean headSome}}
```
```output info
{{#example_out Examples/Intro.lean headSome}}
```
しかし、空のリストでテストしようとすると、2つのエラーが発生します：
```lean
{{#example_in Examples/Intro.lean headNoneBad}}
```
```output error
{{#example_out Examples/Intro.lean headNoneBad}}

 {{#example_out Examples/Intro.lean headNoneBad2}}
```
これは、Leanが式の型を完全に決定できなかったためです。
特に、`List.head?`への暗黙の型引数も、`List.nil`への暗黙の型引数も見つけることができませんでした。
Leanの出力で、`?m.XYZ`は推論できなかったプログラムの一部を表します。
これらの未知の部分は_メタ変数_と呼ばれ、一部のエラーメッセージで発生します。
式を評価するためには、Leanがその型を見つけることができる必要がありますが、空のリストには型を見つけるためのエントリがないため、型が利用できませんでした。
明示的に型を提供することで、Leanは処理を進めることができます：
```lean
{{#example_in Examples/Intro.lean headNone}}
```
```output info
{{#example_out Examples/Intro.lean headNone}}
```
型は型注釈を使用しても提供できます：
```lean
{{#example_in Examples/Intro.lean headNoneTwo}}
```
```output info
{{#example_out Examples/Intro.lean headNoneTwo}}
```
エラーメッセージは有用な手がかりを提供します。
両方のメッセージは_同じ_メタ変数を使用して不足している暗黙の引数を説明しており、Leanは2つの欠けている部分が解決策を共有することを決定しましたが、実際の解決策の値を決定することはできませんでした。

### `Prod`

`Prod`構造体は、"Product"の略で、2つの値を一緒に結合する一般的な方法です。
例えば、`Prod Nat String`には`Nat`と`String`が含まれます。
言い換えれば、`PPoint Nat`は`Prod Nat Nat`に置き換えることができます。
`Prod`は、C#のタプル、Kotlinの`Pair`や`Triple`型、C++の`tuple`と非常に似ています。
多くのアプリケーションでは、`Point`のような単純なケースでさえ、独自の構造体を定義することが最適です。なぜなら、ドメイン用語を使用することでコードを読みやすくすることができるからです。
さらに、構造体型を定義することで、異なるドメイン概念に異なる型を割り当てることにより、それらが混同されるのを防ぐことで、より多くのエラーをキャッチするのに役立ちます。

一方で、新しい型を定義するオーバーヘッドが価値がない場合もあります。
さらに、一部のライブラリは「ペア」よりも具体的な概念がないほど十分に汎用的です。
最後に、標準ライブラリには組み込みのペア型で作業を容易にするさまざまな便利な機能が含まれています。

標準のペア構造体は`Prod`と呼ばれます。
```lean
{{#example_decl Examples/Intro.lean Prod}}
```
リストは非常に頻繁に使用されるため、より読みやすくするための特別な構文があります。
同じ理由で、製品型とそのコンストラクタにも特別な構文があります。
通常、`Prod α β`型は`α × β`と書かれ、集合のデカルト積の通常の表記を反映しています。
同様に、`Prod`の通常の数学的表記が利用可能です。
つまり、次のように書く代わりに：
```lean
{{#example_decl Examples/Intro.lean fivesStruct}}
```
次のように書くだけで十分です：
```lean
{{#example_decl Examples/Intro.lean fives}}
```

両方の表記は右結合です。
これは、次の定義が同等であることを意味します：
```lean
{{#example_decl Examples/Intro.lean sevens}}

 {{#example_decl Examples/Intro.lean sevensNested}}
```
言い換えれば、2つ以上の型のすべての製品、およびそれに対応するコンストラクタは、実際には背後でネストされた製品とネストされたペアです。

### `Sum`

`Sum`データ型は、2つの異なる型の値の間で選択を可能にする一般的な方法です。
例えば、`Sum String Int`は`String`または`Int`です。
`Prod`と同様に、非常に汎用的なコードを書く場合、ドメイン固有の型がない非常に小さなコードセクションで使用する場合、または標準ライブラリに便利な機能が含まれている場合に使用するべきです。
ほとんどの状況では、カスタムの帰納型を使用する方が読みやすく、保守が容易です。

`Sum α β` 型の値は、型 `α` の値に適用されたコンストラクタ `inl` または型 `β` の値に適用されたコンストラクタ `inr` のいずれかです:
```lean
{{#example_decl Examples/Intro.lean Sum}}
```
これらの名前はそれぞれ "left injection" と "right injection" の略です。
`Prod` に対してデカルト積の記法が使われるように、`Sum` には "circled plus" の記法が使われ、`α ⊕ β` は `Sum α β` の別の表記方法です。
`Sum.inl` と `Sum.inr` には特別な構文はありません。

例えば、ペットの名前が犬の名前か猫の名前のどちらかである場合、それらを文字列の和として型を導入することができます:
```lean
{{#example_decl Examples/Intro.lean PetName}}
```
実際のプログラムでは、通常、情報的なコンストラクタ名を持つカスタムの帰納的データ型を定義する方が良いでしょう。
ここでは、`Sum.inl` は犬の名前に、`Sum.inr` は猫の名前に使用されます。
これらのコンストラクタは、動物の名前のリストを書くために使用することができます:
```lean
{{#example_decl Examples/Intro.lean animals}}
```
パターンマッチングを使用して、2つのコンストラクタを区別することができます。
例えば、動物の名前のリストの中で犬の数（つまり `Sum.inl` コンストラクタの数）を数える関数は次のようになります:
```lean
{{#example_decl Examples/Intro.lean howManyDogs}}
```
関数呼び出しは中置演算子よりも先に評価されるので、`howManyDogs morePets + 1` は `(howManyDogs morePets) + 1` と同じです。
予想通り、`{{#example_in Examples/Intro.lean dogCount}}` は `{{#example_out Examples/Intro.lean dogCount}}` を生成します。

 ### `Unit`

 `Unit` は引数のない単一のコンストラクタ、つまり `unit` を持つ型です。
言い換えれば、それは前述のコンストラクタが何の引数も適用されていない単一の値を記述します。
`Unit` は以下のように定義されています:
```lean
{{#example_decl Examples/Intro.lean Unit}}
```

 単独では、`Unit` は特に有用ではありません。
しかし、多相的なコードでは、欠けているデータのプレースホルダーとして使用することができます。
例えば、以下の帰納的データ型は算術式を表します:
```lean
{{#example_decl Examples/Intro.lean ArithExpr}}
```
型引数 `ann` は注釈を意味し、各コンストラクタには注釈が付けられています。
パーサーから来る式はソース位置で注釈が付けられるかもしれないので、`ArithExpr SourcePos` の戻り型は、パーサーが各サブ式に `SourcePos` を置いたことを保証します。
しかし、パーサーから来ない式にはソース位置がないので、その型は `ArithExpr Unit` になります。

 
さらに、Lean のすべての関数には引数があるため、他の言語の引数のない関数は `Unit` 引数を取る関数として表されることがあります。
戻り値の位置では、`Unit` 型は C から派生した言語の `void` に似ています。
C 系の言語では、`void` を返す関数は呼び出し元に制御を返しますが、興味深い値は返しません。
`Unit` は意図的に興味のない値であり、型システムに特別な `void` 機能を要求することなく、これを表現することができます。
Unit のコンストラクタは空の括弧として書くことができます: `{{#example_in Examples/Intro.lean unitParens}} : {{#example_out Examples/Intro.lean unitParens}}`.

 ### `Empty`

 `Empty` データ型にはコンストラクタがまったくありません。
したがって、到達不可能なコードを示します。なぜなら、どのような呼び出しのシリーズも `Empty` 型の値で終了することは決してありません。

 `Empty` は `Unit` ほど頻繁には使用されません。
しかし、いくつかの特殊な文脈で役立ちます。
多くの多相的データ型は、すべてのコンストラクタで型引数を使用するわけではありません。
例えば、`Sum.inl` と `Sum.inr` は `Sum` の型引数のうちの1つしか使用しません。
`Sum` の型引数として `Empty` を使用することで、プログラムの特定のポイントで1つのコンストラクタを除外することができます。
これにより、追加の制約がある文脈で汎用的なコードを使用することができます。

 ### 命名: 和、積、および単位

一般的に、複数のコンストラクタを提供する型は _和型_ と呼ばれ、単一のコンストラクタが複数の引数を取る型は _積型_ と呼ばれます。
これらの用語は、通常の算術で使用される和と積に関連しています。
関与する型が有限数の値を含む場合に、この関係が最も簡単に見られます。
もし `α` と `β` がそれぞれ \\( n \\) と \\( k \\) の異なる値を含む型であるならば、`α ⊕ β` は \\( n + k \\) の異なる値を含み、`α × β` は \\( n \times k \\) の異なる値を含みます。
例えば、`Bool` には `true` と `false` の2つの値があり、`Unit` には1つの値 `Unit.unit` があります。
積 `Bool × Unit` には `(true, Unit.unit)` と `(false, Unit.unit)` の2つの値があり、和 `Bool ⊕ Unit` には `Sum.inl true`、`Sum.inl false`、および `Sum.inr unit` の3つの値があります。
同様に、\\( 2 \times 1 = 2 \\) であり、\\( 2 + 1 = 3 \\) です。

 ## 遭遇する可能性のあるメッセージ

すべての定義可能な構造や帰納的型が `Type` 型を持つわけではありません。
特に、コンストラクタが任意の型を引数として取る場合、帰納的型は異なる型を持たなければなりません。
これらのエラーは通常、"universe levels" について何かを述べています。
例えば、この帰納的型に対して:
```lean
{{#example_in Examples/Intro.lean TypeInType}}
```
Lean は以下のエラーを出します:
```output error
{{#example_out Examples/Intro.lean TypeInType}}
```
後の章で、これがなぜそうなのか、および定義をどのように変更して動作させるかについて説明します。
今のところ、型をコンストラクタ全体の引数としてではなく、帰納的型全体の引数として試してみてください。

同様に、コンストラクタの引数が定義されているデータ型を引数として取る関数である場合、その定義は拒否されます。
例えば:
```lean
{{#example_in Examples/Intro.lean Positivity}}
```
は以下のメッセージを出力します:
```output error
{{#example_out Examples/Intro.lean Positivity}}
```
技術的な理由から、これらのデータ型を許可すると、Lean の内部論理を損なう可能性があり、定理証明者としての使用に適さなくなる可能性があります。

帰納的型に引数を忘れると、混乱するメッセージが出ることもあります。
例えば、`ctor` の型で `MyType` に引数 `α` が渡されていない場合:
```lean
{{#example_in Examples/Intro.lean MissingTypeArg}}
```
Lean は以下のエラーで応答します:
```output error
{{#example_out Examples/Intro.lean MissingTypeArg}}
```
エラーメッセージは、`MyType` の型が `Type → Type` であり、それ自体が型を記述していないと言っています。
`MyType` は引数を必要とし、実際の正真正銘の型になります。

同じメッセージが、定義の型シグネチャなど他の文脈で型引数が省略された場合にも表示されることがあります:
```lean
{{#example_decl Examples/Intro.lean MyTypeDef}}

 {{#example_in Examples/Intro.lean MissingTypeArg2}}
```

## 練習問題

 * リストの最後のエントリを見つける関数を書いてください。`Option`を返すようにしてください。
 * 与えられた述語を満たすリストの最初のエントリを見つける関数を書いてください。定義を `def List.findFirst? {α : Type} (xs : List α) (predicate : α → Bool) : Option α :=` で始めてください。
 * ペアの二つのフィールドを交換する `Prod.swap` 関数を書いてください。定義を `def Prod.swap {α β : Type} (pair : α × β) : β × α :=` で始めてください。
 * `PetName` の例をカスタムデータ型を使用して書き直し、`Sum`を使用するバージョンと比較してください。
 * 二つのリストをペアのリストに組み合わせる `zip` 関数を書いてください。結果のリストは、入力リストの短い方と同じ長さになるようにしてください。定義を `def zip {α β : Type} (xs : List α) (ys : List β) : List (α × β) :=` で始めてください。
 * リストの最初の \\( n \\) エントリを返す多相関数 `take` を書いてください。ここで \\( n \\) は `Nat` です。リストに `n` エントリ未満が含まれている場合、結果のリストは入力リストと同じになるようにしてください。`{{#example_in Examples/Intro.lean takeThree}}` は `{{#example_out Examples/Intro.lean takeThree}}` を生成し、`{{#example_in Examples/Intro.lean takeOne}}` は `{{#example_out Examples/Intro.lean takeOne}}` を生成するようにしてください。
 * 型と算術の間の類似性を使用して、積が和に分配される関数を書いてください。言い換えると、型は `α × (β ⊕ γ) → (α × β) ⊕ (α × γ)` であるべきです。
 * 型と算術の間の類似性を使用して、二倍の乗算を和に変える関数を書いてください。言い換えると、型は `Bool × α → α ⊕ α` であるべきです。
