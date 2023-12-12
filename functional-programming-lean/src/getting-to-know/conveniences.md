# 追加の便利機能

Leanには、プログラムをより簡潔にするための便利機能がいくつか含まれています。

## 自動的な暗黙の引数

Leanで多相関数を書く際、すべての暗黙の引数を列挙する必要は通常ありません。
代わりに、それらを単に言及することができます。
Leanがその型を決定できる場合、それらは自動的に暗黙の引数として挿入されます。
言い換えると、`length`の以前の定義：
```lean
{{#example_decl Examples/Intro.lean lengthImp}}
```
は、`{α : Type}`なしで書くことができます：
```lean
{{#example_decl Examples/Intro.lean lengthImpAuto}}
```
これにより、多くの暗黙の引数を取る高度に多相的な定義を大幅に簡素化することができます。

## パターンマッチング定義

`def`を使用して関数を定義する際、引数に名前を付けてすぐにパターンマッチングで使用することがよくあります。
例えば、`length`では、引数`xs`は`match`でのみ使用されます。
このような状況では、引数に名前を付けずに、`match`式のケースを直接書くことができます。

最初のステップは、引数の型をコロンの右側に移動させ、戻り値の型が関数型になるようにすることです。
例えば、`length`の型は`List α → Nat`です。
次に、`:=`をパターンマッチの各ケースに置き換えます：
```lean
{{#example_decl Examples/Intro.lean lengthMatchDef}}
```

この構文は、複数の引数を取る関数を定義するためにも使用できます。
この場合、それらのパターンはコンマで区切られます。
例えば、`drop`は数値\\( n \\)とリストを取り、最初の\\( n \\)エントリを削除した後のリストを返します。
```lean
{{#example_decl Examples/Intro.lean drop}}
```

名前付き引数とパターンは、同じ定義で使用することもできます。
例えば、デフォルト値とオプショナル値を取り、オプショナル値が`none`の場合にデフォルトを返す関数は次のように書くことができます：
```lean
{{#example_decl Examples/Intro.lean fromOption}}
```
この関数は標準ライブラリでは`Option.getD`と呼ばれ、ドット記法で呼び出すことができます：
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

計算の中間ステップに名前を付けることはしばしば有用です。
多くの場合、中間値はそれ自体で有用な概念を表しており、それらに明示的に名前を付けることでプログラムを読みやすくすることができます。
他の場合では、中間値が複数回使用されます。
他の多くの言語と同様に、Leanで同じコードを2回書くと、それが2回計算されることになりますが、結果を変数に保存すると、計算の結果が保存され再利用されます。

例えば、`unzip`はペアのリストをリストのペアに変換する関数です。
ペアのリストが空の場合、`unzip`の結果は空のリストのペアです。
ペアのリストの先頭にペアがある場合、ペアの2つのフィールドはリストの残りの部分をアンジップする結果に追加されます。
この`unzip`の定義は、その説明に正確に従っています：
```lean
{{#example_decl Examples/Intro.lean unzipBad}}
```
残念ながら、問題があります：このコードは必要以上に遅いです。
リストのペアの各エントリは2回の再帰呼び出しを引き起こし、これによりこの関数は指数関数的な時間を要します。
しかし、両方の再帰呼び出しは同じ結果になるため、再帰呼び出しを2回行う理由はありません。

Leanでは、`let`を使用して再帰呼び出しの結果に名前を付け、それによって保存することができます。
`let`によるローカル定義は、`def`によるトップレベルの定義に似ています：ローカルに定義される名前、必要に応じて引数、型シグネチャを取り、その後に`:=`に続く本体があります。
ローカル定義が利用可能な式（`let`式の_body_と呼ばれます）は、新しい行にあり、ファイル内の`let`キーワードの列よりも少ないか等しい列で始まる必要があります。
例えば、`unzip`では`let`を次のように使用できます：
```lean
{{#example_decl Examples/Intro.lean unzip}}
```
`let`を単一行で使用するには、ローカル定義と本体をセミコロンで区切ります。

