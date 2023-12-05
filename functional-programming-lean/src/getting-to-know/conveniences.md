# 追加の便利機能

Leanには、プログラムをより簡潔にするための便利機能が数多く含まれています。

## 自動的な暗黙の引数

Leanで多相関数を書くとき、通常はすべての暗黙の引数を列挙する必要はありません。
代わりに、単に言及することが可能です。
Leanがその型を判断できる場合は、自動的に暗黙の引数として挿入されます。
言い換えれば、以前の`length`の定義：
```lean
{{#example_decl Examples/Intro.lean lengthImp}}
```
は、`{α : Type}`を書かずに以下のように記述できます：
```lean
{{#example_decl Examples/Intro.lean lengthImpAuto}}
```
これは、多くの暗黙の引数を取る高度に多相的な定義を大幅に簡略化することができます。

## パターンマッチングによる定義

`def`を使って関数を定義する際、引数に名前を付けてただちにパターンマッチングで使用することがよくあります。
例えば、`length`では、`xs`という引数が`match`内でのみ使用されます。
このような場面では、引数に名前を付けることなく、`match`式のケースを直接記述することができます。

最初のステップは、引数の型をコロンの右側に移動し、返り値の型が関数の型になるようにすることです。
例えば、`length`の型は`List α → Nat`です。
次に、`:=`の代わりにパターンマッチの各ケースを置き換えます：
```lean
{{#example_decl Examples/Intro.lean lengthMatchDef}}
```

この構文は複数の引数を取る関数を定義するのにも使用できます。
この場合、そのパターンはコンマで区切られます。
例えば、`drop`は数\\( n \\)とリストを取り、リストから最初の\\( n \\)項目を削除したものを返します。
```lean
{{#example_decl Examples/Intro.lean drop}}
```

名前付きの引数とパターンを同じ定義で使用することもできます。
例えば、デフォルト値とオプショナル値を取り、オプショナル値が`none`の場合にデフォルトを返す関数は次のように書けます：
```lean
{{#example_decl Examples/Intro.lean fromOption}}
```
この関数は標準ライブラリでは`Option.getD`と呼ばれ、ドット記法を使って呼び出すことができます：
```lean
{{#example_in Examples/Intro.lean getD}}
```
```output info
{{#example_out Examples/Intro.lean getD}}
```
```lean
{{#example_in Examples/Intro.lean getDNone}}
```
```output info
{{#example_out Examples/Intro.lean getDNone}}
```

## ローカル定義

計算では、中間的なステップに名前を付けることがしばしば有用です。
多くの場合、中間値はそれ自体で有用な概念を表しており、明示的に名前を付けることでプログラムを読みやすくすることができます。
他の場合、中間値は複数回使用されます。
他のほとんどの言語と同じように、Leanでは同じコードを2回記述すると計算が2回行われますが、変数に結果を保存すると計算の結果が保存され再利用されます。

例えば、`unzip`はペアのリストをリストのペアに変換する関数です。
ペアのリストが空の場合、`unzip`の結果は空のリストのペアです。
ペアのリストの先頭にペアがある場合、そのペアの2つのフィールドはリストの残りの部分のunzipの結果に追加されます。
この`unzip`の定義はその説明に完全に従っています：
```lean
{{#example_decl Examples/Intro.lean unzipBad}}
```
残念ながら問題があります：このコードは必要以上に遅いです。
リストのペアの各エントリは2回の再帰呼び出しにつながり、この関数は指数的な時間を要します。
しかし、両方の再帰呼び出しは同じ結果になるため、再帰呼び出しを2回行う理由はありません。

Leanでは、`let`を使用することで再帰呼び出しの結果に名前を付け、保存することができます。
ローカル定義に使われる`let`は`def`を使ったトップレベル定義に似ています：ローカルで定義される名前、必要に応じて引数、型シグネチャがあり、その後に`:=`に続く本体が続きます。
ローカル定義が有効な式（`let`式の_body_と呼ばれる）は新しい行に書かれ、その行はファイル内の`let`キーワードのカラム以下である必要があります。
例えば、`unzip`では`let`を次のように使用できます：
```lean
{{#example_decl Examples/Intro.lean unzip}}
```
単一行に`let`を使用するには、ローカル定義を本体からセミコロンで区切ります。

