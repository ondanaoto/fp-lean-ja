```markdown
# 配列と終了条件

効率的なコードを書くためには、適切なデータ構造を選択することが重要です。
リンクリストにはそれらの場所があります：一部のアプリケーションでは、リストの末尾を共有する能力が非常に重要です。
しかし、可変長の逐次的なデータコレクションに対するほとんどのユースケースは、メモリオーバーヘッドが少なく、局所性が良い配列によってよりよく対応されます。

ただし、配列はリストに対して以下の2つの欠点があります：
1. 配列はインデックスを通じてアクセスされるため、安全を保つためには[証明義務](../props-proofs-indexing.md)が課せられます。
2. 左から右へと配列全体を処理するループは末尾再帰関数ですが、各呼び出しで減少する引数を持っていません。

配列を効果的に使用するには、配列のインデックスが範囲内にあること、および配列のサイズに近づくインデックスがプログラムの終了をもたらすことをLeanに証明する方法を知っている必要があります。
これらはいずれも、命題の平等ではなく、不等命題を使用して表現されます。

## 不等式

異なる型には異なる順序の概念があるため、不等式は `LE` および `LT` と呼ばれる2つの型クラスによって管理されます。
[標準型クラスに関するセクション](../type-classes/standard-classes.md#equality-and-ordering)のテーブルは、これらのクラスが構文とどのように関連しているかを説明しています：

| 式 | 展開 | クラス名 |
|----|------|--------|
| `{{#example_in Examples/Classes.lean ltDesugar}}` | `{{#example_out Examples/Classes.lean ltDesugar}}` | `LT` |
| `{{#example_in Examples/Classes.lean leDesugar}}` | `{{#example_out Examples/Classes.lean leDesugar}}` | `LE` |
| `{{#example_in Examples/Classes.lean gtDesugar}}` | `{{#example_out Examples/Classes.lean gtDesugar}}` | `LT` |
| `{{#example_in Examples/Classes.lean geDesugar}}` | `{{#example_out Examples/Classes.lean geDesugar}}` | `LE` |

言い換えれば、型は `<` と `≤` 演算子の意味をカスタマイズすることができ、 `>` と `≥`は `<` と `≤` からそれぞれの意味を導き出しています。
`LT` と `LE` のクラスは `Bool`ではなく命題を返すメソッドを持っています：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean less}}
```

`Nat` の `LE` インスタンスは `Nat.le` に委譲します：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean LENat}}
```
`Nat.le` を定義するには、まだ紹介されていないLeanの機能が必要です：それは帰納的に定義された関係です。

### 帰納的に定義された命題、述語、および関係

`Nat.le` は _帰納的に定義された関係_ です。
`inductive` を使用して新しいデータ型を作成できるのと同様に、新しい命題を作成するためにも使用できます。
命題が引数を取る場合、それは一部の引数に対しては真であるが、全てに対して真であるわけではない _述語_ として参照されます。
複数の引数を取る命題は _関係_ と呼ばれます。

帰納的に定義された命題の各コンストラクターは、それを証明する方法です。
言い換えれば、命題の宣言はそれが真であるさまざまな形の証拠を記述しています。
シングルコンストラクターを持つ引数なしの命題は証明が非常に簡単になることがあります：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean EasyToProve}}
```
証明はそのコンストラクターを使用することから成ります：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean fairlyEasy}}
```
実際に、「True」という命題は、常に簡単に証明できるべきであり、`EasyToProve` と同じように定義されます：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean True}}
```

引数を取らない帰納的に定義された命題は、帰納的に定義されたデータ型ほど興味深くありません。
データがそれ自体で興味深いからです—自然数 `3` は `35` とは異なり、3枚のピザを注文した人は35枚が30分後に彼らのドアに届いたら怒るでしょう。
命題のコンストラクターは命題が真である方法を記述していますが、一度命題が証明されると、どの基本的なコンストラクターが使用されたかを知る必要はありません。
これが `Prop` 宇宙におけるほとんどの興味深い帰納的に定義された型が引数を取る理由です。

帰納的に定義された述語 `IsThree` は、その引数が3であることを述べています：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean IsThree}}
```
ここで使用されているメカニズムは、[`HasCol`]といった [インデックス付きファミリー](../dependent-types/typed-queries.md#column-pointers) と全く同様ですが、結果として得られる型は使用できるデータではなく証明可能な命題です。

この述語を使用すると、実際に3が3であることを証明することが可能です：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean threeIsThree}}
```
同様に、`IsFive` はその引数が `5` であると述べる述語です：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean IsFive}}
```