`let`によるローカル定義は、データ型のすべてのケースに一致するのに十分な1つのパターンがある場合にパターンマッチングを使用することもできます。
`unzip`の場合、再帰呼び出しの結果はペアです。
ペアには単一のコンストラクタしかないため、名前`unzipped`はペアパターンに置き換えることができます：
```lean
{{#example_decl Examples/Intro.lean unzipPat}}
```
`let`でのパターンの巧みな使用は、手動でアクセサ呼び出しを書くよりも、コードを読みやすくすることができます。

`let`と`def`の最大の違いは、再帰的な`let`定義は`let rec`と書くことによって明示的に示されなければならないことです。
例えば、リストを反転させる一つの方法には、この定義にあるような再帰的なヘルパー関数が関与します：
```lean
{{#example_decl Examples/Intro.lean reverse}}
```
ヘルパー関数は入力リストを下っていき、一度に一つのエントリを`soFar`に移動させます。
入力リストの終わりに到達すると、`soFar`には入力の反転バージョンが含まれています。

## 型推論

多くの状況で、Leanは自動的に式の型を決定することができます。
これらの場合、トップレベルの定義（`def`を使用）とローカル定義（`let`を使用）の両方から明示的な型を省略することができます。
例えば、`unzip`への再帰呼び出しには注釈が必要ありません：
```lean
{{#example_decl Examples/Intro.lean unzipNT}}
```

経験則として、リテラル値（文字列や数値など）の型を省略することは通常機能しますが、Leanはリテラル数値に意図した型よりも具体的な型を選択することがあります。
関数適用の型は、すでに引数の型と戻り値の型を知っているため、Leanは通常決定することができます。
関数定義の戻り値の型を省略することはしばしば機能しますが、関数引数には通常注釈が必要です。
例の`unzipped`のように、関数ではない定義は、その本体が型注釈を必要としない場合、型注釈を必要としません。

明示的な`match`式を使用する場合、`unzip`の戻り値の型を省略することが可能です：
```lean
{{#example_decl Examples/Intro.lean unzipNRT}}
```

一般的に言って、型注釈は多すぎるくらいがちょうど良いとされています。まず第一に、明示的な型はコードに対する仮定を読者に伝えます。Leanが自身で型を決定できるとしても、繰り返しLeanに型情報を問い合わせることなくコードを読む方が簡単です。第二に、明示的な型はエラーの特定を助けます。プログラムが型について明示的であればあるほど、エラーメッセージは有益な情報を提供できます。これは、非常に表現力豊かな型システムを持つLeanのような言語では特に重要です。第三に、明示的な型は最初にプログラムを書くことを容易にします。型は仕様であり、コンパイラのフィードバックは仕様を満たすプログラムを書く際の有用なツールとなります。最後に、Leanの型推論はベストエフォートシステムです。Leanの型システムが非常に表現力豊かであるため、すべての式に対して「最良」または最も一般的な型を見つけることはありません。これは、型を得たとしても、それが特定のアプリケーションにとって「正しい」型であるとは限らないことを意味します。例えば、`14`は`Nat`または`Int`になり得ます：

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

型注釈がないと、混乱するエラーメッセージが表示されることがあります。`unzip`の定義からすべての型を省略すると：

```lean
{{#example_in Examples/Intro.lean unzipNoTypesAtAll}}
```

`match`式に関するメッセージが表示されます：

```output error
{{#example_out Examples/Intro.lean unzipNoTypesAtAll}}
```

これは、`match`が検査される値の型を知る必要があるが、その型が利用できなかったためです。"メタ変数"はプログラムの未知の部分であり、エラーメッセージでは`?m.XYZ`と書かれています。これらは[多相性に関するセクション](polymorphism.md)で説明されています。このプログラムでは、引数に型注釈が必要です。

非常に単純なプログラムでさえ型注釈を必要とすることがあります。例えば、恒等関数は渡された引数をそのまま返します。引数と型注釈を含むと、次のようになります：

```lean
{{#example_decl Examples/Intro.lean idA}}
```

Leanは自身で戻り値の型を決定できます：

```lean
{{#example_decl Examples/Intro.lean idB}}
```