`let` を使用したローカル定義では、データ型のすべてのケースに一致するのに十分な一つのパターンで、パターンマッチングを使用することもできます。
`unzip`のケースでは、再帰呼び出しの結果はペアです。
ペアには単一のコンストラクタしかないため、`unzipped`という名前をペアパターンで置き換えることができます：
```lean
{{#example_decl Examples/Intro.lean unzipPat}}
```
`let`とパターンを慎重に使用することで、手作業でアクセサ呼び出しを書くよりもコードが読みやすくなることがあります。

`let`と`def`の最大の違いは、再帰的な`let`定義は`let rec`と書くことによって明示的に示されなければならないことです。
例えば、リストを逆順にする一つの方法は再帰的なヘルパー関数を使用することです。この定義でのように：
```lean
{{#example_decl Examples/Intro.lean reverse}}
```
ヘルパー関数は入力リストを下に進むと、一度に一つのエントリを`soFar`に移動します。
入力リストの終わりに到達すると、`soFar`は入力の逆順バージョンを含んでいます。

## 型推論
多くの場面で、Leanは式の型を自動的に判断することができます。そのため、トップレベルの定義（`def`を使用）やローカル定義（`let`を使用）において、明示的な型を省略することが可能です。たとえば、`unzip`への再帰呼び出しにはアノテーションが必要ありません。
```lean
{{#example_decl Examples/Intro.lean unzipNT}}
```

一般的に、リテラル値（文字列や数値など）の型を省略してもうまくいくことが多いですが、Leanはリテラル数値に対して意図された型よりも具体的な型を選択する場合があります。関数適用については、引数の型と戻り値の型が既に分かっているため、Leanは通常その型を判断できます。関数定義における戻り値の型を省略することはよくありますが、関数引数には通常アノテーションが必要です。例の中の`unzipped`のように、関数でない定義には、その本体が型アノテーションを必要としない場合、型アノテーションは不要です。この定義の本体は関数適用です。

`unzip`の戻り値の型を省略することは、明示的な`match`式を使用する場合に可能です。
```lean
{{#example_decl Examples/Intro.lean unzipNRT}}
```

一般的には、型アノテーションを多くすることが、少なくするよりも良いとされています。まず、明示的な型はコードについての仮定を読者に伝えます。Leanが独自に型を判断できるとしても、繰り返しLeanに型情報を問い合わせることなくコードを読む方が容易です。次に、明示的な型はエラーの特定を助けます。プログラムが型について明示的であればあるほど、エラーメッセージは有益になります。これは、非常に表現力豊かな型システムを持つ言語であるLeanにおいて特に重要です。さらに、明示的な型は最初にプログラムを書く際にも役立ちます。型は仕様であり、コンパイラのフィードバックは仕様に合致するプログラムを書くための有用なツールです。最後に、Leanの型推論はベストエフォートシステムです。Leanの型システムが非常に表現力豊かであるため、すべての式に対して「最適」または「最も一般的な」型を見つけることはありません。これは、型が得られたとしても、それが特定のアプリケーションにとって「正しい」型であるという保証はないことを意味します。たとえば、`14`は`Nat`または`Int`になる可能性があります。
```lean
{{#example_in Examples/Intro.lean fourteenNat}}
```
```output info
{{#example_out Examples/Intro.lean fourteenNat}}
```
```lean
{{#example_in Examples/Intro.lean fourteenInt}}
```
```output info
{{#example_out Examples/Intro.lean fourteenInt}}
```

型アノテーションが欠けていると、混乱を招くエラーメッセージが表示されることがあります。`unzip`の定義からすべての型を省略すると、
```lean
{{#example_in Examples/Intro.lean unzipNoTypesAtAll}}
```

`match`式に関するメッセージが表示されます。
```output error
{{#example_out Examples/Intro.lean unzipNoTypesAtAll}}
```
これは、`match`が検査される値の型を知る必要があるが、その型が利用できなかったためです。「メタ変数」とは

、プログラムの未知の部分を表し、エラーメッセージでは`?m.XYZ`として記述されます。これらは[多相性に関するセクション](polymorphism.md)で説明されています。このプログラムでは、引数に対する型アノテーションが必要です。

