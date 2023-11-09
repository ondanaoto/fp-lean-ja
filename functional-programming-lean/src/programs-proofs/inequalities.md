# 不等式についてさらに

Leanの組み込みの証明自動化は、`arrayMapHelper`と`findHelper`が終了することをチェックするのに十分です。
必要だったのは、再帰呼び出しによってその値が減少する式を提供することでした。
それでも、Leanの組み込みの自動化は魔法ではなく、しばしばいくらかの助けが必要です。

## マージソート

終了証明が非自明な関数の一例は、`List`上のマージソートです。
マージソートは2つの段階で構成されています：最初に、リストは半分に分割されます。
各半分はマージソートを使用してソートされ、次に2つのソートされたリストを1つのより大きなソートされたリストに結合する関数を使用して結果がマージされます。
基本ケースは空のリストと単一のリストで、どちらもすでにソートされていると見なされています。

2つのソートされたリストをマージするには、2つの基本ケースを考慮する必要があります：
 1. 入力リストの1つが空の場合、結果はもう1つのリストです。
 2. 両方のリストが空でない場合は、それらのヘッドを比較すべきです。関数の結果は、2つのヘッドのうち小さい方に続いて、両方のリストの残りのエントリをマージした結果です。

これはどちらのリストに対しても構造的に再帰的ではありません。
再帰が終了するのは、各再帰呼び出しで1つの2つのリストからエントリが削除されるためですが、どちらのリストにもなり得ます。
`termination_by`節は、両方のリストの長さの合計を減少する値として使用しています：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean merge}}
```

リストの長さを使用することに加えて、両方のリストを含むペアも提供することができます：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean mergePairTerm}}
```
これは、Leanには、`WellFoundedRelation`と呼ばれる型クラスを通じて表現されるデータのサイズに関する組み込みの概念があるためです。
ペアのインスタンスは、ペアのいずれかの最初のアイテムまたは2番目のアイテムが縮小される場合、それらを自動的に小さくすると見なされます。

リストを分割する簡単な方法は、入力リストの各エントリを2つの交互の出力リストに追加することです：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean splitList}}
```

マージソートは基本ケースに達したかどうかをチェックします。
そうであれば、入力リストを返します。
そうでなければ、入力を分割し、ソートされた各半分の結果をマージします：
```lean
{{#example_in Examples/ProgramsProofs/Inequalities.lean mergeSortNoTerm}}
```
Leanのパターンマッチコンパイラは、`if`によって導入された仮定`h`が、`xs.length < 2`をテストするリストが1エントリより長いものを排除していることを認識するため、「ケース不足」エラーはありません。
しかし、このプログラムは常に終了しますが、構造的に再帰的ではありません：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean mergeSortNoTerm}}
```
それが終了する理由は、`splitList`が常にその入力よりも短いリストを返すためです。
したがって、`halves.fst`と`halves.snd`の長さは`xs`の長さよりも小さいです。
これは`termination_by`節を使用して表現できます：
```lean
{{#example_in Examples/ProgramsProofs/Inequalities.lean mergeSortGottaProveIt}}
```
この節を使用すると、エラーメッセージが変わります。
関数が構造的に再帰的ではないと不平を言う代わりに、Leanはそれが自動的に`(splitList xs).fst.length < xs.length`を証明できなかったことを指摘します：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean mergeSortGottaProveIt}}
```

## リストを分割すると短くなる

`(splitList xs).snd.length < xs.length`も証明する必要があります。
`splitList`は2つのリストにエントリを交互に追加するため、最も証明しやすいのは両方のステートメントを同時に証明することです。つまり、証明の構造が`splitList`を実装するために使用したアルゴリズムに従うことができます。
言い換えると、`∀(lst : List), (splitList lst).fst.length < lst.length ∧ (splitList lst).snd.length < lst.length`を証明するのが最も簡単です。

残念ながら、このステートメントは偽です。
特に、`{{#example_in Examples/ProgramsProofs/Inequalities.lean splitListEmpty}}`は`{{#example_out Examples/ProgramsProofs/Inequalities.lean splitListEmpty}}`です。どちらの出力リストも長さ`0`であり、入力リストの長さ`0`よりは短くありません。
同様に、`{{#example_in Examples/ProgramsProofs/Inequalities.lean splitListOne}}`は`{{#example_out Examples/ProgramsProofs/Inequalities.lean splitListOne}}`となり、`["basalt"]`は`["basalt"]`より短くありません。
しかし、`{{#example_in Examples/ProgramsProofs/Inequalities.lean splitListTwo}}`は`{{#example_out Examples/ProgramsProofs/Inequalities.lean splitListTwo}}`と評価され、これらの出力リストは入力リストよりも短いです。

出力リストの長さは常に入力リストの長さ以下ですが、入力リストに少なくとも2つのエントリがある場合にのみ、厳密に短くなります。
最初のステートメントを証明し、それを後者のステートメントに拡張するのが最も簡単です。
定理のステートメントとして始めます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le0}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le0}}
```
`splitList`はリストに構造的に再帰的なので、証明は帰納法を使用する必要があります。
`splitList`の構造的再帰は、帰納法による証明に完全に適合します：帰納法の基本ケースは再帰の基本ケースと一致し、帰納法のステップは再帰的な呼び出しと一致します。
`induction`戦術により2つのゴールが与えられます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le1a}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le1a}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le1b}}
```

