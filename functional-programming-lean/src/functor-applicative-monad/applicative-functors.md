# アプリカティブ関手

_適用ファンクター_ とは、`pure` および `seq` という2つの追加の操作を利用可能にするファンクターです。
`pure` は `Monad` で使われる同じ演算子です。なぜなら `Monad` は実際には `Applicative` から継承されているからです。
`seq` は `map` によく似ています：それは関数を使用してデータ型の内容を変換することを可能にします。
しかし、`seq` を使うときは、関数自体がデータ型の中に含まれます：`{{#example_out Examples/FunctorApplicativeMonad.lean seqType}}`。
型 `f` の下に関数を配置することで、`Applicative` のインスタンスは関数がどのように適用されるかを制御することができますが、`Functor.map` は無条件で関数を適用します。
二番目の引数は、関数が決して適用されない場合に `seq` の定義を短絡させるために `Unit →` で始まる型を持っています。

この短絡動作の価値は `Applicative Option` のインスタンスで確認できます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean ApplicativeOption}}
```
この場合、`seq` が適用する関数がなければ、その引数を計算する必要はなく、したがって `x` は決して呼び出されません。
同じ考慮が `Except` の `Applicative` インスタンスにも影響しています：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean ApplicativeExcept}}
```
この短絡動作は、関数そのものではなく、関数を _取り囲む_ `Option` もしくは `Except` の構造にのみ依存します。

モナドは、純粋な関数型言語において、順番に実行されるステートメントの概念を捉える方法として見ることができます。
あるステートメントの結果は、どのステートメントが続けて実行されるかに影響を与えます。
これは `bind` の型で見ることができます：`{{#example_out Examples/FunctorApplicativeMonad.lean bindType}}`。
最初のステートメントの結果の値は、次に実行するステートメントを計算する関数への入力です。
`bind` の連続した使用は、命令的プログラミング言語におけるステートメントのシーケンスのようなものであり、`bind` は条件分岐やループのような制御構造を実装するのに十分な力を持っています。

このアナロジーに従って、`Applicative` は副作用を持つ言語における関数適用を捉えます。
Kotlin や C# のような言語において、関数への引数は左から右に評価されます。
先の引数によって実行される副作用は、後になされる引数によって実行される副作用よりも前に起こります。
しかし、関数は引数の特定の _値_ に依存するカスタムの短絡演算子を実装するのに十分な力はありません。

一般的に、`seq` は直接呼び出されません。
代わりに `<*>` 演算子が使用されます。
この演算子はその第二引数を `fun () => ...` でラップして、呼び出し元を簡素化します。
言い換えると、`{{#example_in Examples/FunctorApplicativeMonad.lean seqSugar}}` は `{{#example_out Examples/FunctorApplicativeMonad.lean seqSugar}}` の糖衣構文です。

`seq` を複数の引数で使用できる主な特長は、Lean の多引数関数が実際には、残りの引数を待つ別の関数を返す単一引数関数であるということです。
言い換えれば、もし `seq` の最初の引数が複数の引数を待っていれば、`seq` の結果は残りを待つことになります。
例えば、`{{#example_in Examples/FunctorApplicativeMonad.lean somePlus}}` には `{{#example_out Examples/FunctorApplicativeMonad.lean somePlus}}` の型を持つことができます。
一つの引数を提供すると、`{{#example_in Examples/FunctorApplicativeMonad.lean somePlusFour}}` は `{{#example_out Examples/FunctorApplicativeMonad.lean somePlusFour}}` の型になります。
これは `seq` と共に使用できるため、`{{#example_in Examples/FunctorApplicativeMonad.lean somePlusFourSeven}}` には `{{#example_out Examples/FunctorApplicativeMonad.lean somePlusFourSeven}}` の型があります。