非常に単純なプログラムでさえ、型アノテーションを必要とすることがあります。たとえば、アイデンティティ関数は、渡された引数をそのまま返すだけです。引数と型アノテーションを含む場合、このようになります。Leanは戻り値の型を自分で判断することができます。しかし、引数の型を省略すると、エラーが発生します。
```lean
{{#example_decl Examples/Intro.lean idA}}
```
Lean is capable of determining the return type on its own:
```lean
{{#example_decl Examples/Intro.lean idB}}
```
Omitting the argument type, however, causes an error:
```lean
{{#example_in Examples/Intro.lean identNoTypes}}
```
```output error
{{#example_out Examples/Intro.lean identNoTypes}}
```

一般的に、「推論に失敗しました」といったメッセージや、メタ変数に言及するメッセージは、より多くの型アノテーションが必要であることを示しています。特にLeanを学び始めた段階では、ほとんどの型を明示的に提供することが有用です。

## 同時マッチング

パターンマッチング式は、パターンマッチングの定義と同様に、複数の値に一度にマッチさせることができます。検査される式とそれらがマッチするパターンは、定義で使用される構文と同様に、間にカンマを挟んで記述されます。ここに、同時マッチングを使用した`drop`のバージョンがあります。
```lean
{{#example_decl Examples/Intro.lean dropMatch}}
```

## 自然数のパターン

[データ型とパターン](datatypes-and-patterns.md)のセクションで、`even`は次のように定義されました。
```lean
{{#example_decl Examples/Intro.lean even}}
```
`List.cons`や`List.nil`を直接使用するよりも読みやすいリストパターンの特別な構文があるように、自然数もリテラル数値と`+`を使用してマッチさせることができます。たとえば、`even`は以下のように定義することもできます。
```lean
{{#example_decl Examples/Intro.lean evenFancy}}
```

この表記法では、`+`パターンの引数は異なる役割を果たします。背景では、左側の引数（上記の`n`）はいくつかの`Nat.succ`パターンへの引数になり、右側の引数（上記の`1`）はそのパターンを囲む`Nat.succ`の数を決定します。`halve`の明示的なパターンは、`Nat`を2で割り、余りを捨てますが、
```lean
{{#example_decl Examples/Intro.lean explicitHalve}}
```
は数値リテラルと`+`に置き換えることができます。
```lean
{{#example_decl Examples/Intro.lean halve}}
```
背景では、両方の定義は完全に等価です。覚えておくべきは、`halve n + 1`は`(halve n) + 1`に等しく、`halve (n + 1)`には等しくありません。

この構文を使用する場合、`+`の第二引数は常にリテラルの`Nat`であるべきです。加算は可換ですが、パターン内で引数を反転させると、以下のようなエラーが発生する可能性があります。
```lean
{{#example_in Examples/Intro.lean halveFlippedPat}}
```
```output error
{{#example_out Examples/Intro.lean halveFlippedPat}}
```
この制限により、Leanはパターン内の`+`表記のすべての使用を背景の`Nat.succ`の使用に変換し、言語をシンプルに保つことができます。

## 無名関数

Leanでは、関数はトップレベルで定義される必要はありません。
表現として、関数は`fun`構文を使用して生成されます。
関数式は、`fun`というキーワードで始まり、それに続いて一つ以上の引数があり、`=>`を使用して戻り値の表現と区切られます。
例えば、数に1を加える関数は以下のように書けます：
```lean
{{#example_in Examples/Intro.lean incr}}
```
```output info
{{#example_out Examples/Intro.lean incr}}
```
型注釈は、`def`で使用されるのと同じ方法で書かれます。括弧とコロンを使用しています：
```lean
{{#example_in Examples/Intro.lean incrInt}}
```
```output info
{{#example_out Examples/Intro.lean incrInt}}
```
同様に、暗黙の引数は中括弧を使用して記述されます：
```lean
{{#example_in Examples/Intro.lean identLambda}}
```
```output info
{{#example_out Examples/Intro.lean identLambda}}
```
このスタイルの無名関数式は、しばしば_ラムダ式_と呼ばれます。プログラミング言語の数学的記述では、Leanの`fun`キーワードの代わりにギリシャ文字のλ（ラムダ）が使われるためです。
Leanでは`λ`を`fun`の代わりに使用することが許されていますが、一般的には`fun`と書かれます。

