# 正の数値

あるアプリケーションでは、正の数値のみが意味をなすことがあります。
例えば、コンパイラーやインタプリターはソースコードの位置を示すために1から始まる行番号や列番号を使い、非空リストを表すデータ型では長さがゼロと報告されることはありません。
自然数を用いて、数がゼロでないことを確認するアサーションをコードに散りばめるのではなく、正の数のみを表すデータ型を設計することが有効な場合があります。

正の数値を表す一つの方法は`Nat`に似ていますが、基底ケースが`zero`の代わりに`one`です：
```lean
{{#example_decl Examples/Classes.lean Pos}}
```
このデータ型は正確に意図した値の集合を表していますが、使用するのは非常に不便です。
例えば、数値リテラルは拒否されます：
```lean
{{#example_in Examples/Classes.lean sevenOops}}
```
```output error
{{#example_out Examples/Classes.lean sevenOops}}
```
代わりに、コンストラクタを直接使用する必要があります：
```lean
{{#example_decl Examples/Classes.lean seven}}
```

同様に、加算や乗算を使用するのも容易ではありません：
```lean
{{#example_in Examples/Classes.lean fourteenOops}}
```
```output error
{{#example_out Examples/Classes.lean fourteenOops}}
```
```lean
{{#example_in Examples/Classes.lean fortyNineOops}}
```
```output error
{{#example_out Examples/Classes.lean fortyNineOops}}
```

これらのエラーメッセージはそれぞれ「failed to synthesize instance」で始まります。
これは重複した操作が実装されていないためのエラーであり、実装する必要がある型クラスを説明しています。

## クラスとインスタンス

型クラスは、名前といくつかのパラメーター、そしてメソッドのコレクションで構成されます。
パラメーターはオーバーロード可能な操作が定義されている型を記述し、メソッドはオーバーロード可能な操作の名前と型シグネチャーです。
再び、オブジェクト指向言語との用語の衝突があります。
オブジェクト指向プログラミングでは、メソッドは特定のオブジェクトに接続された関数であり、オブジェクトのプライベート状態への特別なアクセスを持っています。
オブジェクトはそのメソッドを介して操作されます。
Leanでは、「メソッド」という用語はオーバーロード可能であると宣言された操作を指しますが、オブジェクトや値やプライベートフィールドと特別な関連はありません。

加算をオーバーロードする一つの方法は、`Plus`という名前の型クラスを定義し、`plus`という加算メソッドを持つことです。
`Nat`に対する`Plus`のインスタンスが定義されると、`Plus.plus`を使用して2つの`Nat`を加算することが可能になります：
```lean
{{#example_in Examples/Classes.lean plusNatFiveThree}}
```
```output info
{{#example_out Examples/Classes.lean plusNatFiveThree}}
```
引数の型を増やすことで`Plus.plus`をより多くの種類の引数に対応させます。

次の型クラス宣言では、`Plus`がクラスの名前であり、`α : Type`が唯一の引数であり、`plus : α → α → α`が唯一のメソッドです：
```lean
{{#example_decl Examples/Classes.lean Plus}}
```
この宣言により、型`α`に対して`Plus`という型クラスがオーバーロード操作を提供しているとされます。
特に、`plus`というオーバーロード操作がありますが、これは2つの`α`を取り、`α`を返すものです。

型クラスは第一級の市民です。型も第一級の市民です。
特に、型クラスは別の種類の型です。
`{{#example_in Examples/Classes.lean PlusType}}`の型は`{{#example_out Examples/Classes.lean PlusType}}`です。なぜなら、これは型の引数（`α`）を取り、`Plus`の操作が`α`でオーバーロードされることを記述した新しい型を結果とします。

特定の型の`plus`をオーバーロードするためにインスタンスを書きます：
```lean
{{#example_decl Examples/Classes.lean PlusNat}}
```
`instance`の後のコロンは、`Plus Nat`が確かに型であることを示しています。
クラス`Plus`の各メソッドは`:=`を使用して値を割り当てる必要があります。
この場合、メソッドは`plus`のみです。

デフォルトでは、型クラスメソッドは型クラスと同じ名前の名前空間で定義されます。
ユーザーが最初にクラス名を入力する必要がないように、名前空間を`open`することが便利な場合があります。
`open`コマンドの括弧は、名前空間からアクセス可能にする指定された名前のみを示しています：
```lean
{{#example_decl Examples/Classes.lean openPlus}}

{{#example_in Examples/Classes.lean plusNatFiveThreeAgain}}
```
```output info
{{#example_out Examples/Classes.lean plusNatFiveThreeAgain}}
```

`Pos`の加算関数と`Pos`の`Plus`のインスタンスを定義すると、`plus`を使用して`Pos`と`Nat`の値の両方を加算することができるようになります：
```lean
{{#example_decl Examples/Classes.lean PlusPos}}
```

