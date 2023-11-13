# ステップ・バイ・ステップ

`do` ブロックは、1行ずつ実行することができます。
前のセクションからプログラムを始めましょう：
```lean
{{#include ../../../examples/hello-name/HelloName.lean:block1}}
```

## 標準IO

最初の行は `{{#include ../../../examples/hello-name/HelloName.lean:line1}}` ですが、残りは以下の通りです：
```lean
{{#include ../../../examples/hello-name/HelloName.lean:block2}}
```
`←` を使用している `let` 文を実行するには、まず矢印の右側にある表現（この場合は `IO.getStdIn`）を評価します。
この表現は単なる変数なので、その値は検索されます。
結果として得られる値は組み込みの基本的な `IO` アクションです。
次のステップは、この `IO` アクションを実行し、標準入力ストリームを表す値を得ることです。このタイプは `IO.FS.Stream` です。
標準入力は、矢印の左にある名前（ここでは `stdin`）に関連付けられ、`do` ブロックの残りの間使用されます。

二番目の行 `{{#include ../../../examples/hello-name/HelloName.lean:line2}}` の実行も同様に進みます。
まず、表現 `IO.getStdout` が評価され、標準出力を返す `IO` アクションが生成されます。
次に、このアクションが実行され、実際に標準出力が返されます。
最後に、この値は `do` ブロックの残りの間に名前 `stdout` に関連付けられます。

## 質問をする

`stdin` と `stdout` が見つかったので、ブロックの残りは質問とその答えから成り立っています：
```lean
{{#include ../../../examples/hello-name/HelloName.lean:block3}}
```

