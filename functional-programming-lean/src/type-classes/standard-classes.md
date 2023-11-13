以下は、ご指定の`.md`ファイルを日本語に翻訳した内容です。

# 標準クラス

このセクションでは、Leanでタイプクラスを使用してオーバーロードできる多様な演算子や関数を紹介します。
各演算子や関数は型クラスのメソッドに対応しています。
C++とは異なり、Leanの中置演算子は名前付き関数の略称として定義されており、これは新しい型のオーバーロードが演算子自体ではなく、`HAdd.hAdd`のような基本名を用いて行われることを意味しています。

## 算術

ほとんどの算術演算子は、異なるタイプの引数を取り、結果の式のタイプを決める出力パラメータを持つ異種形式で利用可能です。
各々の異種演算子には、対応する同種バージョンがありますが、これは `h` を抜いて `HAdd.hAdd` が `Add.add` になることにより見つけることができます。
以下の算術演算子がオーバーロードされています：

| 演算式 | 展開 | クラス名 |
|--------|------|----------|
| `{{#example_in Examples/Classes.lean plusDesugar}}` | `{{#example_out Examples/Classes.lean plusDesugar}}` | `HAdd` |
| `{{#example_in Examples/Classes.lean minusDesugar}}` | `{{#example_out Examples/Classes.lean minusDesugar}}` | `HSub` |
| `{{#example_in Examples/Classes.lean timesDesugar}}` | `{{#example_out Examples/Classes.lean timesDesugar}}` | `HMul` |
| `{{#example_in Examples/Classes.lean divDesugar}}` | `{{#example_out Examples/Classes.lean divDesugar}}` | `HDiv` |
| `{{#example_in Examples/Classes.lean modDesugar}}` | `{{#example_out Examples/Classes.lean modDesugar}}` | `HMod` |
| `{{#example_in Examples/Classes.lean powDesugar}}` | `{{#example_out Examples/Classes.lean powDesugar}}` | `HPow` |
| `{{#example_in Examples/Classes.lean negDesugar}}` | `{{#example_out Examples/Classes.lean negDesugar}}` | `Neg` |


## ビット単位の演算子

Leanには標準のビット単位の演算子がいくつかあり、型クラスを使ってオーバーロードされています。
`{{#example_in Examples/Classes.lean UInt8}}`、 `{{#example_in Examples/Classes.lean UInt16}}`、 `{{#example_in Examples/Classes.lean UInt32}}`、 `{{#example_in Examples/Classes.lean UInt64}}`、そして `{{#example_in Examples/Classes.lean USize}}` などの固定幅の型に対するインスタンスがあります。
後者は現在のプラットフォーム上のワードサイズで、通常は32ビットまたは64ビットです。
以下のビット単位の演算子がオーバーロードされています：

| 演算式 | 展開 | クラス名 |
|--------|------|----------|
| `{{#example_in Examples/Classes.lean bAndDesugar}}` | `{{#example_out Examples/Classes.lean bAndDesugar}}` | `HAnd` |
| <code class="hljs">x ||| y </code> | `{{#example_out Examples/Classes.lean bOrDesugar}}` | `HOr` |
| `{{#example_in Examples/Classes.lean bXorDesugar}}` | `{{#example_out Examples/Classes.lean bXorDesugar}}` | `HXor` |
| `{{#example_in Examples/Classes.lean complementDesugar}}` | `{{#example_out Examples/Classes.lean complementDesugar}}` | `Complement` |
| `{{#example_in Examples/Classes.lean shrDesugar}}` | `{{#example_out Examples/Classes.lean shrDesugar}}` | `HShiftRight` |
| `{{#example_in Examples/Classes.lean shlDesugar}}` | `{{#example_out Examples/Classes.lean shlDesugar}}` | `HShiftLeft` |

`And` や `Or` という名前は論理接続詞の名前として既に使用されているため、`HAnd` や `HOr` の同種のバージョンは `And` や `Or` ではなく `AndOp` や `OrOp` と呼ばれます。

## 等価性と順序評価

2つの値の等価性をテストするには通常 `BEq` クラスが使用されます。これは「Boolean equality（ブール等価性）」の略です。
Leanが定理証明者としての使用を考慮すると、Leanには実際には2種類の等価演算子があります：
 * _ブール等価性_ は他のプログラミング言語で見られる等価性と同じです。これは2つの値を取って `Bool` を返す関数です。ブール等価性はPythonやC#と同様に二重等号で記述されます。Leanは純粋な関数言語なので、参照と値の等価性は別々にはならず、ポインタは直接観測されることはありません。
 * _命題の等価性_ は、二つのものが等しいという数学的な声明です。命題の等価性は関数ではなく、証明を認める数学的な声明です。それは一体の等号で書かれています。命題の等価性はこの等価性の証拠を分類する型のようなものです。

