# 関数と定義

Leanでは、`def`キーワードを使用して定義を導入します。例えば、文字列 `{{#example_out Examples/Intro.lean helloNameVal}}` を参照する名前 `{{#example_in Examples/Intro.lean helloNameVal}}` を定義するには、以下のように書きます：

```lean
{{#example_decl Examples/Intro.lean hello}}
```

Leanでは、新しい名前を定義するときは等号`=`ではなくコロンイコール演算子`:=`を使用します。これは`=`が既存の式間の等式を表すのに使われ、二つの異なる演算子を使用することで混乱を防ぐためです。

`{{#example_in Examples/Intro.lean helloNameVal}}`の定義では、式 `{{#example_out Examples/Intro.lean helloNameVal}}` が十分に単純なので、Leanは自動的に定義のタイプを決定できます。
しかし、ほとんどの定義はこれほど単純ではないので、通常はタイプを追加する必要があります。
これは、定義されている名前の後にコロンを使用することで行われます。

```lean
{{#example_decl Examples/Intro.lean lean}}
```

名前が定義されたら、以下のように使用することができます。
``` Lean
{{#example_in Examples/Intro.lean helloLean}}
```
出力としては、
``` Lean info
{{#example_out Examples/Intro.lean helloLean}}
```
Leanでは、定義された名前はその定義の後にのみ使用することができます。

多くの言語では、関数の定義は他の値の定義とは異なる構文を使用します。
たとえば、Pythonの関数定義は`def`キーワードで始まりますが、他の定義は等号で定義されます。
Leanでは、関数は他の値と同様に`def`キーワードを使用して定義されます。
それでも、`hello`のような定義は、それぞれ呼び出されるたびに同じ結果を返すゼロ引数関数を参照するのではなく、_直接_ その値を参照する名前を導入します。

## 関数の定義

Leanで関数を定義する方法は様々です。最も単純な方法は、関数の引数を定義のタイプの前にスペースで区切って配置することです。例えば、引数に1を加える関数は以下のように書かれます：

```lean
{{#example_decl Examples/Intro.lean add1}}
```

この関数を`#eval`でテストすると、期待通りの `{{#example_out Examples/Intro.lean add1_7}}` を得ます：
```lean
{{#example_in Examples/Intro.lean add1_7}}
```


関数が複数の引数に適用される場合には、各引数の間にスペースを配置するように、複数の引数を受け取る関数は、引数の名前とタイプの間にスペースを配置して定義されます。関数 `maximum`は、2つの`Nat`引数`n`と`k`を取り、最も大きい方の引数と等しい結果を返します。

```lean
{{#example_decl Examples/Intro.lean maximum}}
```

`maximum`のような定義された関数に引数が与えられると、結果はまず引数名を提供された値で本体内で置き換え、次に結果の本体を評価することによって決定されます。例えば：
```lean
{{#example_eval Examples/Intro.lean maximum_eval}}
```

自然数、整数、および文字列に評価される式には、これを言うタイプがあります（それぞれ`Nat`、`Int`、および`String`）。
これは関数にも当てはまります。
`Nat`を受け取り`Bool`を返す関数は`Nat → Bool`型を持ち、二つの`Nat`を受け取り`Nat`を返す関数は`Nat → Nat → Nat`型を持ちます。

特別なケースとして、Leanは関数の名前が直接`#check`と共に使用されたときに、その関数のシグネチャを返します。
`{{#example_in Examples/Intro.lean add1sig}}` を入力すると `{{#example_out Examples/Intro.lean add1sig}}` を得ます。
しかし、関数の名前を括弧で囲むことでLeanを「だます」ことができ、通常の式として処理されるため、`{{#example_in Examples/Intro.lean add1type}}` を入力すると `{{#example_out Examples/Intro.lean add1type}}` となり、`{{#example_in Examples/Intro.lean maximumType}}` を入力すると `{{#example_out Examples/Intro.lean maximumType}}` を得ます。
この矢印はASCII代替矢印 `->` としても書かれることができるので、上記の関数タイプはそれぞれ `{{#example_out Examples/Intro.lean add1typeASCII}}` および `{{#example_out Examples/Intro.lean maximumTypeASCII}}` として書かれます。

内部的には、すべての関数は実際には正確に1つの引数を期待しています。
`maximum`のように複数の引数を取るように見える関数は、実際には1つの引数を取り、その後新しい関数を返す関数です。
この新しい関数は次の引数を取り、それ以上の引数が期待されなくなるまでプロセスが続きます。
これは、`{{#example_in Examples/Intro.lean maximum3Type}}` を提供すると `{{#example_out Examples/Intro.lean maximum3Type}}` を得、`{{#example_in Examples/Intro.lean stringAppendHelloType}}` を提供すると `{{#example_out Examples/Intro.lean stringAppendHelloType}}` を得ることによって見ることができます。
複数の引数を持つ関数を実装するために関数を返す関数を使用することは、数学者ハスケル・カリーに因んで _カリー化_ と呼ばれます。
関数の矢印は右に結合するため、`Nat → Nat → Nat` は`Nat → (Nat → Nat)`と括弧で囲むべきです。

