# モナドトランスフォーマの並び順

モナドトランスフォーマのスタックからモナドを構築する際、その層の順序が重要であることに注意する必要があります。
同じセットのトランスフォーマを異なる順序で積み重ねると、異なるモナドが生まれます。

このバージョンの `countLetters` は、具体的なモナドを提供する代わりに型クラスを使用して使用可能な効果のセットを記述する点で、前のバージョンと異なります：

```lean
{{#example_decl Examples/MonadTransformers/Defs.lean countLettersClassy}}
```

ステートと例外のモナドトランスフォーマを、以下の2つの異なる順序で組み合わせることができ、どちらの順序でも両方の型クラスのインスタンスがあるモナドになります：

```lean
{{#example_decl Examples/MonadTransformers/Defs.lean SomeMonads}}
```

プログラムが例外を投げない入力に対して実行される場合、両方のモナドは似たような結果をもたらします：

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
`M1`の場合、最も外側のコンストラクタは`Except.ok`で、最終状態とユニットコンストラクタの組み合わせを含んでいます。
`M2`の場合、最も外側のコンストラクタはその組み合わせであり、ユニットコンストラクタに適用された`Except.ok`が含まれます。
最終状態は`Except.ok`の外にあります。
どちらの場合も、プログラムは母音と子音の数を返します。

一方で、文字列が例外を投げる原因となる場合、最終的に母音と子音のカウントを返すモナドは一つだけです。
`M1`を使用すると、例外値のみが返されます：

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countLettersM1Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countLettersM1Error}}
```
`M2`を使用すると、例外値がその時点の状態とペアになります：

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countLettersM2Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countLettersM2Error}}
```

`M2`はデバッグ時に役立つかもしれない情報をより多く提供しているため、`M1`よりも優れていると考えがちです。
しかし、同じプログラムが`M1`と`M2`で_異なる_結果を計算する可能性があり、どちらの答えが必ずしも良いとは限りません。
例外を取り扱うプログラムのステップを行うと、これが明らかになります：

```lean
{{#example_decl Examples/MonadTransformers/Defs.lean countWithFallback}}
```
このプログラムは常に成功しますが、異なる結果で成功することもあります。
例外が投げられない場合、結果は`countLetters`と同じです：

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
しかし、例外が投げられて捕捉された場合、最終状態は非常に異なります。
`M1`の場合、最終状態は`"Fallback"`からの文字数のみを含みます：

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countWithFallbackM1Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countWithFallbackM1Error}}
```
`M2`の場合、最終状態は`"hello"`と`"Fallback"`の両方からの文字数を含んでおり、命令型の言語で期待されるような結果になります：

```lean
{{#example_in Examples/MonadTransformers/Defs.lean countWithFallbackM2Error}}
```
```output info
{{#example_out Examples/MonadTransformers/Defs.lean countWithFallbackM2Error}}
```

`M1`では、例外が投げられると状態は例外が捕捉された地点まで"ロールバック"されます。
`M2`では、例外が捕捉されても状態の変更は保持されます。
この違いは`M1`と`M2`の定義を展開することで確認できます。
`{{#example_in Examples/MonadTransformers/Defs.lean M1eval}}`を展開すると`{{#example_out Examples/MonadTransformers/Defs.lean M1eval}}`になり、`{{#example_in Examples/MonadTransformers/Defs.lean M2eval}}`を展開すると`{{#example_out Examples/MonadTransformers/Defs.lean M2eval}}`になります。
つまり、`M1 α`は初期の文字数を取り、エラーか更新された数に対応する`α`を返す関数を記述します。
`M1`で例外が投げられた場合、最終状態は存在しません。
`M2 α`は初期の文字数を取り、新しい文字数とエラーか`α`をペアにしたものを返す関数を記述します。
`M2`で例外が投げられると、それが状態に影響します。

## 交換可能なモナド

関数型プログラミングの専門用語で、2つのモナドトランスフォーマが_交換可能である_とは、それらが順番を変えてもプログラムの意味が変わらない場合を指します。
`StateT`と`ExceptT`の並び替えはプログラムの結果を変えるため、ステートと例外は交換可能ではありません。
一般的に、モナドトランスフォーマは交換可能ではありません。

すべてのモナドトランスフォーマが交換可能でなくても、いくつかは交換可能です。
例えば、2つの`StateT`は並び替え可能です。
`{{#example_in Examples/MonadTransformers/Defs.lean StateTDoubleA}}`の定義を展開すると、型`{{#example_out Examples/MonadTransformers/Defs.lean StateTDoubleA}}`が得られ、`{{#example_in Examples/MonadTransformers/Defs.lean StateTDoubleB}}`を展開すると`{{#example_out Examples/MonadTransformers/Defs.lean StateTDoubleB}}`が得られます。
言い換えれば、それらの違いはσ型とσ'型がリターンタイプの異なる位置に入れ子になっていることと、引数を異なる順番で受け取っていることです。
クライアントコードは依然として同じ入力を提供する必要があり、また同じ出力を受け取ります。

ほとんどのプログラミング言語では、変更可能な状態と例外の両方を持っており、それは`M2`のように動作します。
これらの言語では、例外が発生した時にロールバックされるべき状態を表現することが困難で、通常は`M1`の明示的な状態値を使用することで模倣する必要があります。
モナドトランスフォーマは問題に応じてエフェクトの順序の解釈を選択する自由を与えますが、どちらの選択もプログラムするのは容易です。
しかし、トランスフォーマを順序付ける際は注意が必要です。
大きな表現力が与えられている分、意図したものが表現されていることを確認する責任が伴います。`countWithFallback`の型シグネチャは、おそらくそれがあるべきよりも多様性を持ちすぎているでしょう。

## 練習問題

* `ReaderT`と`StateT`が交換可能であることを、その定義を展開して得られる型を通じて推論することで確認してください。
* `ReaderT`と`ExceptT`は交換可能ですか？その定義を展開して得られる型を通じて推論することで回答を確認してください。
* `Many`の定義に基づいてモナドトランスフォーマ`ManyT`を構築し、適切な`Alternative`インスタンスが提供されているかを確認してください。またそれが`Monad`契約を満たしていることを確認してください。
* `ManyT`は`StateT`と交換可能ですか？もしそうなら、その定義を展開して得られる型を通じて推論することで回答を確認してください。そうでなければ、`ManyT (StateT σ Id)`のプログラムと`StateT σ (ManyT Id)`のプログラムを書いてください。それぞれのプログラムは、与えられたモナドトランスフォーマの順序に対してより意味をなすものです。