まだ`Plus Float`のインスタンスがないため、`plus`を使用して2つの浮動小数点数を加算しようとすると、おなじみのメッセージで失敗します：
```lean
{{#example_in Examples/Classes.lean plusFloatFail}}
```
```output error
{{#example_out Examples/Classes.lean plusFloatFail}}
```
これらのエラーは、Leanが特定の型クラスのインスタンスを見つけることができなかったことを意味します。

## オーバーロードされた加算

Leanの組み込みの加算演算子は、`HAdd`という型クラスのための構文糖衣であり、その型クラスは加算の引数が異なる型であっても柔軟に対応することができます。
`HAdd`は _異種加算_ を略したものです。
例えば、`Nat`を`Float`に加算することができる`HAdd`のインスタンスを書くことができ、これにより新しい`Float`が生まれます。
プログラマーが`{{#example_eval Examples/Classes.lean plusDesugar 0}}`と書いた場合、それは`{{#example_eval Examples/Classes.lean plusDesugar 1}}`と解釈されます。

`HAdd`の完全な一般性を理解するには[この章の別のセクション](out-params.md)で議論される機能が必要ですが、引数の型を混在させないよりシンプルな型クラス`Add`があります。
Leanのライブラリは、両引数の型が同じである`HAdd`のインスタンスを探す際に、`Add`のインスタンスが見つかるようにセットアップされています。

`Add Pos`のインスタンスを定義することで、`Pos`の値で通常の加算構文を使用できるようになります：
```lean
{{#example_decl Examples/Classes.lean AddPos}}

{{#example_decl Examples/Classes.lean betterFourteen}}
```

## 文字列への変換

