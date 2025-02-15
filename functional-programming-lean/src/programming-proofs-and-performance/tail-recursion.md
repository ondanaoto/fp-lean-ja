# 末尾再帰

Leanの`do`記法では、`for`や`while`などの伝統的なループ構文を使用することが可能ですが、これらの構成は背後で再帰関数の呼び出しに変換されます。
多くのプログラミング言語において、再帰関数はループと比較して重要な不利点を持っています：ループはスタック上のスペースを消費しないのに対して、再帰関数は再帰呼び出しの数に比例してスタック空間を消費します。
スタック空間は典型的に限られており、自然に再帰関数として表されるアルゴリズムをループに書き換え、明示的にヒープ割り当てスタックを伴うことがしばしば必要です。

関数型プログラミングでは、通常は逆のことが当てはまります。
可変ループとして自然に表されるプログラムはスタック空間を消費する場合がありますが、それらを再帰関数に書き換えると、高速に実行されることがあります。
これは関数型プログラミング言語の重要な側面である _末尾呼び出し除去_ によるものです。
末尾呼び出しとは、一つの関数から別の関数への呼び出しが、新しいスタックフレームをプッシュするのではなく、通常のジャンプにコンパイルされるものであり、末尾呼び出し除去とはこの変換を行うプロセスです。

末尾呼び出し除去は単なる任意の最適化ではありません。
その存在は効率的な関数型コードを書くことができるようにするための根本的な部分です。
これが有用であるためには、信頼できるものでなければなりません。
プログラマーは信頼できる方法で末尾呼び出しを特定し、コンパイラがそれらを除去することを信頼できると確信する必要があります。

関数`NonTail.sum`は`Nat`のリストの内容を加算します：
```lean
{{#example_decl Examples/ProgramsProofs/TCO.lean NonTailSum}}
```
この関数をリスト`[1, 2, 3]`に適用すると、次の評価ステップのシーケンスが生じます：
```lean
{{#example_eval Examples/ProgramsProofs/TCO.lean NonTailSumOneTwoThree}}
```
評価ステップにおいて、括弧は`NonTail.sum`への再帰呼び出しを示しています。
言い換えると、３つの数字を加算するために、プログラムはまずリストが空でないかを確認する必要があります。
リストの先頭（`1`）をリストの末尾の合計に加算するためには、まずリストの末尾の合計を計算する必要があります：
```lean
{{#example_eval Examples/ProgramsProofs/TCO.lean NonTailSumOneTwoThree 1}}
```
しかしリストの末尾の合計を計算するためには、プログラムはそれが空かどうかを確認する必要があります。
そしてそれは空ではありません - 末尾は自体が`2`を頭に持つリストです。
結果として待っているステップは`NonTail.sum [3]`の戻りを待つものです：
```lean
{{#example_eval Examples/ProgramsProofs/TCO.lean NonTailSumOneTwoThree 2}}
```
実行時呼び出しスタックの全ポイントは、再帰的な呼び出しの結果に値`1`、`2`、`3`を加算する指示とともに追跡されます。
再帰呼び出しが完了するにつれて、呼び出しを行ったスタックフレームに制御が戻り、加算の各ステップが実行されます。
リストの先頭を保存し、それらを加算するための指示を保存することは無料ではありません。それはリストの長さに比例するスペースが必要です。

関数`Tail.sum`も`Nat`のリストの内容を加算します：
```lean
{{#example_decl Examples/ProgramsProofs/TCO.lean TailSum}}
```
これをリスト`[1, 2, 3]`に適用すると、次の評価ステップのシーケンスが生じます：
```lean
{{#example_eval Examples/ProgramsProofs/TCO.lean TailSumOneTwoThree}}
```
内部のヘルパー関数は再帰的に自分自身を呼び出しますが、それをする際に最終結果を計算するために何も覚えておく必要がありません。
`Tail.sumHelper`が基本ケースに達したとき、制御は直接`Tail.sum`に戻されます。なぜなら、中間の`Tail.sumHelper`の呼び出しは自身の再帰呼び出しの結果を修正せずに単に返すからです。
言い換えると、各再帰的な`Tail.sumHelper`の呼び出しについて単一のスタックフレームを再利用することができます。
末尾呼び出し除去とは、まさにこのスタックフレームの再利用であり、`Tail.sumHelper`は _末尾再帰関数_ として言及されます。

`Tail.sumHelper`への最初の引数には、そうでなければ呼び出しスタックで追跡する必要がある全ての情報が含まれています。すなわち、これまでに遭遇した数字の合計です。
各再帰呼び出しでは、この引数が新しい情報で更新されます。新しい情報を呼び出しスタックに追加するのではありません。
呼び出しスタックからの情報を置き換えるような引数は _アキュムレータ_ と呼ばれます。

執筆時点で、作者のコンピューター上では、`NonTail.sum`は216,856個以上のエントリを持つリストが渡されるとスタックオーバーフローでクラッシュします。一方で、`Tail.sum`は1億個の要素を持つリストをスタックオーバーフローなしに合計することができます。
`Tail.sum`を実行する際に新しいスタックフレームをプッシュする必要がないため、それは可変変数を持つ`while`ループに完全に相当します。各再帰呼び出しでは、スタック上の関数引数が単にリストの次のノードに置き換わります。


## 末尾と非末尾の位置

`Tail.sumHelper`が末尾再帰である理由は、再帰呼び出しが _末尾位置_ にあるためです。
略式に言えば、関数呼び出しが末尾位置にあるとは、呼び出し元が返された値をどのようにも変更する必要がなく、ただ直接返すだけの場合です。
より公式には、末尾位置は式に対して明確に定義することができます。