### 練習問題

 * `String -> String -> String -> String`型の関数`joinStringsWith`を定義して、最初の引数を2番目と3番目の引数の間に配置して新しい文字列を作成します。`{{#example_eval Examples/Intro.lean joinStringsWithEx 0}}`は`{{#example_eval Examples/Intro.lean joinStringsWithEx 1}}`と同じ結果になるはずです。
 * `joinStringsWith ": "`のタイプは何ですか？Leanで答えを確認してください。
 * 与えられた高さ、幅、奥行きで直方体の体積を計算する、`Nat → Nat → Nat → Nat`型の関数`volume`を定義します。

## 型の定義

ほとんどの型付きプログラミング言語には、Cの`typedef`のように型に別名を定義する手段があります。
しかし、Leanでは、型は言語のファーストクラスに属しています - それらは他のどのような式と同じです。
これは、定義が他の値だけでなく型を参照することもできることを意味します。

例えば、もし``String``とタイピングするのが面倒であれば、より短い略称``Str``を定義することができます：
```lean
{{#example_decl Examples/Intro.lean StringTypeDef}}
```
それから、``Str``を``String``の代わりに定義のタイプとして使用することが可能になります：
```lean
{{#example_decl Examples/Intro.lean aStr}}
```

これが機能する理由は、型がLeanの残りの部分と同じ規則に従うためです。
型は式であり、式では、定義済みの名前はその定義と置き換えることができます。
``Str``が``String``を意味するように定義されているため、``aStr``の定義は理にかなっています。

### 出会うかもしれないメッセージ

型のための定義を使用することで実験することは、Leanが整数リテラルのオーバーロードをサポートする方法によってより複雑になります。
もし``Nat``が短すぎる場合、より長い名前``NaturalNumber``が定義されることができます：
```lean
{{#example_decl Examples/Intro.lean NaturalNumberTypeDef}}
```
しかし、``Nat``の代わりに``NaturalNumber``を定義のタイプとして使用すると、期待された効果はありません。
特に、定義：
```lean
{{#example_in Examples/Intro.lean thirtyEight}}
```
は以下のエラーを生じます：
```output error
{{#example_out Examples/Intro.lean thirtyEight}}
```

このエラーは、Leanが数値リテラルを_オーバーロード_することを許可しているために発生します。
意味があるとき、自然数リテラルは新しいタイプについても使用することができます。まるでそれらのタイプがシステムに組み込まれているかのようです。
これは数学を表現する上で便利にするLeanのミッションの一部であり、異なる分野の数学は非常に異なる目的で数値表記を使用します。
このオーバーロードを可能にする特定の機能は、オーバーロードを探す前にすべての定義済みの名前をその定義に置き換えるわけではありません。これが上記のエラーメッセージにつながります。

この制限を回避する方法の1つは、定義の右側に`Nat`のタイプを提供し、`Nat`のオーバーロードルールを`38`に使用させることです：
```lean
{{#example_decl Examples/Intro.lean thirtyEightFixed}}
```
定義はまだタイプが正しいです。なぜなら`{{#example_eval Examples/Intro.lean NaturalNumberDef 0}}`は`{{#example_eval Examples/Intro.lean NaturalNumberDef 1}}`と同じタイプです—定義によって！

また、`Nat`に対して行われるのと同等に機能する`NaturalNumber`のオーバーロードを定義するという解決策もあります。
ただし、これにはLeanのより進んだ機能が必要になります。

最後に、`def`の代わりに`abbrev`を使用して`Nat`のための新しい名前を定義すると、オーバーロード解決中に定義済みの名前をその定義に置き換えることができます。
`abbrev`を使用して書かれた定義は常に展開されます。
例えば、
```lean
{{#example_decl Examples/Intro.lean NTypeDef}}
```
および
```lean
{{#example_decl Examples/Intro.lean thirtyNine}}
```
は問題なく受け入れられます。

内部的には、一部の定義はオーバーロード解決中に展開可能として内部的にマークされているものがあり、他はそうではありません。
展開されるべき定義は_還元可能_と呼ばれます。
還元可能性に対する制御は、Leanがスケールすることを可能にするために不可欠です：全ての定義を完全に展開すると、マシンが処理するのに時間がかかる非常に大きなタイプが生じ、ユーザーが理解するのが難しくなります。
`abbrev`で生成された定義は還元可能としてマークされます。