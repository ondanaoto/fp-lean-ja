以下は、「依存型を使ったプログラミングの落とし穴」と題された文書の日本語訳です。

# 依存型を使ったプログラミングの落とし穴

依存型の柔軟性により、型の言語が十分に表現力を持ち、それほど表現力のない型システムでは記述できない変化を記述できるため、より有用なプログラムが型チェッカーによって受け入れられます。
同時に、依存型が非常に細かい仕様を表現できることにより、より多くのバグを含むプログラムが型チェッカーにより拒否されます。
この力は、コストがかかります。

型を返す関数（`Row`など）の内部と、それらが生成する型との密接な結合は、より大きな難しさの一例です。関数を型に使用する際に、その関数のインターフェースと実装の区別が崩れ始めます。
通常、型シグネチャや関数の入出力動作を変えない限り、リファクタリングはすべて有効です。
関数は、より効率的なアルゴリズムやデータ構造を使用するように書き換えることができ、バグを修正し、コードの明快さを改善しても、クライアントコードを壊さずに済むのです。
ただし、関数を型に使用した場合、関数の実装の内部が型の一部となり、他のプログラムへの_インターフェース_の一部となります。

例として、`Nat`上の加算の次の二つの実装を考えてみましょう。
`Nat.plusL`は最初の引数に対して再帰的です：
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean plusL}}
```
一方、`Nat.plusR`は二番目の引数に対して再帰的です：
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean plusR}}
```
加算の両実装は基本となる数学的概念に忠実であり、同じ引数を与えられると同じ結果を返します。

しかし、これらの二つの実装は型に使用される際にはかなり異なるインターフェースを提示します。
例として、二つの`Vect`を連結する関数を考えてみましょう。
この関数は、引数の長さの合計が判るような長さの`Vect`を返すべきです。
`Vect`は本質的には型がより情報豊富な`List`なので、`List.append`と全く同じように、最初の引数にパターンマッチを使い再帰的に関数を書くのは理にかなっています。
型シグネチャと、プレースホルダーを指し示す初期のパターンマッチから始めると、次の二つのメッセージが出ます：
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendL1}}
```
最初のメッセージは`nil`のケースで、プレースホルダーを`plusL 0 k`の長さを持つ`Vect`に置き換えるべきだと述べています：
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL1}}
```
二番目のメッセージは`cons`のケースで、プレースホルダーを`plusL (n✝ + 1) k`の長さを持つ`Vect`に置き換えるべきだと述べています：
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL2}}
```
`n`の後の記号を_ダガー_と呼び、Leanが内部的に発明した名前を示すために使用されます。
内部で、最初の`Vect`に対するパターンマッチは、構成要素`cons`にインデックス`n + 1`があり、`Vect`の尾部が長さ`n`を持っているため、第一の`Nat`の値を洗練させました。
ここで、`n✝`は引数`n`より一つ小さい`Nat`です。

## 素性の等価性（Definitional Equality）

`plusL`の定義にはパターンケース`0, k => k`があります。
これは最初のプレースホルダーで使われる長さに適用されるため、プレースホルダーの型`Vect α (Nat.plusL 0 k)`を`Vect α k`と書くこともできます。
同様に、`plusL`にはパターンケース`n + 1, k => plusN n k + 1`が含まれます。
これにより、第二のプレースホルダーの型は同様に`Vect α (plusL n✝ k + 1)`と書くことができます。

舞台裏で何が起こっているのかを明らかにする最初のステップは、`Nat`の引数を明示的に書くことです。これはエラーメッセージをダガー無しにすることにもつながります。なぜなら今プログラムの中で名前が明示的に書かれるからです：
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendL3}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL3}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL4}}
```
プレースホルダーに型の簡単なバージョンを注釈すると、型エラーが発生しないため、プログラムに書かれた型はLean自身が見つけた型に等価です：
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendL5}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL5}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL6}}
```

第一のケースでは`Vect α k`を要求し、`ys`はその型です。
これは、空のリストを他のリストに追加するとその他のリストが返されるという方法と平行しています。
最初のプレースホルダーの代わりに`ys`を入れることで、埋められるべき残り一つのプレースホルダー以外のプログラムが出来上がります：
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendL7}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL7}}
```

ここで非常に重要なことが起きています。
`Nat.plusL 0 k`が期待されるコンテキストで`Vect α k`が受け取られました。
しかし、`Nat.plusL`は`abbrev`ではないので、型チェック中に実行されるかのようには見えません。
別のことが起きています。