無名関数は、`def`で使用される複数パターンスタイルもサポートしています。
例えば、自然数の前任者を返す関数は以下のように書けます：
```lean
{{#example_in Examples/Intro.lean predHuh}}
```
```output info
{{#example_out Examples/Intro.lean predHuh}}
```
Lean自身の関数の記述には、名前付きの引数と`match`表現があります。
Leanの便利な構文の省略形は、舞台裏でより単純な構文に展開され、時にはその抽象化が漏れることがあります。

引数を取る`def`を使用した定義は、関数式として書き直すことができます。
例えば、引数を二倍にする関数は次のように書けます：
```lean
{{#example_decl Examples/Intro.lean doubleLambda}}
```

無名関数が非常に単純な場合、例えば`{{#example_eval Examples/Intro.lean incrSteps 0}}`のような場合、関数を作成するための構文はかなり冗長になることがあります。
その特定の例では、関数を導入するために6文字の非空白文字が使用されており、その本体は3文字の非空白文字のみで構成されています。
これらの単純なケースのために、Leanは省略形を提供しています。
括弧で囲まれた式の中で、中央のドット文字`·`は引数として立ち、括弧内の式は関数の本体となります。
その特定の関数は、`{{#example_eval Examples/Intro.lean incrSteps 1}}`とも書けます。

中央のドットは、常に_最も近い_周囲の括弧セットから関数を作成します。
例えば、`{{#example_eval Examples/Intro.lean funPair 0}}`は数のペアを返す関数であり、`{{#example_eval Examples/Intro.lean pairFun 0}}`は関数と数のペアです。
複数のドットを使用すると、それらは左から右へ引数になります。以下の例では、この機能を示しています：
```lean
{{#example_eval Examples/Intro.lean twoDots}}
```

無名関数は、`def`や`let`を使用して定義された関数とまったく同じ方法で適用できます。以下のコマンド`{{#example_in Examples/Intro.lean applyLambda}}`の結果は次のようになります：
```output info
{{#example_out Examples/Intro.lean applyLambda}}
```
一方で、`{{#example_in Examples/Intro.lean applyCdot}}`の結果は次のようになります：
```output info
{{#example_out Examples/Intro.lean applyCdot}}
```

## 名前空間

Leanの各名前は、名前の集合である_名前空間_に存在します。名前は`.`を使用して名前空間に配置されますので、`List.map`は`List`名前空間の`map`という名前です。異なる名前空間の名前は、それ以外が同一であっても互いに競合しません。これは`List.map`と`Array.map`が異なる名前であることを意味します。名前空間はネストされることがあり、`Project.Frontend.User.loginTime`はネストされた名前空間`Project.Frontend.User`の`loginTime`という名前です。

名前は直接名前空間内で定義されることがあります。例えば、`double`という名前は`Nat`名前空間で定義されます：
```lean
{{#example_decl Examples/Intro.lean NatDouble}}
```
`Nat`も型の名前であるため、ドット表記を使用して`Nat`型の式に`Nat.double`を呼び出すことができます：
```lean
{{#example_in Examples/Intro.lean NatDoubleFour}}
```
```output info
{{#example_out Examples/Intro.lean NatDoubleFour}}
```

名前を直接名前空間に定義することに加えて、`namespace`および`end`コマンドを使用して一連の宣言を名前空間に配置することができます。たとえば、これは`triple`と`quadruple`を`NewNamespace`名前空間に定義します：
```lean
{{#example_decl Examples/Intro.lean NewNamespace}}
```
それらを参照するには、名前の前に`NewNamespace.`を付けます：
```lean
{{#example_in Examples/Intro.lean tripleNamespace}}
```
```output info
{{#example_out Examples/Intro.lean tripleNamespace}}
```
```lean
{{#example_in Examples/Intro.lean quadrupleNamespace}}
```
```output info
{{#example_out Examples/Intro.lean quadrupleNamespace}}
```