`match`式が末尾位置にある場合、その各ブランチも末尾位置にあります。
`match`が一つのブランチを選択すると、制御は直接それに進みます。
同様に、`if`式が末尾位置にあれば、その`if`式の両方のブランチが末尾位置です。
最後に、`let`式が末尾位置にあるならば、そのボディもそうです。

その他の位置は末尾位置ではありません。
関数またはコンストラクタへの引数は末尾位置にないです。なぜなら評価は引数の値に適用される関数またはコンストラクタを追跡する必要があるからです。
内部関数のボディは末尾位置にないです。なぜなら制御がそれに通過するとは限らないからです：関数ボディは関数が呼び出されるまで評価されません。
同様に、関数型のボディも末尾位置ではありません。
`(x : α) → E`で`E`を評価するには、結果の型が`(x : α) → ...`で囲まれている必要があると追跡する必要があります。

`NonTail.sum`では、再帰呼び出しは`+`の引数であるため末尾位置にありません。
`Tail.sumHelper`では、再帰呼び出しはすぐ下にあるパターンマッチの下であり、それ自体が関数のボディであるため末尾位置にあります。

執筆時点で、Leanは再帰関数の直接的な末尾呼び出しのみを除去します。
これは、`f`の定義における`f`への末尾呼び出しが除去されることを意味しますが、他の関数`g`への末尾呼び出しは除去されません。
確かに、他の関数への末尾呼び出しを除去することも可能であり、スタックフレームを節約することもできますが、これはまだLeanでは実装されていません。

## リストの反転

関数`NonTail.reverse`は各サブリストの先頭を結果の末尾へと追加することでリストを反転させます：
```lean
{{#example_decl Examples/ProgramsProofs/TCO.lean NonTailReverse}}
```
これを使用して`[1, 2, 3]`を反転すると、次のようなステップのシーケンスになります：
```lean
{{#example_eval Examples/ProgramsProofs/TCO.lean NonTailReverseSteps}}
```

末尾再帰バージョンは、アキュムレータで各ステップで`· ++ [x]`の代わりに`x :: ·`を使用します：
```lean
{{#example_decl Examples/ProgramsProofs/TCO.lean TailReverse}}
```
これは`NonTail.reverse`で計算する際に各スタックフレームに保存されたコンテキストが基本ケースから始めて適用されるからです。
各々の「記憶された」コンテキストは後入れ先出しの順番で実行されます。
一方で、アキュムレータをパスするバージョンは、元々の基本ケースからではなく、リストの最初のエントリから開始して、初期アキュムレータ値をリストを通じて左から右へ修正します。シリーズの還元ステップにおいて見ることができるように：
```lean
{{#example_eval Examples/ProgramsProofs/TCO.lean TailReverseSteps}}
```
言い換えると、非末尾再帰バージョンは基本ケースで始まり、リストを右から左へ再帰の結果の修正を行います。
リスト中のエントリはアキュムレータに対して先入れ先出しの順番で影響を与えます。
アキュムレータを持つ末尾再帰バージョンはリストの先頭から始まり、リストを通じて左から右へ初期アキュムレータ値を修正します。

加算は交換可能なので、`Tail.sum`ではこのことを考慮に入れる必要がありませんでした。
リストの連結は交換可能ではないので、逆の方向で実行されたときに同じ効果が得られる操作を慎重に見つける必要があります。
`NonTail.reverse`で再帰の結果の後に`[x]`を連結することは、結果が逆順で構築される際に、リストの始めに`x`を追加することと同じです。

## 複数の再帰呼び出し

`BinTree.mirror`の定義には、二つの再帰呼び出しがあります：
```lean
{{#example_decl Examples/Monads/Conveniences.lean mirrorNew}}
```
`reverse`や`sum`のような関数を`while`ループで行うのが典型的な命令型言語が、「この種のトラバーサルでは再帰関数を使用します。
この関数はアキュムレータをパスするスタイルを用いて末尾再帰に単純に書き直されることはできません。

通常、各再帰的なステップに複数の再帰呼び出しが必要な場合、アキュムレータをパスするスタイルを使用することは難しいでしょう。
この難しさは、再帰関数をループに書き換え、明示的なデータ構造を使用するという難しさに似ていますが、さらにLeanに対して関数が終了することを納得させるという複雑さが加わっています。
しかしながら、`BinTree.mirror`の場合のように、複数の再帰的な呼び出しがしばしばそれ自体に複数の再帰的な発生をもつコンストラクタを持つデータ構造を示します。
これらのケースでは、構造の深さはその全体的なサイズに比べてしばしば対数的であり、スタックとヒープ間のトレードオフはそれほど厳しくありません。これらの関数を末尾再帰的にするための系統的な技法がありますが、それは継続渡しスタイルを使用することなどですが、それらはこの章の範囲を超えています。

## 練習問題

以下の非末尾再帰的な関数を累積引数を使った末尾再帰的な関数に変換してください：

```lean
{{#example_decl Examples/ProgramsProofs/TCO.lean NonTailLength}} 
```

```lean
{{#example_decl Examples/ProgramsProofs/TCO.lean NonTailFact}}
```

`NonTail.filter`の変換は、末尾再帰を通じて一定のスタック空間を使用し、入力リストの長さに比例する線形な時間で実行されるプログラムとなるべきです。
元のプログラムに対する許容可能な定数係数のオーバーヘッドが生じる場合があります：
```lean
{{#example_decl Examples/ProgramsProofs/TCO.lean NonTailFilter}}
```