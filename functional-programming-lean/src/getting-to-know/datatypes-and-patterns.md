# データ型とパターン

構造体は、複数の独立したデータピースを組み合わせて、新しい型によって代表される一貫性のある全体を形成することを可能にします。
値のコレクションをまとめてグループ化するような構造体のような型は、_積型(product types)_ と呼ばれます。
しかし、多くのドメイン概念は構造体として自然に表現することができません。
たとえば、アプリケーションがユーザー権限を追跡する必要があり、ユーザーにはドキュメントの所有者、ドキュメントの編集者、そしてドキュメントの読み取りのみが可能な者といった区別があります。
計算機は、加算、減算、乗算などの複数の二項演算子を持っています。
構造体では、複数の選択肢をエンコードする簡単な方法はありません。

同様に、構造体は固定のフィールドセットを追跡する優れた方法ですが、多くのアプリケーションでは任意の数の要素を含むデータが必要です。
木やリストのような最も古典的なデータ構造の多くは再帰的な構造を持っており、リストの尾はそれ自体がリストであり、二分木の左右の枝がそれ自体が二分木です。
先述の計算機では、式自体の構造が再帰的です。
加算式の加数がそれ自体が乗算式であることがあります。

選択肢を許すデータ型は、_和型（sum types）_ と呼ばれ、自分自身のインスタンスを含むことができるデータ型は _再帰データ型(recursive datatypes)_ と呼ばれます。
再帰型の和型は _帰納データ型（inductive datatypes）_ と呼ばれ、それに関するステートメントを証明するために数学的帰納法を使用することができます。
プログラミングするときは、帰納データ型はパターンマッチングと再帰関数を通じて処理されます。

標準ライブラリの組み込み型の多くは実際には帰納データ型です。
例えば、`Bool`は帰納データ型です：
```lean
{{#example_decl Examples/Intro.lean Bool}}
```
この定義には2つの主要な部分があります。
最初の行は新しい型の名前を提供し（`Bool`）、残りの行はそれぞれコンストラクタを記述しています。
構造体のコンストラクタと同じように、帰納データ型のコンストラクタも他のデータを受け取り、保持するだけの単なる受動的な場所であり、任意の初期化や検証コードを挿入する場所ではありません。
構造体とは異なり、帰納データ型には複数のコンストラクタがあります。
ここでは、`true`と`false`の2つのコンストラクタがあり、どちらも引数を取りません。
構造体宣言がその名前を宣言した型の後に名前付けられた名前空間に配置するように、帰納データ型はそのコンストラクタの名前を名前空間に配置します。
Lean標準ライブラリでは、`true`と`false`はこの名前空間から再エクスポートされるため、`Bool.true`と`Bool.false`としてではなく、単独で書くことができます。

データモデリングの観点から、帰納データ型は、他の言語ではシールされた抽象クラスが使用される多くの同じ文脈で使用されます。
C#やJavaなどの言語では、`Bool`の類似の定義を次のように書くかもしれません：
```C#
abstract class Bool {}
class True : Bool {}
class False : Bool {}
```
ただし、これらの表現の詳細はかなり異なります。特に、非抽象クラスは新しいタイプとデータの新しい割り当て方法の両方を作ります。オブジェクト指向の例では、`True`と`False`は`Bool`よりも具体的な型でありますが、Leanの定義では新しい型`Bool`のみが導入されます。

非負整数の型`Nat`は帰納データ型です：
```lean
{{#example_decl Examples/Intro.lean Nat}}
```
ここでは、`zero`は0を表し、`succ`は他のいくつかの数の後継を表します。
`succ`の宣言で言及されている`Nat`は、ちょうど定義中にある同じ型`Nat`です。
_後継者_ とは「それよりも1大きい」という意味であり、5の後継者は6、32,185の後継者は32,186です。
この定義を使うと、`{{#example_eval Examples/Intro.lean four 1}}`は`{{#example_eval Examples/Intro.lean four 0}}`として表されます。
この定義は`Bool`の定義に似ていますが、名前が少し違います。
唯一の本当の違いは、`succ`の後に`(n : Nat)`が続き、これはコンストラクタ`succ`が型`Nat`の引数を取り、その引数は偶然`n`という名前であることを指定します。
名前`zero`と`succ`はそれらの型にちなんで名付けられた名前空間にあり、それゆえにそれぞれ`Nat.zero`と`Nat.succ`として参照されなければなりません。

引数名、例えば`n`は、Leanのエラーメッセージや数学的証明を書く際に提供されるフィードバックに表示されます。
Leanには名前によって引数を提供するオプショナルな構文もあります。
一般的には、引数名の選択は構造体のフィールド名の選択ほど重要ではなく、APIの大部分を形成しないためです。