両方の等価性の概念は重要であり、異なる目的で使用されます。
プログラム内では、二つの値が等しいかどうかを判断する必要がある場合にブール等価性が役立ちます。
たとえば、`{{#example_in Examples/Classes.lean boolEqTrue}}` は `{{#example_out Examples/Classes.lean boolEqTrue}}` と評価され、`{{#example_in Examples/Classes.lean boolEqFalse}}` は `{{#example_out Examples/Classes.lean boolEqFalse}}` と評価されます。
関数など一部の値は、等価性をチェックできません。
たとえば、`{{#example_in Examples/Classes.lean functionEq}}` は以下のエラーを発生させます：
```output error
{{#example_out Examples/Classes.lean functionEq}}
```
このメッセージが示すように、`==` は型クラスを使用してオーバーロードされています。
式 `{{#example_in Examples/Classes.lean beqDesugar}}` は実際に `{{#example_out Examples/Classes.lean beqDesugar}}` の略です。

命題の等価性はプログラムの呼び出しではなく、数学的な声明です。
命題はある声明の証拠を記述する型のようなものなので、命題の等価性は `String` や `Nat → List Int` のような型とブール等価性よりも共通点が多いです。
これは自動的にチェックすることは不可能ということを意味します。
しかし、任意のふたつの式の等価性は、それらが同じ型を持っていればLeanで宣言することができます。
命題 `{{#example_in Examples/Classes.lean functionEqProp}}` は十全に妥当な声明です。
数学の視点からすると、ふたつの関数が等しいのは、等しい入力を等しい出力に写像する場合ですので、この声明は事実に基づいていますが、Leanを説得するためには二行の証明が必要です。

一般的に、Leanをプログラミング言語として使用する場合、命題よりむしろブール関数を使用する方が簡単です。
しかし、`Bool` のコンストラクターで `true` と `false` という名前が示すように、この違いは時々あいまいになります。
いくつかの命題は _可決定_ です。つまり、それらはブール関数のようにチェックすることができます。
命題が真または偽であるかどうかをチェックする関数は _決定プロセデュア_ と呼ばれ、その命題の真実性または虚偽性の _証拠_ を返します。
可決定の命題の例には、自然数の等価性・不等価性、文字列の等価性、それ自体が可決定である命題の "ands" と "ors" が含まれます。

Leanでは、`if` は可決定の命題と共に動作します。
たとえば、`2 < 4` は命題です：
```lean
{{#example_in Examples/Classes.lean twoLessFour}}
```
```output info
{{#example_out Examples/Classes.lean twoLessFour}}
```
それにもかかわらず、`if` の条件としてそれを記述するのは完全に許容されます。
たとえば、`{{#example_in Examples/Classes.lean ifProp}}` は `Nat` 型を持ち、`{{#example_out Examples/Classes.lean ifProp}}` と評価されます。


すべての命題が可決定であるわけではありません。
もしそうであれば、コンピュータは真の命題を決定プロセデュアを実行することによって証明できるでしょう、そして数学者は仕事を失うでしょう。
より具体的に言うと、可決定の命題は決定プロセデュアであるメソッドを持つ `Decidable` 型クラスのインスタンスを持っています。
`Bool` であるかのように可決定でない命題を使用しようとすると、`Decidable` インスタンスを見つけることができない結果となります。
例えば、`{{#example_in Examples/Classes.lean funEqDec}}` は以下を引き起こします：
```output error
{{#example_out Examples/Classes.lean funEqDec}}
```

以下の通常は可決定の命題は型クラスを使ってオーバーロードされています：

| 演算式 | 展開 | クラス名 |
|--------|------|----------|
| `{{#example_in Examples/Classes.lean ltDesugar}}` | `{{#example_out Examples/Classes.lean ltDesugar}}` | `LT` |
| `{{#example_in Examples/Classes.lean leDesugar}}` | `{{#example_out Examples/Classes.lean leDesugar}}` | `LE` |
| `{{#example_in Examples/Classes.lean gtDesugar}}` | `{{#example_out Examples/Classes.lean gtDesugar}}` | `LT` |
| `{{#example_in Examples/Classes.lean geDesugar}}` | `{{#example_out Examples/Classes.lean geDesugar}}` | `LE` |

`LT` や `LE`の新しいインスタンスを定義する方法がまだ示されていないため、新しいインスタンスを定義するのが困難かもしれません。

