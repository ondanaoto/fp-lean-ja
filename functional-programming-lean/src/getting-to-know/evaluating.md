# 式の評価

プログラマーとしてLeanを学ぶ際に理解すべき最も重要なことは、
評価とはどのように機能するかです。評価は、算数のように、
式の値を見つけるプロセスです。例えば、
15 - 6の値は9であり、2 × (3 + 1)の値は8です。
後者の式の値を見つけるためには、まず3 + 1が4に置き換えられ、
次に2 × 4が求められ、最終的に8に還元されます。
時には、数学的な式に変数が含まれることもあります：_x_ + 1の値は、
_x_の値がわかるまで計算することはできません。
Leanでは、プログラムはまず何よりも式であり、
計算について考える主要な方法はその式を評価してその値を見つけることです。

ほとんどのプログラミング言語は _命令型_ であり、プログラムは
プログラムの結果を見つけるために実行されるべき
一連の文で構成されています。プログラムは変更可能なメモリに
アクセスできるため、変数によって参照される値は
時間と共に変わることがあります。変更可能な状態に加えて、
プログラムはファイルの削除、外部ネットワークへの接続、
例外のスローまたはキャッチ、データベースからのデータ読み取りなど、
他の副作用を持つこともあります。"副作用"とは、実質的に、
数学的な式の評価モデルに従わないプログラムで
発生することがあるものを説明するための総称です。

しかし、Leanでは、プログラムは数学的な式のように機能します。
変数に値が与えられると、それは再割り当てされることはありません。
式の評価は副作用を持ちえません。もし二つの式が同じ値を持てば、
それらを入れ替えてもプログラムは異なる結果を計算することはありません。
これはLeanを使って`Hello, world!`をコンソールに出力することが
できないという意味ではありませんが、
I/Oを実行することはLeanを使用する際の
核となる部分ではありません。したがって、
この章はLeanと対話的に式を評価する方法に焦点を置いていますが、
次の章では`Hello, world!`プログラムを書き、
コンパイルし、そして実行する方法について説明します。

Leanに式を評価させるには、エディターで式の前に`#eval`を書きます。
すると、結果が報告されます。一般的に、結果はカーソルやマウスポインタを
`#eval`の上に置くことで見つかります。例えば、

```lean
#eval {{#example_in Examples/Intro.lean three}}
```
は値 `{{#example_out Examples/Intro.lean three}}` を出力します。

Leanは、算数演算子の通常の優先順位と結合規則に従います。つまり、

```lean
{{#example_in Examples/Intro.lean orderOfOperations}}
```
は値 `{{#example_out Examples/Intro.lean orderOfOperations}}` を出力し、
`{{#example_out Examples/Intro.lean orderOfOperationsWrong}}` ではありません。


通常の数学の記法とほとんどのプログラミング言語では、
関数をその引数に適用するために括弧を使用します（例えば、`f(x)`）が、
Leanでは単に関数をその引数の隣に書きます（例えば、`f x`）。
関数の適用は最も一般的な操作の一つなので、それを簡潔に保つことが重要です。
```Lean
#eval String.append("Hello, ", "Lean!")
```
と書いて
`{{#example_out Examples/Intro.lean stringAppendHello}}`を計算するのではなく、
代わりに次のように書きます。
``` Lean
{{#example_in Examples/Intro.lean stringAppendHello}}
```
関数の二つの引数は、単にスペースで隣に書かれています。

算数の演算子の順序の規則では、式`(1 + 2) * 5`で括弧が必要であるように、
関数の引数が他の関数呼び出しによって計算されるべきときは括弧も同様に必要です。
例えば、
``` Lean
{{#example_in Examples/Intro.lean stringAppendNested}}
```
括弧が必要なのは、そうでないと第二の`String.append`が、
最初のものの引数として解釈されるのではなく、
`"oak "`と`"tree"`を引数として渡される関数になるためです。
内側の`String.append`呼び出しの値を最初に見つけた後、
それを`"great "`に追加することができ、最終的な値
`{{#example_out Examples/Intro.lean stringAppendNested}}`が得られます。

命令型言語にはしばしば、条件に基づいてどの命令を実行するか決定する
条件文と、どちらの式を評価するかをブール値に基づいて決定する
条件式の2種類の条件があります。例えば、CおよびC++では、
条件文は`if`と`else`を使って書かれ、条件式は
三項演算子`?`と`:`を使って書かれます。Pythonでは、
条件文は`if`で始められますが、条件式では`if`が
中間に置かれます。
Leanは式指向の関数型言語なので、条件文は存在せず、
条件式のみが存在します。それらは`if`、`then`、`else`を使って書かれます。
例えば、
``` Lean
{{#example_eval Examples/Intro.lean stringAppend 0}}
```
は
``` Lean
{{#example_eval Examples/Intro.lean stringAppend 1}}
```
に評価され、それは
```lean
{{#example_eval Examples/Intro.lean stringAppend 2}}
```
に評価され、最終的には`{{#example_eval Examples/Intro.lean stringAppend 3}}`に評価されます。

簡潔にするために、このような一連の評価ステップは時には矢印で書かれることがあります：
```lean
{{#example_eval Examples/Intro.lean stringAppend}}
```

## 遭遇する可能性のあるメッセージ

引数が足りない関数の適用をLeanに評価させると、エラーメッセージが出ます。
特に、例
```lean
{{#example_in Examples/Intro.lean stringAppendReprFunction}}
```
はかなり長いエラーメッセージを出力します：
```output error
{{#example_out Examples/Intro.lean stringAppendReprFunction}}
```

このメッセージは、引数の一部だけに適用されたLean関数が、
残りの引数を待機している新しい関数を返すために発生します。
Leanは関数をユーザーに表示することはできず、そのため表示を求められたときにエラーを返します。


## 練習問題

以下の式の値は何でしょうか？手作業で計算し、
その後Leanに入力して作業を確認してください。

 * `42 + 19`
 * `String.append "A" (String.append "B" "C")`
 * `String.append (String.append "A" "B") "C"`
 * `if 3 == 3 then 5 else 7`
 * `if 3 == 4 then "equal" else "not equal"`

---