何が起きているのかを理解する鍵は、Leanは型チェック中に`abbrev`を展開するだけではないということです。
また、二つの型が互いに等価かどうかをチェックするときに計算を行うことができるため、一方の型の式を他方の型が期待されるコンテキストで使用することができます。
この性質は_素性の等価性_と呼ばれ、それは微妙です。

確かに、書かれた内容が同一の二つの型は素性の等価性と見なされるべきです。`Nat`と`Nat`、または`List String`と`List String`は等しいと考えられるべきです。
異なるデータ型から構築された任意の二つの具体的な型は等しくありませんので、`List Nat`は`Int`と等しくありません。
また、内部名の単なるリネームによって異なる型も等しいので、`(n : Nat) → Vect String n`は`(k : Nat) → Vect String k`と同じです。
型が通常のデータを含むので、素性の等価性はまたデータが等しいときを記述する必要もあります。
同じコンストラクタの使用は等しく、`0`は`0`に等しく、`[5, 3, 1]`は`[5, 3, 1]`に等しくなります。

しかし型は単なる関数の矢印やデータ型、コンストラクタ以上のものを含みます。
型はまた_変数_や_関数_を含みます。
変数の素性の等価性は比較的単純です。つまり、各変数はそれ自体にのみ等しいので、`(n k : Nat) → Vect Int n`は`(n k : Nat) → Vect Int k`とは素性の等価性を持ちません。
一方、関数の場合はより複雑です。
数学では、二つの関数が同一の入出力動作を持っている場合に等しいと考えられますが、それをチェックする効率的なアルゴリズムは存在せず、素性の等価性のポイント全体がどちらの型も交換可能であるかをLeanがチェックすることにあります。
代わりに、Leanは二つの関数が素性の等価性を持つと考えるのは、それらが定義的に等しい本文を持つ`fun`式の両方のときです。
言い換えれば、二つの関数は同じアルゴリズムを使用し、同じヘルパーを呼び出す必要があります。
これは通常は非常に役に立たないので、関数の素性の等価性は、主に全く同じ定義された関数が二つの型で発生するときに利用されます。

関数が型で呼ばれたとき、素性の等価性のチェックは関数呼び出しを減らすことを伴うかもしれません。
型`Vect String (1 + 4)`は`Vect String (3 + 2)`と素性の等価性を持つのは、`1 + 4`が`3 + 2`に素性の等価性を持つためです。
それらが等しいかどうかをチェックするために、両方が`5`に減らされ、コンストラクタールールが5回使用されます。
関数がデータに適用された素性の等価性は、まずそれらが既に同じかどうかを見ることによってチェックされます。つまり、`["a", "b"] ++ ["c"]`を`["a", "b"] ++ ["c"]`と等しいかどうかをチェックするために減らす必要はありません。
そうでなければ、関数は呼び出され、それが値で置き換えられ、値がチェックされるです。

すべての関数の引数が具体的なデータではありません。
例えば、型はコンストラクタ`zero`と`succ`から構築されていない`Nat`を含むかもしれません。
型`(n : Nat) → Vect String n`において、変数`n`は`Nat`ですが、関数が呼び出される前にはそれがどの`Nat`なのかは知ることができません。
実際、関数は最初に`0`で呼ばれ、次に`17`で、その次は`33`で呼ばれるかもしれません。
`appendL`の定義で見られたように、型が`Nat`である変数は`plusL`のような関数に渡されるかもしれません。
確かに、型`(n : Nat) → Vect String n`は型`(n : Nat) → Vect String (Nat.plusL 0 n)`に素性の等価性を持ちます。

`n`と`Nat.plusL 0 n`が素性の等価性を持っているということは、`plusL`のパターンマッチがその_最初の_引数を検査するということです。
これは問題を引き起こします：`(n : Nat) → Vect String n`は`(n : Nat) → Vect String (Nat.plusL n 0)`と素性の等価性を持っていません。これは、加算の左右の恒等性を持っているはずですが、パターンマッチが変数に対してスタックするためです。
`n`の実際の値がわかるまで、`Nat.plusL n 0`のどのケースを選べばいいのかがわからないからです。

同じ問題がクエリ例の`Row`関数にも現れます。
型`Row (c :: cs)`はあらゆるデータ型に還元されません。なぜなら`Row`の定義は一人称リストと2つ以上のエントリを持つリストのための別々のケースを持っているからです。
言い換えれば、変数`cs`を具体的な`List`コンストラクタにマッチしなければスタックしてしまいます。
これがほぼすべての`Row`を取り扱う、または構築する関数が`Row`自体と同じ3つのケースにマッチする必要がある理由です。それをアンスタックすると、パターンマッチングやコンストラクタに使用できる具体的な型が明らかになります。