`nil`ケースのゴールは、単純化器を呼び出して`splitList`の定義を展開させることで証明されます。なぜなら空リストの長さは空リストの長さ以下だからです。
同様に、`cons`ケースで`splitList`を単純化すると、ゴール内の長さに`Nat.succ`が周囲に付けられます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le2}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le2}}
```
これは、`List.length`の呼び出しがリスト`x :: xs`の頭を消費し、入力リストの長さと最初の出力リストの長さの両方に`Nat.succ`を変換するためです。

Leanでの`A ∧ B`の書き方は`And A B`の略です。
`And`は`Prop`宇宙の構造タイプです：
```lean
{{#example_decl Examples/ProgramsProofs/Inequalities.lean And}}
```
言い換えると、`A ∧ B`の証明は、`left`フィールドに`A`の証明を、`right`フィールドに`B`の証明を適用した`And.intro`コンストラクタで構成されています。

`cases`戦術によって、データタイプの各コンストラクタまたは命題の各潜在的証明を順番に検討することができます。
これは、再帰のない`match`式に対応します。
`cases`を構造体に使用すると、構造体が分解され、構造の各フィールドに対する仮定が追加されます。これは、パターンマッチ式がプログラムの使用のために構造体のフィールドを抽出するのと同じようにです。
構造が1つのコンストラクタしか持たないため、構造に`cases`を使用すると追加のゴールは発生しません。

`ih`が`List.length (splitList xs).fst ≤ List.length xs ∧ List.length (splitList xs).snd ≤ List.length xs`の証明であるので、`cases ih`を使用すると、`List.length (splitList xs).fst ≤ List.length xs`と、`List.length (splitList xs).snd ≤ List.length xs`の仮定があることを意味します：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le3}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le3}}
```

証明のゴールも`And`なので、`constructor`戦術を使用して`And.intro`を適用できます。この結果、各引数についてのゴールが得られます：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le4}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean splitList_shorter_le4}}
```

`left`ゴールは`left✝`の仮定に非常に似ていますが、ゴールは不等式の両側を`Nat.succ`で包むだけです。
同様に、`right`ゴールは`right✝`仮定と似ていますが、ゴールは入力リストの長さにのみ`Nat.succ`を追加します。
これらの`Nat.succ`の包みが声明の真実を保持することを証明する時が来ました。

### 両側に一つを追加する

`left`ゴールの場合、証明するステートメントは`Nat.succ_le_succ : n ≤ m → Nat.succ n ≤ Nat.succ m`です。
言い換えると、`n ≤ m`であれば、両側に1を加えてもこの事実は変わりません。
なぜこれが真実なのでしょうか？
`n ≤ m`の証明は、`m - n`インスタンスの`Nat.le.step`コンストラクタで包まれた`Nat.le.refl`コンストラクタです。
両側に1を加えることは単に、以前より一つ大きい数字に`refl`を適用し、同じ数の`step`コンストラクタがあることを意味します。

より公式には、証明は`n ≤ m`に関する証拠に基づいて帰納法で行われます。
証拠が`refl`であれば、`n = m`なので、`Nat.succ n = Nat.succ m`であり、`refl`を再び使用できます。
証拠が`step`であれば、帰納法の仮定は`Nat.succ n ≤ Nat.succ m`に関する証拠を提供し、ゴールは`Nat.succ n ≤ Nat.succ (Nat.succ m)`を示すことです。
これは、帰納法の仮定を使用して`step`と一緒に行うことができます。

Leanでの定理のステートメントは：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean succ_le_succ0}}
```
エラーメッセージはそれを再度表明しています：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean succ_le_succ0}}
```

最初のステップは`intro`戦術を使用し、仮説`n ≤ m`をスコープに導入し、それに名前を付けることです：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean succ_le_succ1}}
```
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean succ_le_succ1}}
```

証明は`n ≤ m`の証拠に関する帰納法なので、次の戦術は`induction h`です：
```leantac
{{#example_in Examples/ProgramsProofs/Inequalities.lean succ_le_succ3}}
```
これにより、`Nat.le`の各コンストラクタに対して2つのゴールが得られます：
```output error
{{#example_out Examples/ProgramsProofs/Inequalities.lean succ_le_succ3}}
```
`refl`のゴールは`refl`を使用して自体解決できるので、`constructor`戦術が選択されます。
`step`のゴールも`step`コンストラクタの使用を必要とします：
```leantac
{{#example_in Examples/ProgramsProof