C#やJavaでは、`Nat`は次のように定義できます：
```C#
abstract class Nat {}
class Zero : Nat {}
class Succ : Nat {
  public Nat n;
  public Succ(Nat pred) {
	n = pred;
  }
}
```
上で述べた`Bool`の例と同様に、これはLeanの同等物よりも多くの型を定義します。
さらに、この例は、Leanのデータ型コンストラクタがC#やJavaのコンストラクタとは異なり、抽象クラスのサブクラスに非常に似ていることを強調しています。というのも、ここで示されているコンストラクタには実行される初期化コードが含まれているからです。

和型は、TypeScriptで識別された共用体を文字列タグを使用してエンコードする方法にも似ています。TypeScriptでは、`Nat` は以下のように定義される可能性があります。

```typescript
interface Zero {
    tag: "zero";
}

interface Succ {
    tag: "succ";
    predecessor: Nat;
}

type Nat = Zero | Succ;
```

C#やJavaと同様に、このエンコーディングではLeanよりも多くの型が生じます。というのも、`Zero` と `Succ` はそれぞれ独自の型を形成しているからです。これはまた、Leanのコンストラクターが、内容を識別するタグを含むJavaScriptまたはTypeScriptのオブジェクトに対応していることを示しています。

## パターンマッチング

多くの言語では、これらの種類のデータは、最初にインスタンスオブ演算子を使用してどのサブクラスを受け取ったかをチェックし、その後、与えられたサブクラスで使用可能なフィールドの値を読み取ることによって消費されます。
インスタンスオブチェックはどのコードを実行するかを決定し、このコードに必要なデータが利用可能であることを保証し、フィールド自体はデータを提供します。
Leanでは、これらの両方の目的は _パターンマッチング_ によって同時に提供されます。

パターンマッチングを使用する関数の例は、その引数が`Nat.zero`の場合に`true`を返す`isZero`関数です。
```lean
{{#example_decl Examples/Intro.lean isZero}}
```
`match`式は関数の引数`n`を分解するために与えられます。
もし`n`が`Nat.zero`によって構築された場合、パターンマッチの最初の分岐が取られ、結果は`true`です。
もし`n`が`Nat.succ`によって構築された場合、2番目の分岐が取られ、結果は`false`です。

例えば、`{{#example_eval Examples/Intro.lean isZeroZeroSteps 0}}`の評価は次のように進行します：
```lean
{{#example_eval Examples/Intro.lean isZeroZeroSteps}}
```

`{{#example_eval Examples/Intro.lean isZeroFiveSteps 0}}`の評価も同様に進行します：
```lean
{{#example_eval Examples/Intro.lean isZeroFiveSteps}}
```

`isZero`のパターンの2番目の分岐における`k`は装飾的なものではありません。
これにより、`succ`への引数の`Nat`が見えるようになり、提供された名前を使って指定できます。
そのより小さな数を使用して、式の最終結果を計算することができます。

例えば、ある数 \\( n \\) の後継者が \\( n \\) よりも1大きい（\\( n + 1\\)）のと同じように、数の前者はそれよりも1小さいです。
もし`pred`が`Nat`の前者を見つける関数であるならば、以下の例が期待される結果を見つけることになるでしょう：
```lean
{{#example_in Examples/Intro.lean predFive}}
```
```output info
{{#example_out Examples/Intro.lean predFive}}
```
```lean
{{#example_in Examples/Intro.lean predBig}}
```
```output info
{{#example_out Examples/Intro.lean predBig}}
```
`Nat`は負の数を表すことができないため、`0`は少し問題です。
通常、`Nat`を使用する際には、通常負の数を生じさせるような演算子は、`0`自体を生成するように再定義されます：
```lean
{{#example_in Examples/Intro.lean predZero}}
```
```output info
{{#example_out Examples/Intro.lean predZero}}
```

`Nat`の前者を見つけるためには、最初にそれを作成するためにどのコンストラクタが使用されたかをチェックする必要があります。
もし`Nat.zero`であれば、結果は`Nat.zero`です。
もし`Nat.succ`であれば、`k`という名前を使用してその下の`Nat`を参照します。
そして、この`Nat`が望ましい前者なので、`Nat.succ`の分岐の結果は`k`です。
```lean
{{#example_decl Examples/Intro.lean pred}}
```
この関数を`5`に適用すると、以下のステップが得られます：
```lean
{{#example_eval Examples/Intro.lean predFiveSteps}}
```

構造体と和型の両方でパターンマッチングが使用されます。
例えば、`Point3D`から3番目の次元を抽出する関数は次のように書かれます：
```lean
{{#example_decl Examples/Intro.lean depth}}
```
この場合、単に`z`アクセサーを使う方がはるかに簡単でしたが、構造パターンは時々関数を書く一番簡単な方法です。