しかし、引数の型を省略するとエラーが発生します：

```lean
{{#example_in Examples/Intro.lean identNoTypes}}
```
```output error
{{#example_out Examples/Intro.lean identNoTypes}}
```

一般的に、「推論に失敗した」と言うメッセージやメタ変数に言及するメッセージは、より多くの型注釈が必要である兆候です。特にLeanを学んでいる間は、ほとんどの型を明示的に提供すると役立ちます。

## 同時マッチング

パターンマッチング式は、パターンマッチング定義と同様に、一度に複数の値にマッチすることができます。検査される式とそれにマッチするパターンは、定義で使用される構文と同様に、間にコンマを入れて書かれます。ここに`drop`の同時マッチングを使用したバージョンがあります：

```lean
{{#example_decl Examples/Intro.lean dropMatch}}
```

## 自然数のパターン

[データ型とパターン](datatypes-and-patterns.md)のセクションでは、`even`は次のように定義されていました：

```lean
{{#example_decl Examples/Intro.lean even}}
```

リストのパターンを`List.cons`や`List.nil`を直接使用するよりも読みやすくする特別な構文があるように、自然数もリテラル数値と`+`を使用してマッチングできます。例えば、`even`は次のように定義することもできます：

```lean
{{#example_decl Examples/Intro.lean evenFancy}}
```

この表記法では、`+`パターンの引数は異なる役割を果たします。舞台裏では、左の引数（上記の`n`）はいくつかの`Nat.succ`パターンへの引数になり、右の引数（上記の`1`）はパターンの周りにどれだけの`Nat.succ`をラップするかを決定します。`halve`の明示的なパターンは、`Nat`を2で割って余りを捨てるものです：

```lean
{{#example_decl Examples/Intro.lean explicitHalve}}
```

これは数値リテラルと`+`に置き換えることができます：

```lean
{{#example_decl Examples/Intro.lean halve}}
```

舞台裏では、両方の定義は完全に同等です。覚えておくべきことは、`halve n + 1`は`(halve n) + 1`と同等であり、`halve (n + 1)`ではありません。

この構文を使用する場合、`+`の第二引数は常にリテラルの`Nat`であるべきです。加算は交換可能ですが、パターン内で引数を反転させると、次のようなエラーが発生することがあります：

```lean
{{#example_in Examples/Intro.lean halveFlippedPat}}
```
```output error
{{#example_out Examples/Intro.lean halveFlippedPat}}
```

この制限により、Leanはパターン内の`+`表記のすべての使用を基本的な`Nat.succ`の使用に変換でき、舞台裏で言語をよりシンプルに保つことができます。

## 無名関数

Leanでは、関数をトップレベルで定義する必要はありません。表現としての関数は`fun`構文を使って生成されます。関数式はキーワード`fun`で始まり、その後に一つ以上の引数が続き、`=>`を使用して戻り値の式から引数が分離されます。例えば、数値に1を加える関数は次のように書かれます：

```lean
{{#example_in Examples/Intro.lean incr}}
```
```output info
{{#example_out Examples/Intro.lean incr}}
```

型注釈は`def`と同じ方法で書かれ、括弧とコロンを使用します：

```lean
{{#example_in Examples/Intro.lean incrInt}}
```
```output info
{{#example_out Examples/Intro.lean incrInt}}
```

同様に、暗黙の引数は波括弧で書かれることがあります：

```lean
{{#example_in Examples/Intro.lean identLambda}}
```
```output info
{{#example_out Examples/Intro.lean identLambda}}
```

このスタイルの無名関数式は、しばしば_ラムダ式_と呼ばれます。なぜなら、プログラミング言語の数学的記述で一般的に使用される記法では、Leanのキーワード`fun`のところにギリシャ文字のλ（ラムダ）が使用されるからです。Leanでは`λ`を`fun`の代わりに使用することが許されていますが、一般的には`fun`を書くことが最も一般的です。

無名関数は`def`で使用される複数パターンスタイルもサポートしています。例えば、自然数の前任者が存在する場合にそれを返す関数は次のように書かれます：

