# 同一性の証明

末尾再帰とアキュムレータを使用するように書き換えられたプログラムは、元のプログラムとはかなり異なって見えることがあります。
元の再帰関数はよく理解しやすいですが、実行時にスタックを使い果たすリスクがあります。
両方のバージョンのプログラムを例でテストして単純なバグを除外した後、プログラムが同値であることを一度で証明するために証明が使用されることがあります。

## `sum`が等しいことの証明

`sum`の両方のバージョンが等しいことを証明するには、スタブ証明付きの定理のステートメントから始めます：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq0}}
```

予想通り、Leanは未解決のゴールを記述しています：

```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq0}}
```

ここでは`rfl`タクティックは適用されません。なぜなら`NonTail.sum`と`Tail.sum`は定義上等しくないためです。
しかし、関数が等しいということはただの定義的な等価性以上の方法で可能です。
それはまた、同じ入力で等価な出力を生成することを証明することによって二つの関数が等しいことを証明することができるということを意味します。
言い換えれば、\\( f = g \\) は、全ての可能な入力 \\( x \\) に対して \\( f(x) = g(x) \\) を証明することによって証明され得ます。
この原理は_関数拡張性_と呼ばれています。
関数拡張性はまさに`NonTail.sum`が`Tail.sum`と等しい理由です：彼らはどちらも数値のリストを合計する。

Leanのタクティック言語では、`funext`を使用して関数拡張性が呼び出され、任意の引数に使用される名前について続けられます。
任意の引数はコンテキストに仮定として追加され、ゴールはこの引数に適用された関数が等しいことを要求するように変わります：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq1}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq1}}
```

このゴールは引数`xs`に関する帰納法によって証明することができます。
両方の`sum`関数は空リストに適用されたときに`0`を返すため、これは基本ケースとなります。
入力リストの始めに数値を追加すると、両方の関数はその数値を結果に追加するため、これは帰納ステップとなります。
`induction`タクティックを呼び出すと2つのゴールが生じます：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq2a}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq2a}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq2b}}
```

空リストに対する基本ケースは`rfl`を用いて解決できます。なぜなら両方の関数は空リストに`0`を返すからです：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq3}}
```

帰納ステップを解決する最初のステップは、目標の単純化であり、`simp`に`NonTail.sum`と`Tail.sum`を展開するよう求めることです：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq4}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq4}}
```
`Tail.sum`を展開すると、それが直ちに`Tail.sumHelper`に委譲することが明らかになります。これもまた単純化すべきです：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq5}}
```
結果として得られるゴールでは、`sumHelper`は計算を一歩進めて`y`をアキュムレータに加えています：

```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq5}}
```
帰納法の仮説との書き換えはゴールから`NonTail.sum`の全ての言及を取り除きます：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq6}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq6}}
```
この新しいゴールは、リストの合計にある数値を加えることは、その数値を`sumHelper`の初期アキュムレータとして使用するのと同じであることを述べています。
明確性のため、この新しいゴールは別の定理として証明することができます：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad0}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad0}}
```
再び、これは基本ケースが`rfl`を使用する帰納法による証明です：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad1}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad1}}
```
これは帰納的ステップであるため、ゴールは帰納法の仮説`ih`に一致するように単純化されるべきです。
`Tail.sum`および`Tail.sumHelper`の定義を使って単純化すると、以下のようになります：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad2}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad2}}
```
理想的には、帰納法の仮説を使用して`Tail.sumHelper (y + n) ys`を置き換えることができますが、一致しません。
帰納法の仮説は`Tail.sumHelper n ys`に使用できますが、`Tail.sumHelper (y + n) ys`には使用できません。
言い換えれば、この証明は行き詰まっています。