## 再帰関数

定義されている名前を参照する定義は _再帰定義_ と呼ばれます。
帰納データ型は再帰的であることが許され、実際に`Nat`は`succ`が別の`Nat`を要求するためそのようなデータ型の例です。
再帰データ型は技術的な要因（利用可能なメモリなど）によってのみ制限される任意に大きなデータを表すことができます。
データ型定義内で各自然数のための構造体を書き留めることは不可能であるように、また、各可能性のためのパターンマッチケースを書き留めることも不可能です。

再帰データ型は再帰関数によってうまく補完されます。
`Nat`上の単純な再帰関数はその引数が偶数かどうかをチェックします。
この場合、`zero`は偶数です。
このような非再帰ブランチのコードは _ベースケース_ と呼ばれます。
奇数の後継者は偶数であり、偶数の後継者は奇数です。
これは、`succ`で構築された数が、その引数が偶数でない場合にのみ偶数であることを意味します。
```lean
{{#example_decl Examples/Intro.lean even}}
```

これは、`Nat`上の再帰関数を書くための典型的な思考パターンです。
最初に`zero`に対して何をするかを特定します。
次に、任意の`Nat`に対する結果をその後継者に対する結果に変換する方法を決定し、この変換を再帰呼び出しの結果に適用します。
このパターンは _構造的再帰_ と呼ばれます。

多くの言語とは異なり、Leanはデフォルトで、すべての再帰関数が最終的にベースケースに到達することを保証します。
プログラミングの観点から、これにより偶発的な無限ループが除外されます。
しかし、特に定理を証明する際には、この機能は特に重要であり、無限ループは大きな困難を引き起こします。
その結果として、Leanは元の数に対して自身を再帰的に呼び出そうとする`even`のバージョンを受け付けません。

```lean
{{#example_in Examples/Intro.lean evenLoops}}
```
エラーメッセージの重要な部分は、Leanが再帰関数が常にベースケースに到達することを判定できなかったということです（なぜなら、実際には到達しないからです）。
```output error
{{#example_out Examples/Intro.lean evenLoops}}
```

足し算は二つの引数を取りますが、調べる必要があるのはそのうちの一つだけです。数\\(n\\)にゼロを足すときは、単に\\(n\\)を返します。\\(k\\)の後継者に\\(n\\)を足すときは、\\(k\\)に\\(n\\)を足した結果の後継者を取ります。
```lean
{{#example_decl Examples/Intro.lean plus}}
```
`plus`の定義で、`k'`という名前が選ばれているのは、それが引数`k`と関連してはいるものの、同一ではないことを示すためです。例えば、`{{#example_eval Examples/Intro.lean plusThreeTwo 0}}`の評価を追っていくと、以下のステップになります：
```lean
{{#example_eval Examples/Intro.lean plusThreeTwo}}
```

足し算とは、\\(n + k\\)が\\(n\\)に対して`Nat.succ`を\\(k\\)回適用することと考えることもできます。同様に、かけ算\\(n × k\\)は\\(n\\)を自分自身に\\(k\\)回足し、引き算\\(n - k\\)は\\(n\\)の前任者を\\(k\\)回取ることです。
```lean
{{#example_decl Examples/Intro.lean times}}

{{#example_decl Examples/Intro.lean minus}}
```

すべての関数を構造的再帰を使って簡単に書くことができるわけではありません。 足し算を`Nat.succ`の繰り返し、かけ算を足し算の繰り返し、引き算を前任者の繰り返しとする理解から、割り算を引き算の繰り返しとして実装することを思いつくかもしれません。この場合、分子が除数より小さい場合、結果はゼロです。それ以外の場合は、結果は分子から除数を引いたものを除数で割った結果の後継者になります。
```lean
{{#example_in Examples/Intro.lean div}}
```
二番目の引数が`0`でない限り、このプログラムは終了します。なぜなら常にベースケースに進むための進捗を作っているからです。しかし、それは構造的再帰ではありません。なぜなら、ゼロのための結果を見つけ、任意の`Nat`の結果をその後継者の結果に変換するというパターンに従っていないからです。 特に、関数の再帰的呼び出しが入力コンストラクタの引数に対してではなく別の関数呼び出しの結果に適用されるためです。したがって、Leanは次のメッセージを出してそれを拒否します：
```output error
{{#example_out Examples/Intro.lean div}}
```
このメッセージは、`div`が手動での終了証明を必要とすることを意味しています。このトピックは[最終章](../programs-proofs/inequalities.md#division-as-iterated-subtraction)で詳しく探究されています。