```lean
{{#example_in Examples/Intro.lean predHuh}}
```
```output info
{{#example_out Examples/Intro.lean predHuh}}
```

Lean自身による関数の説明には名前付き引数と`match`式があります。Leanの便利な構文の省略形は、舞台裏でよりシンプルな構文に展開され、時には抽象化が漏れることがあります。

引数を取る`def`を使用した定義は、関数式として書き直すことができます。例えば、引数を2倍にする関数は次のように書かれます：

```lean
{{#example_decl Examples/Intro.lean doubleLambda}}
```

無名関数が非常にシンプルな場合、例えば `{{#example_eval Examples/Intro.lean incrSteps 0}}` のような場合、関数を作成するための構文はかなり冗長になることがあります。
特にその例では、関数を導入するために6つの非空白文字が使用され、その本体はわずか3つの非空白文字から成り立っています。
これらのシンプルなケースのために、Leanは省略形を提供しています。
括弧で囲まれた式の中で、中央のドット文字 `·` は引数を表すことができ、括弧内の式が関数の本体になります。
その特定の関数は `{{#example_eval Examples/Intro.lean incrSteps 1}}` としても書くことができます。

中央のドットは常に _最も近い_ 括弧のセットから関数を作成します。
例えば、`{{#example_eval Examples/Intro.lean funPair 0}}` は数値のペアを返す関数であり、`{{#example_eval Examples/Intro.lean pairFun 0}}` は関数と数値のペアです。
複数のドットが使用される場合、それらは左から右へと引数になります：
```lean
{{#example_eval Examples/Intro.lean twoDots}}
```

無名関数は `def` や `let` を使用して定義された関数と全く同じ方法で適用することができます。
コマンド `{{#example_in Examples/Intro.lean applyLambda}}` の結果は以下の通りです：
```output info
{{#example_out Examples/Intro.lean applyLambda}}
```
一方で `{{#example_in Examples/Intro.lean applyCdot}}` の結果は以下の通りです：
```output info
{{#example_out Examples/Intro.lean applyCdot}}
```

## 名前空間

Lean内の各名前は _名前空間_ に存在します。これは名前の集合です。
名前は `.` を使用して名前空間に配置されるため、`List.map` は `List` 名前空間の `map` という名前です。
異なる名前空間の名前は、それ以外が同一であっても互いに衝突しません。
これは `List.map` と `Array.map` が異なる名前であることを意味します。
名前空間は入れ子にすることができるので、`Project.Frontend.User.loginTime` は入れ子になった名前空間 `Project.Frontend.User` の `loginTime` という名前です。

名前は直接名前空間内で定義することができます。
例えば、`double` という名前は `Nat` 名前空間で定義することができます：
```lean
{{#example_decl Examples/Intro.lean NatDouble}}
```
`Nat` は型の名前でもあるため、ドット表記を使用して `Nat.double` を型 `Nat` の式に対して呼び出すことができます：
```lean
{{#example_in Examples/Intro.lean NatDoubleFour}}
```
```output info
{{#example_out Examples/Intro.lean NatDoubleFour}}
```

名前空間内で直接名前を定義することに加えて、`namespace` と `end` コマンドを使用して一連の宣言を名前空間に配置することができます。
例えば、これにより `triple` と `quadruple` が `NewNamespace` 名前空間で定義されます：
```lean
{{#example_decl Examples/Intro.lean NewNamespace}}
```
それらを参照するには、名前の前に `NewNamespace.` を付けます：
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

名前空間は _開く_ ことができ、それにより名前空間内の名前を明示的な修飾なしで使用することができます。
式の前に `open MyNamespace in` を書くと、`MyNamespace` の内容が式で利用可能になります。
例えば、`timesTwelve` は `NewNamespace` を開いた後に `quadruple` と `triple` の両方を使用します：
```lean
{{#example_decl Examples/Intro.lean quadrupleOpenDef}}
```
名前空間はコマンドの前にも開くことができます。
これにより、コマンドのすべての部分が名前空間の内容を参照することができ、単一の式だけでなくなります。
これを行うには、コマンドの前に `open ... in` を配置します。
```lean
{{#example_in Examples/Intro.lean quadrupleNamespaceOpen}}
```
```output info
{{#example_out Examples/Intro.lean quadrupleNamespaceOpen}}
```
関数のシグネチャは名前の完全な名前空間を示します。
名前空間は、ファイルの残りのすべてのコマンドに対して _すべて_ 開くこともできます。
これを行うには、トップレベルの使用から `in` を省略するだけです。