別の役立つ組み込みクラスは`ToString`と呼ばれています。
`ToString`のインスタンスは、与えられた型の値を文字列に変換する標準的な方法を提供します。
例えば、`ToString`インスタンスは、値が補間文字列に出現した場合に使用され、[説明の`IO`](../hello-world/running-a-program.html#running-a-program)の最初で使用された`IO.println`関数が値をどのように表示するかを決定します。

例えば、`Pos`を`String`に変換する一つの方法は、その内部構造を明らかにすることです。
関数`posToString`は、`Pos.succ`の使用を括弧で囲むかどうかを決定する`Bool`を取りますが、これは初回の関数呼び出しでは`true`に、すべての再帰的呼び出しでは`false`にすべきです。
```lean
{{#example_decl Examples/Classes.lean posToStringStructure}}
```
この関数を`ToString`インスタンスに使用すると：
```lean
{{#example_decl Examples/Classes.lean UglyToStringPos}}
```
その結果、情報豊富ながらも圧倒的な出力になります：
```lean
{{#example_in Examples/Classes.lean sevenLong}}
```
```output info
{{#example_out Examples/Classes.lean sevenLong}}
```

一方で、すべての正の数には対応する`Nat`があります。
それを`Nat`に変換してから`ToString Nat`インスタンス（つまり、`Nat`に対する`toString`のオーバーロード）を使用することは、はるかに短い出力をすばやく生成する手段です：
```lean
{{#example_decl Examples/Classes.lean posToNat}}

{{#example_decl Examples/Classes.lean PosToStringNat}}

{{#example_in Examples/Classes.lean sevenShort}}
```
```output info
{{#example_out Examples/Classes.lean sevenShort}}
```
インスタンスが複数定義されている場合、最も新しいものが優先されます。
さらに、型に`ToString`インスタンスがある場合、型が`deriving Repr`で定義されていなかったとしても、`#eval`の結果を表示するために使用できるため、`{{#example_in Examples/Classes.lean sevenEvalStr}}`の出力は`{{#example_out Examples/Classes.lean sevenEvalStr}}`になります。

## オーバーロードされた乗算

乗算については、`HAdd`のように引数の型が混合されることを許容する型クラス`HMul`があります。
`{{#example_eval Examples/Classes.lean plusDesugar 0}}`が`{{#example_eval Examples/Classes.lean plusDesugar 1}}`と解釈されるのと同様に、`{{#example_eval Examples/Classes.lean timesDesugar 0}}`は`{{#example_eval Examples/Classes.lean timesDesugar 1}}`と解釈されます。
引数の型が同じである乗算の一般的なケースでは、`Mul`インスタンスが十分です。

`Pos`に対して`Mul`のインスタンスを持つことは、通常の乗算構文を`Pos`で使用するためにあります：
```lean
{{#example_decl Examples/Classes.lean PosMul}}
```
このインスタンスを持つことで、乗算が予想通りに機能します：
```lean
{{#example_in Examples/Classes.lean muls}}
```
```output info
{{#example_out Examples/Classes.lean muls}}
```

## リテラル数字

正の数値のコンストラクタの列を書き出すことはかなり不便です。
この問題を解決する方法の一つは、`Nat`を`Pos`に変換する関数を提供することです。
ただし、このアプローチには欠点があります。
まず、`Pos`は`0`を表すことができないため、結果の関数は`Nat`を大きな数値に変換するか、あるいは`Option Pos`を返すことになります。
どちらもユーザーにとって特に便利ではありません。
第二に、関数を明示的に呼び出す必要があるため、正の数値を使用するプログラムを`Nat`を使用するプログラムよりも書くことがはるかに不便になります。
正確な型と便利なAPIの間にトレードオフがあると、正確な型はあまり有用ではなくなります。

Leanでは、自然数のリテラルは`OfNat`という型クラスを使用して解釈されます：
```lean
{{#example_decl Examples/Classes.lean OfNat}}
```
この型クラスには2つの引数があります：`α`は自然数がオーバーロードされている型であり、名前のない`Nat`引数はプログラムで遭遇した実際のリテラル数値です。
その後、メソッド`ofNat`は数値リテラルの値として使用されます。
クラスに`Nat`引数が含まれているため、数値が意味をなす値にのみインスタンスを定義することが可能になります。

`OfNat`は、型クラスの引数が型である必要はないことを示しています。
Leanの型は言語の中で第一級の参加者であり、関数への引数として渡したり、`def`や`abbrev`で定義を与えたりすることができるので、より柔軟性の低い言語で許可できない位置で非型引数を防ぐための障壁はありません。
この柔軟性により、特定の型だけでなく特定の値に対するオーバーロード操作を提供することが可能になります。

例えば、4未満の自然数を表現する和型は次のように定義できます：
```lean
{{#example_decl Examples/Classes.lean LT4}}
```
任意のリテラル数値をこの型に対して使うことは意味がありませんが、4未満の数なら明らかに理にかなっています：
```lean
{{#example_decl Examples/Classes.lean LT4ofNat}}
```
これらのインスタンスを使用すれば、下記の例が機能します：
```lean
{{#example_in Examples/Classes.lean LT4three}}
```
```output info
{{#example_out Examples/Classes.lean LT4three}}
```
```lean
{{#example_in Examples/Classes.lean LT4zero}}
```
```output info
{{#example_out Examples/Classes.lean LT4zero}}
```
しかし、範囲外のリテラルはまだ許可されません：
```lean
{{#example_in Examples/Classes.lean LT4four}}
```
```output error
{{#example_out Examples/Classes.lean LT4four}}
```

`Pos`に対しては、`OfNat`インスタンスは`Nat.zero`を除く任意の`Nat`で機能するべきです。
別の言い方をすれば、自然数`n`に対して、インスタンスは`n + 1`で動作するべきです。
名前のように`α`が自動的に関数の暗黙の引数となり、Leanが自動的に埋めてくれるように、インスタンスも自動的な暗黙の引数を受け取ることができます。
このインスタンスでは、引数`n`はどんな`Nat`も表し、インスタンスはそれより1大きい`Nat`に対して定義されています：
```lean
{{#example_decl Examples/Classes.lean OfNatPos}}
```
ユーザーが書いた数字より1少ない`Nat`を表しているので、ヘルパー関数`natPlusOne`はその引数より1大きい`Pos`を返します。
これにより自然数リテラルを正の数に使用できるようになりますが、ゼロには使用できません：
```lean
{{#example_decl Examples/Classes.lean eight}}

{{#example_in Examples/Classes.lean zeroBad}}
```
```output error
{{#example_out Examples/Classes.lean zeroBad}}
```

## 練習問題

### 別の表現法

正数をある`Nat`の後継として表現するという代替方法があります。
`Pos`の定義を置き換えて、`Nat`を含む名前`succ`のコンストラクタを持つ構造体にします：
```lean
{{#example_decl Examples/Classes.lean AltPos}}
```
このバージョンの`Pos`が便利に使えるように`Add`、`Mul`、`ToString`、および`OfNat`のインスタンスを定義してください。

### 偶数

偶数のみを表すデータ型を定義してください。それを便利に使えるように`Add`、`Mul`、および`ToString`のインスタンスを定義してください。
`OfNat`は[次のセクション](polymorphism.md)で紹介される機能を必要とします。

### HTTPリクエスト

HTTPリクエストは、`GET`や`POST`などのHTTPメソッドの識別から始まり、URIとHTTPバージョンを添えています。
HTTPメソッドの面白いサブセットを表す帰納的型と、HTTPレスポンスを表す構造体を定義してください。
レスポンスはデバッグ可能にするため`ToString`インスタンスを持っているべきです。
それぞれのHTTPメソッドに異なる`IO`アクションを関連付けるための型クラスを使用し、各メソッドを呼び出して結果を印刷するテストハーネスとしての`IO`アクションを記述してください。