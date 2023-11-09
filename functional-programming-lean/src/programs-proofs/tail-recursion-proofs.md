# 同値の証明

末尾再帰とアキュムレータを使用するように書き直されたプログラムは、元のプログラムとはかなり異なるように見える場合があります。
元の再帰関数はよく理解されやすいものの、実行時にスタックを使い果たすリスクがあります。
シンプルなバグを排除するために両バージョンのプログラムを例でテストした後、証明を使って両プログラムが同値であることを一挙に示すことができます。

## `sum`が等しいことの証明

`sum`の両バージョンが等しいことを証明するために、定理文とスタブ証明を記述して始めます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq0}}
```
予想される通り、Leanは未解決のゴールを記述します：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq0}}
```

`rfl`タクティックはここでは適用できません。なぜなら、`NonTail.sum`と`Tail.sum`は定義上等価ではないからです。
しかし、関数が等しいことは定義上の等価性よりも多くの方法であり得ます。
つまり、二つの関数が同じ入力に対して等しい結果を生成することを証明することによって関数が等しいことも証明できます。
言い換えれば、\\( f = g \\)は、すべての可能な入力\\( x \\)に対して\\( f(x) = g(x) \\)を証明することで証明できます。
この原理は_関数拡張性_と呼ばれます。
関数拡張性は`NonTail.sum`が`Tail.sum`と等しい理由です：両者とも数値のリストを合計します。

Leanのタクティック言語で、関数拡張性は`funext`を使用し、任意の引数に使用する名前に続いて呼び出されます。
任意の引数はコンテキストに仮定として追加され、ゴールはこの引数に適応された関数が等しいことの証明を要求するように変更されます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq1}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq1}}
```

このゴールは引数`xs`についての帰納法によって証明できます。
両方の`sum`関数は空リストに適応されたときに`0`を返す、これが基底ケースとして働きます。
入力リストの始めに数値を追加することは両方の関数にその数値を結果に追加するように求めます。これが帰納ステップとして働きます。
`induction`タクティックを使うと二つのゴールが生まれます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq2a}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq2a}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq2b}}
```

`nil`のための基底ケースは、両関数が空リストに渡された時に`0`を返すので`rfl`を使って解決できます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq3}}
```

帰納ステップを解決する最初のステップはゴールを簡単化することで、`simp`に`NonTail.sum` と `Tail.sum`を展開してもらいます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq4}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq4}}
```
`Tail.sum`を展開すると、直ちに`Tail.sumHelper`に委譲することが明らかになり、これも簡単化されるべきです：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq5}}
```
結果のゴールでは、`sumHelper`が計算のステップを踏んで`y`をアキュムレータに加えています：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq5}}
```
帰納法の仮定で書き換えるとゴールから`NonTail.sum`のすべての記述が除去されます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq6}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq6}}
```
この新しいゴールは、リストの合計にどれかの数を加えることが`sumHelper`を使ってその数を初期アキュムレータとして使用することと同じであることを述べています。
明確さのため、この新しいゴールは別の定理として証明されるかもしれません：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad0}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad0}}
```
再び、これは基底ケースで`rfl`を使用する帰納法による証明です：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad1}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad1}}
```
帰納ステップのためには、帰納法の仮定`ih`に一致するまでゴールを簡単化すべきです。
`Tail.sum`と`Tail.sumHelper`の定義を使って簡単化すると次のようになります：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad2}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad2}}
```
理想的には、帰納法の仮定を使って`Tail.sumHelper (y + n) ys`を置換できるのですが、一致しません。
帰納法の仮定を使えるのは`Tail.sumHelper n ys`のためで`Tail.sumHelper (y + n) ys`ではありません。
言い換えれば、この証明は停止しています。

## 二度目の試み

証明を探り続けるよりも、一度立ち止まって考える時が来ています。
末尾再帰関数のバージョンが非末尾再帰バージョンと等しいのはなぜでしょうか？
根本的に、リストの各エントリでアキュムレータは再帰の結果に加えられる量と同じだけ増えます。
この洞察は、エレガントな証明を書くために使うことができます。
重要なのは、帰納法による証明は任意のアキュムレータの値に帰納法の仮定を適用できるようにセットアップされなければならないことです。

