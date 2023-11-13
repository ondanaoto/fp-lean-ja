```markdown
# より多くの不等式

Leanの組み込みの証明自動化は、`arrayMapHelper` と `findHelper` が終了することを確認するのに十分です。
必要だったのは、各再帰呼び出しで減少する値を表す式を提供することだけでした。
しかし、Leanの組み込み自動化は魔法ではなく、しばしばいくつかの助けを必要とします。

## マージソート

終了証明が非自明な関数の一例は `List` 上のマージソートです。
マージソートは2つのフェーズで構成されます：まず、リストは半分に分かれます。
それぞれの半分はマージソートを使用してソートされ、そして結果は二つのソートされたリストを大きなソートされたリストに結合する関数を使用してマージされます。
基底ケースは空リストと単一要素のリストであり、どちらもすでにソートされたとみなされます。

二つのソートされたリストをマージするためには、考えるべき2つの基本ケースがあります：
 1. 入力リストの一方が空の場合、結果はもう一方のリストです。
 2. 両方のリストが空でない場合、それらの頭を比較すべきです。関数の結果は二つの頭のうち小さい方で、それに両方のリストの残りのエントリをマージした結果が続きます。

これはどちらのリストに対しても構造的に再帰的ではありません。
再帰が終了するのは、各再帰呼び出しでどちらかのリストからエントリが削除されるためですが、どちらのリストであっても構いません。
`termination_by` 句は両方のリストの長さの合計を減少値として使用します：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean merge}}
```

リストの長さを使用する代わりに、両方のリストを含んだペアを提供することもできます：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean mergePairTerm}}
```
これは、Leanが `WellFoundedRelation` という型クラスを通じてデータのサイズの概念を組み込んでおり、ペアに対するインスタンスが自動的に、ペアの第一または第二のアイテムが縮小する場合にそれをより小さなものと見なすからです。

リストを分割する簡単な方法は、入力リストの各エントリを二つの交互の出力リストに追加することです：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean splitList}}
```

マージソートは基底ケースに達しているかどうかを確認します。
もし達していれば、入力リストを返します。
そうでなければ、入力を分割し、それぞれの半分をソートした結果をマージします：
```lean
{{#example_in Examples/ProgramsProofs/Inequalities.lean mergeSortNoTerm}}
```
Leanのパターンマッチコンパイラーは、`if`によって導入された仮定 `h` は `xs.length < 2` をテストするために一つ以上のエントリを持たないリストを排除しているので、「ケースが不足している」というエラーはありません。
しかし、このプログラムは常に終了しますが、構造的に再帰的ではありません：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean mergeSortNoTerm}}
```
終了する理由は、`splitList`が常にその入力よりも短いリストを返すからです。
したがって、`halves.fst` と `halves.snd` の長さは `xs` の長さよりも少なくなります。
これは `termination_by` 句を使って表現することができます：
```lean
{{#example_in Examples/ProgramsProofs/Inequalities.lean mergeSortGottaProveIt}}
```
この句により、エラーメッセージが変わります。
関数が構造的に再帰的ではないという不満ではなく、 Leanは `(splitList xs).fst.length < xs.length` の自動証明に失敗したと指摘しています：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean mergeSortGottaProveIt}}
```

## リストを分割することはそれを短くする

`(splitList xs).snd.length < xs.length` を証明することも必要になります。
`splitList`は二つのリストにエントリを交互に追加するため、両方のステートメントを一度に証明するのが最も簡単で、証明の構造が `splitList`の実装に使用されたアルゴリズムに従うことができます。
言い換えると、`∀(lst : List), (splitList lst).fst.length < lst.length ∧ (splitList lst).snd.length < lst.length` を証明するのが最も簡単です。

残念ながら、このステートメントは偽です。
特に、`{{#example_in Examples/ProgramsProofs/Inequalities.lean splitListEmpty}}` は `{{#example_out Examples/ProgramsProofs/Inequalities.lean splitListEmpty}}` です。両方の出力リストは長さ `0` であり、これは入力リストの長さ `0` よりも短くありません。
同様に、`{{#example_in Examples/ProgramsProofs/Inequalities.lean splitListOne}}` は `{{#example_out Examples/ProgramsProofs/Inequalities.lean splitListOne}}` に評価され、`["basalt"]` は `["basalt"]` より短くありません。
しかし、`{{#example_in Examples/ProgramsProofs/Inequalities.lean splitListTwo}}` は `{{#example_out Examples/ProgramsProofs/Inequalities.lean splitListTwo}}` に評価され、これらの出力リストは入力リストよりも短くなります。

実は、出力リストの長さは常に入力リストの長さ以下ですが、入力リストに少なくとも二つのエントリが含まれている場合にのみ、厳密に短くなります。
前者のステートメントを証明してから、後者のステートメントに拡張するのが最も簡単です。
定理ステートメントから始めます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le0}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le0}}
```
`splitList`はリストに構造的に再帰的なので、証明は帰納法を使用すべきです。
`splitList`の構造的再帰は帰納法の証明に完全に一致します：帰納法の基底ケースは再帰の基底ケースに一致し、帰納ステップは再帰呼び出しに一致します。
`induction` タクティックは2つのゴールを与えます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le1a}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le1a}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le1b}}
```