名前空間は_オープン_されることがあり、それにより名前空間内の名前を明示的な指定なしに使用できるようになります。式の前に`open MyNamespace in`と書くと、`MyNamespace`の内容がその式で利用可能になります。たとえば、`timesTwelve`は`NewNamespace`を開いた後に`quadruple`と`triple`の両方を使用します：
```lean
{{#example_decl Examples/Intro.lean quadrupleOpenDef}}
```
名前空間はコマンドの前にもオープンされることがあります。これにより、コマンドの全部分が名前空間の内容を参照できるようになり、単一の式だけではなくなります。これを行うには、コマンドの前に`open ... in`を配置します。
```lean
{{#example_in Examples/Intro.lean quadrupleNamespaceOpen}}
```
```output info
{{#example_out Examples/Intro.lean quadrupleNamespaceOpen}}
```
関数のシグネチャは名前の完全な名前空間を示します。名前空間は、ファイルの残りの部分に対して_すべての_後続のコマンドのために追加でオープンされることがあります。これを行うには、トップレベルでの`open`の使用から`in`を省略します。

## if let

合計型の値を扱う際、しばしば関心のあるのは単一のコンストラクタのみです。例えば、Markdownのインライン要素のサブセットを表すこの型が与えられた場合：
```lean
{{#example_decl Examples/Intro.lean Inline}}
```
文字列要素を認識し、その内容を抽出する関数は次のように書くことができます：
```lean
{{#example_decl Examples/Intro.lean inlineStringHuhMatch}}
```
この関数の本体を書く別の方法は、`if`と`let`を組み合わせて使用します：
```lean
{{#example_decl Examples/Intro.lean inlineStringHuh}}
```
これはパターンマッチングの`let`構文に非常に似ています。違いは、合計型に使用できることと、`else`ケースでフォールバックが提供されていることです。いくつかの文脈では、`match`の代わりに`if let`を使用することで、コードを読みやすくすることができます。

## 位置に基づく構造体引数

[構造体に関するセクション](structures.md)では、構造体を構築するための2つの方法を紹介しています：
 1. コンストラクタを直接呼び出す方法、例えば`{{#example_in Examples/Intro.lean pointCtor}}`のように。
 2. ブレース表記を使用する方法、例えば`{{#example_in Examples/Intro.lean pointBraces}}`のように。

いくつかの文脈では、名前ではなく位置に基づいて引数を渡すことが便利ですが、コンストラクタを直接指定せずに。たとえば、類似の構造体型をさまざまに定義することは、ドメインの概念を別々に保つのに役立ちますが、コードを読む自然な方法は、それぞれを基本的にタプルとして扱うことかもしれません。これらの文脈では、引数は山括弧`⟨`と`⟩`で囲むことができます。`Point`は`{{#example_in Examples/Intro.lean pointPos}}`のように書くことができます。注意してください！これらの括弧は、小なり記号`<`と大なり記号`>`に似ていますが、異なります。それぞれ`\<`と`\>`を使用して入力できます。

名前付きコンストラクタ引数のためのブレース表記と同様に、この位置に基づく構文は、Leanが型注釈またはプログラム内の他の型情報から構造体の型を決定できる文脈でのみ使用できます。たとえば、`{{#example_in Examples/Intro.lean pointPosEvalNoType}}`は次のエラーを出力します：
```output error
{{#example_out Examples/Intro.lean pointPosEvalNoType}}
```
エラーの中のメタ変数は、利用可能な型情報がないためです。`{{#example_in Examples/Intro.lean pointPosWithType}}`のように注釈を追加することで問題が解決します：
```output info
{{#example_out Examples/Intro.lean pointPosWithType}}
```

## 文字列補間

Leanでは、文字列の先頭に`s!`を付けると、文字列内の中括弧で囲まれた式がその値に置き換えられる_補間_が発生します。これはPythonの`f`文字列やC#の`$`で始まる文字列と似ています。例えば、
```lean
{{#example_in Examples/Intro.lean interpolation}}
```
は次の出力を生成します：
```output info
{{#example_out Examples/Intro.lean interpolation}}
```

すべての式が文字列に補間できるわけではありません。例えば、関数を補間しようとするとエラーになります。
```lean
{{#example_in Examples/Intro.lean interpolationOops}}
```
は次の出力を生成します：
```output info
{{#example_out Examples/Intro.lean interpolationOops}}
```
これは、関数を文字列に変換する標準的な方法がないためです。Leanコンパイラは、さまざまな型の値を文字列に変換する方法を説明する表を保持しており、「failed to synthesize instance」というメッセージは、Leanコンパイラが与えられた型のためにこの表でエントリを見つけられなかったことを意味します。これは、[構造体に関するセクション](structures.md)で説明された`deriving Repr`構文と同じ言語機能を使用しています。