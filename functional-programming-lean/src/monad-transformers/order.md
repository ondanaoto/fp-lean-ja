# モナドトランスフォーマの順序付け

モナドトランスフォーマのスタックからモナドを構成する際、そのレイヤーの順序が重要であることに気をつける必要があります。
同じ集合のトランスフォーマを異なる順序で重ねると、異なるモナドが生まれます。

このバージョンの `countLetters` は、具体的なモナドを提供する代わりに型クラスを使用して利用可能なエフェクトの集合を記述する点で、前のバージョンとは異なります:

```lean
{{#example_decl Examples/MonadTransformers/Defs.lean countLettersClassy}}
```

ステートと例外のモナドトランスフォーマは、以下の二つの異なる順序で組み合わせることができ、どちらの順序でも両方の型クラスのインスタンスがあるモナドになります:

```lean
{{#example_decl Examples/MonadTransformers/Defs.lean SomeMonads}}
```

プログラムが例外を投げない入力に対して実行される場合、両方のモナドは似たような結果をもたらします:

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countLettersM1Ok}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countLettersM1Ok}}
```
```lean
{{#example_in Examples/MonadTransformers/Defs.lean countLettersM2Ok}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countLettersM2Ok}}
```
しかし、これらの戻り値の間には微妙な違いが存在します。
`M1`の場合、最も外側のコンストラクタは `Except.ok` で、最終状態とユニットコンストラクタの組み合わせを含みます。
`M2`の場合、最も外側のコンストラクタは組み合わせであり、ユニットコンストラクタへの `Except.ok` の適用が含まれます。
最終状態は `Except.ok` の外側にあります。
どちらの場合も、プログラムは母音と子音の数を返します。

一方、文字列が例外を投げる原因となった場合、母音と子音のカウントを返すモナドは一つだけです。
`M1`を使用すると、例外値のみが返されます:

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countLettersM1Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countLettersM1Error}}
```
`M2`を使用すると、例外値がその時点での状態とペアになります:

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countLettersM2Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countLettersM2Error}}
```

`M2`はデバッグ時に役立つかもしれない情報をより提供しているため、`M1`より優れていると考えがちです。
しかし、同じプログラムが `M1` と `M2` で_異なる_結果を計算する可能性があり、いずれの答えが必ずしも他方より良いと言える根拠はありません。
例外を扱うプログラムのステップを追加すると、これが明らかになります:

```lean
{{#example_decl Examples/MonadTransformers/Defs.lean countWithFallback}}
```
このプログラムは常に成功しますが、異なる結果で成功することがあります。
例外が投げられない場合、結果は `countLetters` と同じです:

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countWithFallbackM1Ok}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countWithFallbackM1Ok}}
```
```lean
{{#example_in Examples/MonadTransformers/Defs.lean countWithFallbackM2Ok}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countWithFallbackM2Ok}}
```
しかし、例外が投げられて捕まえられた場合、最終状態は非常に異なります。
`M1`の場合、最終状態は`"Fallback"`からの文字数だけを含みます:

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countWithFallbackM1Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countWithFallbackM1Error}}
```
`M2`では、最終状態は`"hello"`と`"Fallback"`の両方からの文字数を含むものであり、命令型言語で想定されるような結果になります:

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countWithFallbackM2Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countWithFallbackM2Error}}
```

`M1`では、例外を投げると状態が例外が捕まえられたところまで「ロールバック」されます。
`M2`では、例外を投げて捕まえることを通じても状態の変更は永続します。
この違いは `M1` と `M2` の定義を展開することで見ることができます。
`{{#example_in Examples/MonadTransformers/Defs.lean M1eval}}` を展開すると `{{#example_out Examples/MonadTransformers/Defs.lean M1eval}}` になり、`{{#example_in Examples/MonadTransformers/Defs.lean M2eval}}` を展開すると `{{#example_out Examples/MonadTransformers/Defs.lean M2eval}}` になります。
つまり、`M1 α` は初期の文字数を取り、エラーか更新された数に対応する `α` を返す関数を記述します。
`M1`で例外が投げられた場合、最終状態はありません。
`M2 α` は初期の文字数を取り、新しい文字数とエラーか `α` をペアにしたものを返す関数を記述します。
`M2`で例外が投げられると、それが状態に伴います。

## 交換可能なモナド

関数型プログラミングの専門用語で、二つのモナドトランスフォーマが_交換可能である_とは、それらが順番を変えてもプログラムの意味が変わらない場合を指します。
`StateT`と`ExceptT`を並び替えるとプログラムの結果が変わることから、ステートと例外は交換可能ではないということです。
一般的には、モナドトランスフォーマは交換可能とは限りません。

すべてのモナドトランスフォーマが交換可能でなくても、いくつかは交換可能です。
たとえば、二つの`StateT`の使用は並び替えることができます。
`{{#example_in Examples/MonadTransformers/Defs.lean StateTDoubleA}}` の定義を展開すると、型 `{{#example_out Examples/MonadTransformers/Defs.lean StateTDoubleA}}` が得られ、`{{#example_in Examples/MonadTransformers/Defs.lean StateTDoubleB}}` を展開すると `{{#example_out Examples/MonadTransformers/Defs.lean StateTDoubleB}}` が得られます。
言い換えると、それらの違いは `σ` 型と `σ'` 型がリターンタイプの異なる場所に入れ子になっていることと、引数を異なる順序で受け取っていることです。
クライアントコードは依然として同じ入力を提供する必要があり、また同じ出力を受け取ります。

ほとんどのプログラム言語では、変更可能なステートと例外の両方を持っており、それは `M2` のように動作します。
これらの言語では、例外が投げられたときにロールバックされるべきステートを表現することが難しく、通常は `M1` の明示的なステート値の受け渡しと非常に似た方法で模倣する必要があります。
モナドトランスフォーマは、問題に応じてエフェクトの順序の解釈を選択する自由を与えますが、どちらの選択もプログラムしやすいです。
しかし、トランスフォーマを順序付けするときは注意を払う必要があります。
大きな表現力には、意図したものが表現されているか確認する責任が伴います。`countWithFallback` の型シグネチャは、おそらくそれがあるべきよりも多様性を持ちすぎているでしょう。

## 練習問題

 * `ReaderT` と `StateT` が交換可能であることを、その定義を展開して結果となる型についての推論によって確認してください。
 * `ReaderT` と `ExceptT` は交換可能ですか？その定義を展開して結果となる型についての推論によって回答を確認してください。
 * `Many` の定義に基づいてモナドトランスフォーマ `ManyT` を構築し、適切な `Alternative` インスタンスを備えていることを確認してください。それが `Monad` 契約を満たしていることを確認してください。
 * `ManyT` は `StateT` と交換可能ですか？もしそうなら、定義を展開して結果となる型についての推論によって回答を確認してください。そうでない場合は、`ManyT (StateT σ Id)` のプログラムと `StateT σ (ManyT Id)` のプログラムを書いてください。それぞれのプログラムは、与えられたモナドトランスフォーマの順序付けに対してより意味をなすものです。