## if let

値が和型（sum type）である場合、しばしば関心のあるコンストラクタは一つだけであることが多いです。例えば、Markdownのインライン要素のサブセットを表すこの型があったとします：
```lean
{{#example_decl Examples/Intro.lean Inline}}
```
文字列要素を認識し、その内容を抽出する関数は以下のように書けます：
```lean
{{#example_decl Examples/Intro.lean inlineStringHuhMatch}}
```
この関数の本体を書く別の方法では、`if`と`let`を一緒に使います：
```lean
{{#example_decl Examples/Intro.lean inlineStringHuh}}
```
これはパターンマッチングの`let`構文に非常に似ています。違いは、和型に使えることで、`else`のケースでフォールバックが提供されている点です。一部の文脈では、`match`の代わりに`if let`を使うことでコードが読みやすくなる場合があります。

## 位置による構造体引数

[構造体に関するセクション](structures.md)では、構造体を構築する二つの方法を提示しています：
 1. コンストラクタを直接呼び出す方法、例：`{{#example_in Examples/Intro.lean pointCtor}}`。
 2. ブレース記法を使う方法、例：`{{#example_in Examples/Intro.lean pointBraces}}`。

一部の文脈では、引数を名前ではなく位置によって渡すことが便利ですが、コンストラクタを直接指名することなく。たとえば、類似の構造体タイプをいくつか定義することで、ドメインの概念を分けておくことができますが、コードを読む自然な方法ではそれぞれを本質的にタプルとして扱うかもしれません。これらの文脈では、引数を角括弧`⟨`と`⟩`で囲むことができます。`Point`は `{{#example_in Examples/Intro.lean pointPos}}`のように書けます。注意してください！これらの括弧は小なり記号`<`と大なり記号`>`に似ていますが、異なるものです。それらは`\<`と`\>`を使って入力できます。

名前付きコンストラクタ引数のブレース記法と同様に、この位置による構文は、Leanがプログラム内の型注釈または他の型情報から構造体の型を判断できる文脈でのみ使用できます。例えば、`{{#example_in Examples/Intro.lean pointPosEvalNoType}}`は次のようなエラーを引き起こします：
```output error
{{#example_out Examples/Intro.lean pointPosEvalNoType}}
```
エラーのメタ変数は、利用可能な型情報がないために発生します。注釈を追加することで、例えば`{{#example_in Examples/Intro.lean pointPosWithType}}`のように問題が解決します：
```output info
{{#example_out Examples/Intro.lean pointPosWithType}}
```

## 文字列補間

Leanでは、文字列の前に`s!`を付けることで　_補間_　が行われ、文字列内の波括弧で囲まれた式がその値に置き換えられます。これはPythonの`f`文字列やC#の`$`付き文字列に似ています。たとえば、
```lean
{{#example_in Examples/Intro.lean interpolation}}
```
は以下の出力を生成します：
```output info
{{#example_out Examples/Intro.lean interpolation}}
```
すべての表現が文字列に埋め込めるわけではありません。
例えば、関数を埋め込もうとするとエラーが発生します。
```lean
{{#example_in Examples/Intro.lean interpolationOops}}
```
は以下の出力を生み出します
```output info
{{#example_out Examples/Intro.lean interpolationOops}}
```
これは、関数を文字列に変換する標準的な方法がないためです。
Leanコンパイラは、さまざまな型の値を文字列に変換する方法を記述した表を維持しており、メッセージ `failed to synthesize instance` は、Leanコンパイラがその型に対する表のエントリを見つけられなかったことを意味します。
これは、[構造体に関するセクション](structures.md)で説明された `deriving Repr` 構文と同じ言語機能を使用しています。
