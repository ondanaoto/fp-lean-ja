```markdown
# 要約

## モナドの組み合わせ

スクラッチからモナドを書く際には、それぞれのエフェクトをモナドに追加する方法を説明する設計パターンが存在します。
リーダーエフェクトは、モナド型がリーダーの環境からの関数として追加されます。状態エフェクトは、初期状態から最終状態までの値をペアとして含む関数によって追加されます。失敗や例外は、戻り値の型に和型（union type）を加えることで追加され、ログやその他の出力は、戻り値の型に積型（product type）を含むことで追加されます。
既存のモナドも、その効果を新しいモナドに含めるために、戻り値の型の一部として活用できます。

これらの設計パターンは、_モナドトランスフォーマー_を定義することで、再使用可能なソフトウェアコンポーネントのライブラリに変えられます。モナドトランスフォーマーは基本的なモナドにエフェクトを追加します。
モナドトランスフォーマーは、より単純なモナド型を引数として取り込み、強化されたモナド型を返します。
最低限、モナドトランスフォーマーは以下のインスタンスを提供するべきです：
 1. 内部の型が既にモナドであるとする`Monad`インスタンス
 2. 内部モナドから拡張されたモナドへアクションを変換する`MonadLift`インスタンス

モナドトランスフォーマーは多相的な構造や帰納的データ型として実装されることがありますが、通常は、基盤となるモナド型から拡張されたモナド型への関数として実装されます。

## エフェクトのための型クラス

特定のエフェクトを実装する一般的なアプローチは、そのエフェクトを持つモナドを定義し、それを他のモナドに追加するためのモナドトランスフォーマーを定義し、そのエフェクトに対する汎用インターフェースを提供する型クラスを定義することです。
これにより、プログラムを必要なエフェクトだけを指定するように記述し、実行環境が適切なエフェクトを持つ任意のモナドを提供することが可能です。

場合によっては、補助的な型情報（例えば、状態を提供するモナドの状態の型や、例外を提供するモナドの例外の型など）が出力パラメータとして使用されますが、必ずしもそうではありません。
出力パラメータは、様々なエフェクトを一度に使う単純なプログラムに最も役立ちますが、同じプログラム内で同じエフェクトを複数回使う場合には、型チェッカーが早い段階で誤った型に固定するリスクがあります。
したがって、一般的なパラメータを持つ型クラスと、`-Of`で終わる型クラスの両方が通常提供されます。

## モナドトランスフォーマーは交換可能ではありません

モナドにトランスフォーマーの順序を変更すると、そのモナドを使用するプログラムの意味が変わる可能性がある点に注意が必要です。
例えば、`StateT`と`ExceptT`の順序を変えることで、例外が投げられた場合に状態変更が失われるプログラム、あるいは変更が保持されるプログラムのどちらかが生じます。
多くの命令型言語は後者のみをサポートしていますが、モナドトランスフォーマーによる柔軟性の増加は、適切な種類を選ぶための細心の注意を要求します。

## `do`表記とモナドトランスフォーマー

Leanの`do`ブロックは、早期リターンによる特定の値での終了、ローカルに変更可能な変数、`for`ループを中断・続行する`break`と`continue`、そして単一の分岐で済む`if`文をサポートします。
これがLeanを使用して証明を書く際に邪魔になるように見えるかもしれませんが、実際にはこれはモナドトランスフォーマーを一般的に使う際により便利な構文であることが多いです。
その背後では、`do`ブロックに記述されているモナドはこれらの追加エフェクトをサポートするために`ExceptT`と`StateT`を適切に利用することによって拡張されています。
```