`appendL`の足りないケースには`Vect α (Nat.plusL n k + 1)`が要求されます。
インデックスの中に`+ 1`があることから、次のステップは`Vect.cons`の使用であることが示唆されます：
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendL8}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendL8}}
```
`appendL`への再帰的な呼び出しは所望の長さの`Vect`を構築できます:
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean appendL9}}
```
プログラムが完成した今、`n`と`k`に明示的なマッチングを除去することで、関数を読みやすく、また呼び出しやすくすることができます:
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean appendL}}
```

定義上の等価性を用いて型を比較することは、定義上の等価性に関わるすべてのもの、含む関数定義の内部、を依存型やインデックス付きのファミリーを使うプログラムの_インターフェース_の一部にします。関数の内部を型にさらけ出すことは、そのプログラムを使用するプログラムがもはや型検査に合格しないかもしれないというリスクをもたらす可能性があります。 特に、`plusL`が`appendL`の型に用いられていることは、`plusL`の定義を同等な`plusR`に置き換えることができないことを意味します。

## 加算におけるスタック

`plusR`を使って`append`が定義された場合はどうなるでしょうか？ 同じように、明示的な長さと各ケースでのプレースホルダーアンダースコアで始めると、以下のような有用なエラーメッセージが表示されます:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendR1}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendR1}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendR2}}
```
しかし、最初のプレースホルダーに`Vect α k`型注釈を配置しようとすると、型不一致のエラーが生じます:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendR3}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendR3}}
```
このエラーは、`plusR 0 k`と`k`が定義上等価では_ない_と指摘しています。

これは`plusR`が以下の定義を持っているためです:
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean plusR}}
```
そのパターンマッチングは_第二_引数に対して行われており、変数`k`がその位置に存在することが、計算を行えないように阻止しています。 Leanの標準ライブラリの`Nat.add`は`plusL`ではなく`plusR`に相当し、この定義を使おうとするとまさに同じ困難に直面します:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendR4}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendR4}}
```

加算が変数で_スタック_しています。 スタックを解消するには[命題等価](../type-classes/standard-classes.md#equality-and-ordering)を必要とします。

## 命題等価

命題等価は、二つの式が等しいことを数学的に主張するものです。 定義上の等価性が必要な時にLeanが自動的に検査する一方で、命題等価性の主張は明確な証明が必要です。 一度等価命題が証明されると、プログラム内で型を変更し、一方の等価性を他方に置き換えることができ、型検査をスタックから解放することができます。

定義上の等価性がこれほど制限されているのはアルゴリズムで検査できるようにするためです。 命題等価性はより豊かですが、コンピュータが一般的に二つの式が命題的に等しいかどうかを検査することはできませんが、提案された証明が実際に証明であることを検証できます。 定義上の等価性と命題等価性の間の分割は、人間と機械間の労働分担を表しています：面倒な等価性は定義上の等価性の一部として自動的にチェックされるので、命題等価性の興味深い問題を解決するために人間の頭脳を解放します。
同様に、定義上の等価性は型検査によって自動的に呼び出されるのに対し、命題等価性は明示的に訴える必要があります。

[命題、証明、およびインデックス](../props-proofs-indexing.md)で、`simp`を使っていくつかの等価性ステートメントが証明されています。これらすべての等価性ステートメントは、実際にはすでに定義上の等価性であることです。 通常、命題等価性ステートメントは、それらが定義上の等価性であるか、既に証明されている等価性に十分に近い形にまで導かれた後、`simp`のようなツールを利用して簡略化されたケースを処理することで証明されます。 `simp`タクティクは非常に強力です：舞台裏では、証明を構築するためにいくつかの高速で自動化されたツールを使用しています。 `rfl`というよりシンプルなタクティクは、命題等価性を証明するために定義上の等価性を特に使用します。 `rfl`の名前は、すべてがそれ自体と等しいという等価性の特性である_反射性_を短くしたものです。

`appendR`をスタックから解放するには、`k = Nat.plusR 0 k`の証明が必要ですが、これは`plusR`がその第二引数の変数にスタックするため、定義上の等価性ではありません。 それを計算させるには、`k`を具体的なコンストラクタにしなければなりません。 これはパターンマッチングのための仕事です。

特に、`k`が_どんな_ `Nat`でもありうるので、このタスクには`(k : Nat) → k = Nat.plusR 0 k`の証明を返すことができる関数が必要です。 初期のパターンズとプレースホルダーで始まって、以下のようなメッセージが出力されます:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean plusR_zero_left1}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean plusR_zero_left1}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean plusR_zero_left2}}
```
パターンマッチングを通じて`k`を`0`に絞り込んだ後、最初のプレースホルダーは定義上で成立しているステートメントの証拠を代表します。 `rfl`タクティクがそれを処理し、第二のプレースホルダーだけが残されます:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean plusR_zero_left3}}
```

第二のプレースホルダーは少し厄介です。 式`{{#example_in Examples/DependentTypes/Pitfalls.lean plusRStep}}`は定義上で`{{#example_out Examples/DependentTypes/Pitfalls.lean plusRStep}}`と等価です。 これは、ゴールが`k + 1 = Nat.plusR 0 k + 1`としても書くことができることを意味します:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean plusR_zero_left4}}
```
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean plusR_zero_left4}}
```

等価性ステートメントの両側の`+ 1`の下には、関数自体が返す別のインスタンスがあります。 他の言い方をすれば、`k`に対する再帰呼び出しは`k = Nat.plusR 0 k`であることを証明する証拠を返すはずです。 等価性が等価性でなかったら、それは関数引数に適用されないでしょう。 言い換えれば、もし`x = y`ならば、`f x = f y`です。 標準ライブラリには`congrArg`という関数があり、一つの関数と等価性の証明を受け取り、等価性の両側にその関数が適用された新しい証明を返します。 この場合、関数は`(· + 1)`です:
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean plusR_zero_left_done}}
```

命題等価性はプログラム内で右向きの三角演算子`▸`を使って展開されます。 最初の引数として等価性の証明を受け取り、第二の引数として他の式を受け取るこのオペレーターは、第二の引数の型の中で等価性の左側のインスタンスを右側のものと置き換えることができます。 つまり、以下の定義にはタイプエラーが含まれていません:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendRsubst}}
```
最初のプレースホルダーは予想通りの型を持っています:
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendRsubst}}
```
これで`ys`で記入できるようになります:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean appendR5}}
```