前の試みを捨て、洞察を以下の声明としてエンコードすることができます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper0}}
```
この声明で、`n`がコロンの後の型の一部であることは非常に重要です。
結果のゴールは`∀ (n : Nat)`で始まります。これは「すべての`n`に対して」という意味です：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper0}}
```
帰納法のタクティックを使うとこの「すべてに対して」の声明を含むゴールが生じます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper1a}}
```
"nil"のケースでは、ゴールは：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper1a}}
```
`cons`のための帰納ステップでは、帰納法の仮定と特定のゴールの両方に「すべての`n`に対して」と含まれます：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper1b}}
```
言い換えれば、ゴールは証明がより困難になりましたが、帰納法の仮定はそれに応じてより有用になりました。

「すべての\\( x \\)に対して」と始まる声明のための数学的証明は、任意の\\( x \\)を仮定して、声明を証明すべきです。
「任意の」とは、\\( x \\)の追加のプロパティが仮定されないという意味なので、結果の声明は_任意の_\\( x \\)に対して動作します。
Leanでは、「すべてに対して」の声明は依存関数です：特定の値に適用される場合も、それは命題の証拠を返します。
同様に、任意の\\( x \\)を選択するプロセスは、`fun x => ...`を使用するのと同じです。
タクティック言語で、任意の\\( x \\)を選択するプロセスは`intro`タクティックを使用して実行され、タクティックスクリプトが完了したときに背後で関数を生成します。
`intro`タクティックには、この任意の値に使用される名前を提供されるべきです。

"nil"ケースに`intro`タクティックを使用すると、ゴールから`∀ (n : Nat),`を取り除き、仮定`n : Nat`を追加します：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper2}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper2}}
```
この命題の平等の両側は定義上`n`に等しいので、`rfl`で十分です：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper3}}
```
`cons`ゴールも「すべての」を含んでいます：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper3}}
```
これは`intro`の使用を示唆しています。
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper4}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper4}}
```
証明ゴールは`NonTail.sum`と`Tail.sumHelper`が`y :: ys`に適用されるのを含みます。
簡単化（simplifier）は次のステップをより明確にすることができます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper5}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper5}}
```
このゴールは帰納法の仮定に非常に近く一致しています。
それに一致しない二つの方法があります：
 * 方程式の左側は`n + (y + NonTail.sum ys)`ですが、帰納法の仮定は左側が`NonTail.sum ys`に加えられる数である必要があります。
   言い換えれば、このゴールは`(n + y) + NonTail.sum ys`に書き換えられるべきで、これは自然数の加算が結合的であるために有効です。
 * 左側が`(y + n) + NonTail.sum ys`に書き換えられたとき、右側のアキュムレータの引数は`y + n`ではなく`n + y`であるべきです、一致するためには。
   この書き換えは加算がまた交換可能であるために有効です。

加算の結合性と交換性は既にLeanの標準ライブラリで証明されています。
結合性の証明は`{{#example_in Examples/ProgramsProofs/TCO.lean NatAddAssoc}}`と名付けられ、その型は`{{#example_out Examples/ProgramsProofs/TCO.lean NatAddAssoc}}`です。一方、交換性の証明は`{{#example_in Examples/ProgramsProofs/TCO.lean NatAddComm}}`と呼ばれ、その型は`{{#example_out Examples/ProgramsProofs/TCO.lean NatAddComm}}`です。
通常、`rw`タクティックは等式を型とする式が与えられます。
しかしながら、代わりに等式を戻り型とする依存関数が引数として与えられると、ゴールの中に一致するものを見つけるためにその関数に引数を見つけようと試みます。
結合性を適用する機会は1回のみですが、証明ゴールに一致する`{{#example_in Examples/ProgramsProofs/TCO.lean NatAddAssoc}}`の右側なので書き換えの方向を逆にしなければなりません：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper6}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper6}}
```
しかし、`{{#example_in Examples/ProgramsProofs/TCO.lean NatAddComm}}`で直接書き換えると誤った結果となります。
`rw`タクティックは誤った場所での書き換えを推測し、意図しないゴールを導きます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper7}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper7}}
```
これは`Nat.add_comm`に明示