ある数が3であるならば、それに2を加えた結果は5であるべきです。
これは定理のステートメントとして表現できます：
```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive0}}
```
結果として得られるゴールは関数型です：
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive0}}
```
したがって、`intro` タクティックを使用して引数を仮定に変換することができます：
```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1}}
```
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1}}
```
`n` が3であるという仮定があれば、`IsFive` の構築子を使用して証明を完成することが可能です：
```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1a}}
```
しかし、これはエラーを発生させます：
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1a}}
```
このエラーは、`n + 2` が定義上 `5` に等しくないために発生します。
通常の関数定義では、仮定 `three` に依存したパターンマッチングを使用して `n` を `3` に洗練させることができます。
依存するパターンマッチングのタクティックに相当するものは `cases` であり、その構文は `induction` のようです：
```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive2}}
```
残るケースでは、`n` は `3` に洗練されています：
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive2}}
```
`3 + 2` が定義上 `5` に等しいため、構築子は今適用可能です：
```leantac
{{#example_decl Examples/ProgramsProofs/Arrays.lean threePlusTwoFive3}}
```

標準の偽の命題 `False` にはコンストラクターがないため、直接の証拠を提供することは不可能です。
`False` の証拠を提供する唯一の方法は、仮定自体が不可能である場合と同様に、タイプシステムが到達不可能であると見ることができるコードをマークするために `nomatch` を使用できる場合です。
[初めての証明に関する間奏曲](../props-proofs-indexing.md#connectives) で説明されているように、否定 `Not A` は `A → False` の略です。
`Not A` はまた `¬A` と書くことができます。

4が3でないということはありません：
```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean fourNotThree0}}
```
初期の証明ゴールには `Not` が含まれます：
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean fourNotThree0}}
```
実際にそれが関数型であることは、 `simp` を使用することで露見できます：
```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean fourNotThree1}}
```
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean fourNotThree1}}
```
ゴールが関数型であるため、`intro` を使用して引数を仮定に変換できます。
`simp` を保持する必要はありません、なぜなら `intro` は `Not` の定義自体をアンフォールドできるからです：
```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean fourNotThree2}}
```
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean fourNotThree2}}
```
この証明では、`cases` タクティックはすぐにゴールを解決します：
```leantac
{{#example_decl Examples/ProgramsProofs/Arrays.lean fourNotThreeDone}}
```
`Vect String 2` のパターンマッチに `Vect.nil` のケースを含める必要がないのと同じように、`IsThree 4` のケース証明に `isThree` のケースを含める必要はありません。

### 自然数の不等式

`Nat.le` の定義にはパラメータとインデックスがあります：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean NatLe}}
```
パラメータ `n` は小さくなければならない数であり、インデックスは `n` 以上でなければならない数です。
`refl` コンストラクターは両方の数が等しい場合に使用され、`step` コンストラクターはインデックスが `n` より大きい場合に使用されます。

証拠の観点から、 \\( n \leq k \\) の証明はいくつかの数 \\( d \\) を見つけることで構成され、 \\( n + d = m \\) です。
Leanでは、証明は `Nat.le.refl` コンストラクターに \\( d \\) インスタンスの `Nat.le.step` でラップされて構成されます。
各 `step` コンストラクターはそのインデックス引数に1を加算するため、 \\( d \\) の `step` コンストラクターは大きい数に \\( d \\) を加算します。
たとえば、4が7以下であるという証拠は `refl` の周りに3つの `step` で構成されています：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean four_le_seven}}
```

厳密な未満の関係は、左の数に1を加えることで定義されます：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean NatLt}}
```
4が7より厳密に小さいという証拠は `refl` の周りに2つの `step` で構成されています：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean four_lt_seven}}
```
これは、`4 < 7` が `5 ≤ 7` と等価であるためです。

## 終了条件の証明

関数 `Array.map` は配列を関数で変換し、入力配列の各要素に関数を適用した結果を含む新しい配列を返します。
末尾再帰関数としてそれを書くことは、アキュムレータ内の出力配列を渡す関数に委譲する通常のパターンに従います。
アキュムレータは空の配列で初期化されます。
アキュムレータを渡すヘルパー関数は、配列の現在のインデックスを追跡する引数も取り、それは `0` から始まります：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayMap}}
```