`nil` ケースのゴールは、`splitList` の定義を展開するように指示することにより、単純化を呼び出すことによって証明することができます。なぜなら、空リストの長さは空リストの長さより小さいか等しいからです。
同様に、`splitList` とともに単純化することにより、`cons` ケースはゴールの長さに `Nat.succ` を配置します：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le2}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le2}}
```
これは、`List.length` の呼び出しがリスト `x :: xs` の頭を消費し、入力リストの長さと最初の出力リストの長さの両方を `Nat.succ` に変換するためです。

Leanで `A ∧ B` と書くのは `And A B` のショートカットです。
`And` は `Prop` 宇宙内の構造型です：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean And}}
```
言い換えれば、`A ∧ B` の証明は `left` フィールドに `A` の証明と `right` フィールドに `B` の証明が適用された `And.intro` コンストラクタで構成されています。

`cases` タクティックは、データ型の各コンストラクタまたは提案の潜在的な証明を順番に考慮するための証明を可能にします。
それは再帰を伴わない `match` 式に対応します。
構造に `cases` を使用すると、構造が分解され、各フィールドの構造に対する仮定が追加されます。これはパターンマッチ式がプログラムで使用するために構造のフィールドを抽出するようなものです。
構造には一つのコンストラクタしかないので、構造に `cases` を使用することは追加のゴールではありません。

`ih` は `List.length (splitList xs).fst ≤ List.length xs ∧ List.length (splitList xs).snd ≤ List.length xs` の証明であるので、`cases ih` を使用すると `List.length (splitList xs).fst ≤ List.length xs` という仮定と `List.length (splitList xs).snd ≤ List.length xs` という仮定になります：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le3}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le3}}
```

証明のゴールも `And` なので、`constructor` タクティックを使って `And.intro` を適用することができ、各引数のゴールが作成されます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le4}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le4}}
```

`left` ゴールは `left✝` 仮定に非常に似ていますが、ゴールは不等式の両側を `Nat.succ` でラップします。
同様に、`right` ゴールは `right✝` 仮定に似ていますが、ゴールは `Nat.succ` を入力リストの長さにのみ追加します。
これらの `Nat.succ` のラッピングがステートメントの真実を保つことを証明する時がきました。

### 両側に1を追加する

`left` ゴールの場合、証明すべきステートメントは `Nat.succ_le_succ : n ≤ m → Nat.succ n ≤ Nat.succ m` です。
言い換えれば、もし `n ≤ m` であれば、両側に1を追加してもこの事実は変わりません。
なぜこれが真実なのでしょうか？
`n ≤ m` の証明は `m - n` という `Nat.le.step` コンストラクタのインスタンスであり、前よりも大きい数値に `refl` が適用される場合、同じ数の `step` コンストラクタがラップされます。

より形式的には、証明は `n ≤ m` の証拠に対する帰納法によるものです。
証拠が `refl` の場合、それは `n = m` なので、`Nat.succ n = Nat.succ m` であり `refl` を再び使用できます。
証拠が `step` の場合、帰納法の仮説は `Nat.succ n ≤ Nat.succ m` の証拠を提供し、目標は `Nat.succ n ≤ Nat.succ (Nat.succ m)` を示すことです。
これは帰納仮説と一緒に `step` を使用することによって行われます。

Leanでの定理ステートメントは：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean succ_le_succ0}}
```
エラーメッセージはそれを再確認します：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean succ_le_succ0}}
```

最初のステップは、仮説 `n ≤ m` をスコープ内にもたらして名前を付ける `intro` タクティックを使用することです：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean succ_le_succ1}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean succ_le_succ1}}
```

証明は `n ≤ m` の証拠の帰納法によるので、次のタクティックは `induction h` です：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean succ_le_succ3}}
```
これにより、`Nat.le` の各コンストラクタについて一度に二つのゴールが得られます：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean succ_le_succ3}}
```
`refl` のゴールは `refl` を使用して解決することができますが、`constructor` タクティックが選びます。
`step` のゴールも `step` コンストラクタの使用を必要とします：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le4}}
```
これで、対象の `And` の左右両側のゴールが得られました。

これらのゴールは、`And` 構造のフィールドに挙げられた名前と同じものです。よって、`case` タクティック（`cases` と混同しないように注意）を使って、それぞれに集中することができます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le5a}}
```
単一のエラーではなく、それぞれの `skip` の上にある二つのメッセージがあります。
`left` のゴールには、`Nat.succ_le_succ` を使用できます：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le5a}}
```
右側のゴールでは、`Nat.le_succ_of_le` が適合します：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le5b}}
```
両方の定理には、`n ≤ m` という前提条件が含まれます。
これらは `left✝` および `right✝` の仮定として見つかるため、`assumption` タクティックで最後のゴールを処理することができます：
```leantac
{{#example_decl Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le}}
```
次に、少なくとも二つの要素を含むリストを分割した場合に、それぞれの結果が厳密に短くなることを証明する必要がある実際の定理に戻ります。そのためには次のように始めます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_start}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_start}}
```
`lst` が少なくとも二つの要素を含んでいるため、`match` を使ってそれらを明らかにし、依存型のパターンマッチングを通じて型を詰めることができます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_1}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_1}}
```
`simplify` を使うと `x` と `y` が取り除かれ、計算されたリストの長さがそれぞれ `Nat.succ` を増やします：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_2}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_2}}
```
`simplify` を `simp_arith` に置き換えると、これらの `Nat.succ` コンストラクタが取り除かれます。なぜなら、`simp_arith` では `n + 1 < m + 1` が `n < m` を意味するという事実を利用しているからです：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_2b}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_2b}}
```
このゴールは、`splitList_shorter_le` と一致するので、これを使用して証明を結論付けることができます：
```leantac
{{#example_decl Examples/ProgramsProofs/Inequalities.lean splitList_shorter}}
```