さらに、`<`、`==`、`>` を使って値を比較することは非効率的です。
他の値よりも一つの値が小さいかどうかをチェックし、それから等しいかどうかをチェックすることは、大きなデータ構造にまたがって二度の走査を必要とすることもあります。
この問題を解決するために、JavaとC#では、一度に3つの操作すべてを実装するためにクラスによってオーバーライド可能な標準の `compareTo` および `CompareTo` メソッドがあります。
これらのメソッドは、受信側が引数よりも小さい場合には負の整数を、等しい場合にはゼロを、受信側が引数よりも大きい場合には正の整数を返します。
整数の意味をオーバーロードするのではなく、Leanにはこれら3つの可能性を記述する組み込みの帰納型があります：
```lean
{{#example_decl Examples/Classes.lean Ordering}}
```
`Ord` 型クラスはこれらの比較のためにオーバーロードすることができます。
`Pos`に対する実装は以下のようになります：
```lean
{{#example_decl Examples/Classes.lean OrdPos}}
```
Javaで `compareTo` が正しいアプローチであるような状況では、Leanで `Ord.compare` を使用します。

## ハッシュ化

JavaとC#にはそれぞれ `hashCode` と `GetHashCode` メソッドがあり、これはハッシュテーブルなどのデータ構造で使用される値のハッシュを計算します。
Leanの対応する機能は `Hashable` と呼ばれる型クラスです：
```lean
{{#example_decl Examples/Classes.lean Hashable}}
```
2つの値がそのタイプのための `BEq` インスタンスに従って等しいと考えられる場合、それらは同じハッシュを持つべきです。
言い換えれば、`x == y` であれば、`hash x == hash y` であるべきです。
`x ≠ y` の場合、`hash x` が `hash y` と異なる必要は必ずしもありません（結局のところ、`Nat` の値は `UInt64` の値よりも無限に多いですが）、しかし異なる値が異なるハッシュを持つ可能性が高い場合、ハッシュに基づくデータ構造のパフォーマンスは向上します。
これは Java と C# と同じ期待です。

標準ライブラリには構成子の異なるフィールドのためにハッシュを組み合わせるために使用できる `{{#example_in Examples/Classes.lean mixHash}}` という関数が含まれています。
帰納データ型のための合理的なハッシュ関数は、各構成子にユニークな番号を割り当て、その番号を各フィールドのハッシュと混ぜることによって書かれます。
たとえば、`Pos` の`Hashable` インスタンスは以下のように書くことができます：

```lean
{{#example_decl Examples/Classes.lean HashablePos}}
```

多相型の `Hashable` インスタンスは再帰的インスタンス検索を使用しています。
`α` をハッシュ可能なときに限り、`NonEmptyList α` をハッシュ化することが可能です：

```lean
{{#example_decl Examples/Classes.lean HashableNonEmptyList}}
```

二分木は `BEq` と `Hashable` の実装において再帰と再帰的インスタンス検索の両方を使用します：

```lean
{{#example_decl Examples/Classes.lean TreeHash}}
```

## 標準クラスの導出

`BEq` や `Hashable` のようなクラスのインスタンスは、手作業で実装するにはしばしば面倒です。
Leanには _インスタンス導出_ と呼ばれる機能があり、多くの型クラスのためにコンパイラが自動的に適切な動作をするインスタンスを構築することを可能にします。
実際に、「構造に関するセクション」(../getting-to-know/structures.md) で `Point` の定義における `deriving Repr` のフレーズはインスタンス導出の一例です。

インスタンスは２つの方法で導出できます。
最初の方法は構造体や帰納型を定義する際に使用できます。
この場合、型宣言の最後に `deriving` を追加し、導出されるべきクラスの名前を続けます。
既に定義された型の場合、スタンドアロンの `deriving` コマンドを使用できます。
`deriving instance C1, C2, ... for T` と書くことで、事後に型 `T` に対して `C1, C2, ...` のインスタンスを導出します。

`BEq` および `Hashable` のインスタンスは、`Pos` と `NonEmptyList` のために非常に少ないコード量で導出することができます：

```lean
{{#example_decl Examples/Classes.lean BEqHashableDerive}}
```

少なくとも以下のクラスについてはインスタンスを導出できます：
 * `Inhabited`
 * `BEq`
 * `Repr`
 * `Hashable`
 * `Ord`

しかし、場合によっては導出された `Ord` インスタンスがアプリケーションで求められる正確な順序を生成しないことがあります。
そのような場合は、手作業で `Ord` インスタンスを書くことが適切です。
Leanの上級ユーザーによってインスタンスを導出できるクラスの集合を拡張することが可能です。

プログラマの生産性やコードの可読性に明白な利点がある一方で、インスタンスを導出することはコードの保守管理を容易にします。なぜなら、型の定義が進化するにつれてインスタンスが更新されるからです。
データ型の更新に関する変更セットは、等式テストとハッシュ計算への行ごとの決まりきった変更がなければ、読むのが簡単です。