ヘルパーは各繰り返しでインデックスがまだ範囲内にあるかどうかをチェックするべきです。
もしそうならば、変換された要素をアキュムレータの末尾に追加しインデックスを `1` 増やしてループを続けるべきです。
そうでなければ、プログラムは終了しアキュムレータを返すべきです。最初の実装では、Leanは配列インデックスが有効であることを証明できないため失敗します：
```lean
{{#example_in Examples/ProgramsProofs/Arrays.lean mapHelperIndexIssue}}
```
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean mapHelperIndexIssue}}
```
しかし、条件式は既に配列インデックスの有効性が必要とする正確な条件（すなわち、`i < arr.size`）をチェックしています。`if`に名前を追加すると問題が解決します。なぜなら、それは配列インデックスの戦略が使用できる前提を追加するからです：
```lean
{{#example_in Examples/ProgramsProofs/Arrays.lean arrayMapHelperTermIssue}}
```
しかし、Leanは修正されたプログラムを受け入れません。なぜなら、再帰的な呼び出しは入力コンストラクターのいずれかの引数に対して行われないからです。実際、アキュムレータとインデックスの両方が増えるのではなく、縮小しています：
```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean arrayMapHelperTermIssue}}
```
それにもかかわらず、この関数は終了します。そのため、単に`partial`とマークすることは不幸です。

なぜ`arrayMapHelper`は終了するのでしょうか？各繰り返しは、インデックス`i`が配列`arr`の範囲内にまだあるかどうかをチェックします。もし範囲内であれば、`i`は増加し、ループは繰り返します。そうでなければ、プログラムは終了します。`arr.size`は有限数であるため、`i`は有限回数しか増加することはできません。各呼び出しで関数の引数が減少するわけではありませんが、`arr.size - i`はゼロに向かって減少します。

Leanに別の式を使用して終了を指示することを伝えるために、定義の最後に`termination_by`節を提供できます。`termination_by`節には2つのコンポーネントがあります：関数の引数の名前と、各呼び出しで減少するようなその名前を使用した式です。`arrayMapHelper`の場合、最終的な定義は次のようになります：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayMapHelperOk}}
```

同様の終了証明は`Array.find`という関数を書くために使用されます。これは配列内の最初の要素を検索し、その要素とそのインデックスの両方を返します：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayFind}}
```
再び、ヘルパー関数は`i`が増加するにつれて`arr.size - i`が減少するために終了します：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayFindHelper}}
```

すべての終了引数がこれほど簡単なわけではありません。しかし、関数の引数に基づいたいくつかの式が各呼び出しで減少するという基本的な構造は、すべての終了証明において発生します。ときには創造力が必要です。なぜなら、関数が終了する理由を把握するため、そしてときにはLeanが終了引数を受け入れるために追加の証明が必要です。

## 練習問題

 * 配列に対して、`termination_by`節を使用し、末尾再帰アキュムレータ渡し関数を使用して`ForM (Array α)`インスタンスを実装してください。
 * `termination_by`節を必要とせず、末尾再帰アキュムレータ渡し関数を使用して配列を反転する関数を実装してください。
 * `Array.map`、`Array.find`及び`ForM`インスタンスをアイデンティティモナドの`for ... in ...`ループを使用し再実装し、結果のコードを比較してください。
 * `for ... in ...`ループをアイデンティティモナドで使用し配列反転を再実装してください。それを末尾再帰関数と比較してください。