残りのプレースホルダーを埋めるには、もう一度加算のインスタンスをスタックから解放する必要があります:
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean appendR5}}
```
ここで証明しなければならないステートメントは、`Nat.plusR (n + 1) k = Nat.plusR n k + 1`であり、これを`▸`と組み合わせて式の一番上に`+ 1`を引き出すことができれば、`cons`のインデックスと一致させることができます。

証明は、`plusR`に対する第二の引数としての`k`にパターンマッチングする再帰関数です。 `plusR`自体がその第二引数にパターンマッチングしているので、パターンマッチングを通じてそれを「スタックから解放」し、計算の動作を明らかにすることができます。 この証明の骨格は`plusR_zero_left`に非常に似ています:
```lean
{{#example_in Examples/DependentTypes/Pitfalls.lean plusR_succ_left_0}}
```

残りのケースの型は定義上で`Nat.plusR (n + 1) k + 1 = Nat.plusR n (k + 1) + 1`と等しいので、`plusR_zero_left`で行われたのと同じように`congrArg`を使って解決できます:
```output error
{{#example_out Examples/DependentTypes/Pitfalls.lean plusR_succ_left_2}}
```
これにより完成した証明になります:
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean plusR_succ_left}}
```

完成した証明は`appendR`の第二のケースでスタックから解放するために使うことができます:
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean appendR}}
```
`appendR`の長さの引数を再び暗黙的にすると、証明に訴えるために明示的に名前をつけることはなくなりますが、Leanの型検査は裏で十分な情報を持っており、自動的にそれらを埋めることができます。 他の値では型がマッチしないためです:
```lean
{{#example_decl Examples/DependentTypes/Pitfalls.lean appendRImpl}}
```

## 長所と短所

インデックス付きファミリーは重要な特性を持っています：それらに対するパターンマッチングは定義上の等価性に影響を与えます。 例えば、`Vect`に対する`match`式の`nil`ケースでは、単に長さが`0`に_なります_。 定義上の等価性は非常に便利です。なぜならそれは常に活発であって、明示的に呼び出す必要がないからです。

しかし、依存型とパターンマッチングに定義上の等価性を使用することは、ソフトウェアエンジニアリングに重大な欠点を有しています。まず、関数は型で使われるために特別に書かれなければなりません。型で使うことが便利な関数は、最も効率的なアルゴリズムを使用していないかもしれません。一度関数が型を使用してさらけ出されると、その実装はインターフェースの一部となり、将来のリファクタリングに困難をもたらします。 二つ目に、定義上の等価性は遅い可能性があります。 当事者が2つの式が定義上等しいかどうかをチェックするように求めた場合、質問である関数が複雑で多くの抽象化の層を持っている場合、大量のコードを実行する必要があるかもしれません。 三つ目に、定義上の等価性の失敗によるエラーメッセージは必ずしも理解しやすいわけではありません。な