ブロック内の最初の文 `{{#include ../../../examples/hello-name/HelloName.lean:line3}}` は表現から成っています。
表現を実行するには、まず評価されます。
この場合、`IO.FS.Stream.putStrLn` は `IO.FS.Stream → String → IO Unit` という型を持っています。
これは、ストリームと文字列を受け取り、`IO` アクションを返す関数です。
この表現は関数呼び出しに[アクセッサ記法](../getting-to-know/structures.md#behind-the-scenes)を使用しています。
この関数は二つの引数、標準出力ストリームと文字列を受け取って適用されます。
表現の値は、文字列と改行文字を出力ストリームに書き込む `IO` アクションです。
この値が見つかった次のステップは、それを実行し、文字列と改行が実際に `stdout` に書き込まれることです。
表現のみから成る文では、新しい変数は導入されません。

次の文 `{{#include ../../../examples/hello-name/HelloName.lean:line4}}` では、`IO.FS.Stream.getLine` は `IO.FS.Stream → IO String` という型を持っており、これはストリームから文字列を返す `IO` アクションへの関数を意味します。
これもアクセッサ記法の例です。
この `IO` アクションが実行され、プログラムはユーザーが入力行を完了するまで待ちます。
ユーザーが「`David`」と書いたと仮定します。
結果として得られた行（`"David\n"`）は `input` と関連付けられ、エスケープシーケンス `\n` は改行を示します。

```lean
{{#include ../../../examples/hello-name/HelloName.lean:block5}}
```

次の行 `{{#include ../../../examples/hello-name/HelloName.lean:line5}}` は `let` 文です。
このプログラムの他の `let` 文とは異なり、`:=` を使用して `←` ではありません。
これは表現が評価されることを意味し、結果として得られる値は `IO` アクションである必要はなく、実行されません。
このケースでは、`String.dropRightWhile` は文字列と文字に対する述語を取り、述語を満たす文字がすべて取り除かれた新しい文字列を返します。
たとえば、
```lean
{{#example_in Examples/HelloWorld.lean dropBang}}
```
は
```output info
{{#example_out Examples/HelloWorld.lean dropBang}}
```
を生成し、
```lean
{{#example_in Examples/HelloWorld.lean dropNonLetter}}
```
は
```output info
{{#example_out Examples/HelloWorld.lean dropNonLetter}}
```
を生成します。この例では、文字列の右側からすべての非英数字文字が取り除かれます。
現在のプログラムの行では、空白文字（改行を含む）が入力文字列の右側から取り除かれ、「"David"」が結果として得られ、ブロックの残りの間 `name` と関連付けられます。


## ユーザーに挨拶する

`do` ブロックで実行される残りの文は、単一の文です：
```lean
{{#include ../../../examples/hello-name/HelloName.lean:line6}}
```
`putStrLn` への文字列引数は文字列補間を介して構築され、「"Hello, David!"」という文字列が生成されます。
この文が表現であるため、それは改行付きの標準出力にこの文字列を印刷する `IO` アクションを生成するための評価されます。
表現が評価された後、結果として得られる `IO` アクションが実行され、挨拶する結果になります。

## `IO` アクションの値として

上記の説明では、表現を評価すると「IO」アクションを実行するという区別が必要な理由を見るのが難しい可能性があります。
結局のところ、各アクションは生成された直後に実行されます。
他の言語で行われているように、評価中に効果を単に持ち出すのではないでしょうか？

答えは二つあります。
まず第一に、評価と実行を分けることで、プログラムはどの関数が副作用を持つことができるかについて明確にする必要があります。
プログラムの効果がない部分は、プログラマーの頭の中であれリーンのフォーマルプルーフの機能を使用している場合であれ、数学的推論により適しているため、この分離はバグを避けるのに役立つ可能性があります。
第二に、全ての `IO` アクションが存在する時に実行される必要はありません。
言及されたアクションを実行せずに済む機能により、普通の関数が制御構造として使用されます。

たとえば、関数 `twice` は `IO` アクションを引数としてとり、最初のものを2回実行する新しいアクションを返します。
```lean
{{#example_decl Examples/HelloWorld.lean twice}}
```
たとえば、
```lean
{{#example_in Examples/HelloWorld.lean twiceShy}}
```
を実行すると
```output info
{{#example_out Examples/HelloWorld.lean twiceShy}}
```
が印刷されます。
これは、基礎となるアクションを任意の回数実行するバージョンに一般化することができます：
```lean
{{#example_decl Examples/HelloWorld.lean nTimes}}
```
`Nat.zero` の基本ケースでは、結果は `pure ()` です。
関数 `pure` は副作用がない `IO` アクションを作成しますが、この場合は `pure` の引数、つまり `Unit` のコンストラクタを返します。
何もしないアクションとして何も興味深いものを返さない `pure ()` は、同時に完全につまらないと同時に非常に役立つものです。
再帰ステップでは、`do` ブロックを使用して、最初に `action` を実行し、次に再帰呼び出しの結果を実行するアクションを作成します。
`{{#example_in Examples/HelloWorld.lean nTimes3}}` の実行は、次の出力を引き起こします:
```output info
{{#example_out Examples/HelloWorld.lean nTimes3}}
```

関数を制御構造として使用することに加えて、`IO` アクションが第一級の値である事実は、それらが後で実行するためにデータ構造に保存されることを意味します。
たとえば、関数 `countdown` は `Nat` を取り、各 `Nat` に対する未実行の `IO` アクションのリストを返します：
```lean
{{#example_decl Examples/HelloWorld.lean countdown}}
```
この関数には副作用がなく、何も印刷しません。
例えば、それは引数に適用され、結果のアクションリストの長さをチェックすることができます：
```lean
{{#example_decl Examples/HelloWorld.lean from5}}
```
このリストには6つの要素（各数字に1つずつ、そしてゼロのために「"Blast off!"」アクション）が含まれています：
```lean
{{#example_in Examples/HelloWorld.lean from5length}}
```
```output info
{{#example_out Examples/HelloWorld.lean from5length}}
```

関数 `runActions` はアクションのリストを取り、それらをすべて順番に実行する単一のアクションを構築します：
```lean
{{#example_decl Examples/HelloWorld.lean runActions}}
```
その構造は `nTimes` のものと実質的に同じですが、`Nat.succ` の各々に実行されるのは1つのアクションではなく、`List.cons` の下にあるアクションが実行されることです。
同様に、`runActions` 自体はアクションを実行しません。
それはアクションを実行する新しいアクションを作成し、そのアクションは `main` の一部として実行される位置に置かれる必要があります：
```lean
{{#example_decl Examples/HelloWorld.lean main}}
```
このプログラムを実行すると、次の出力が得られます：
```output info
{{#example_out Examples/HelloWorld.lean countdown5}}
```

このプログラムが実行されると、何が起こるでしょうか？
最初のステップは `main` を評価することです。それは次のようになります：
```lean
{{#example_eval Examples/HelloWorld.lean evalMain}}
```
結果として得られる `IO` アクションは `do` ブロックです。
`do` ブロックの各ステップは、次々に実行され、期待される出力が得られます。
最後のステップ、`pure ()` には効果がなく、`runActions` の定義に基本ケースが必要だからだけに存在します。

## 練習問題

次のプログラムの実行を紙に書き出して、ステップごとにたどりましょう：
```lean
{{#example_decl Examples/HelloWorld.lean ExMain}}
```
プログラムの実行を追跡している間に、表現が評価されている時と `IO` アクションが実行されている時を識別します。
`IO` アクションの実行が副作用を引き起こす場合は、それを書き留めます。
これを行った後、Leanでプログラムを実行し、副作用に関してあなたの予測が正しかったかを再確認してください。