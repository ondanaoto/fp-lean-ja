# doの機能強化

The content you provided seems to have placeholders for code examples (`{{#example_decl ... }}`, `{{#example_in ... }}`, `{{#example_out ... }}`, etc.) that are intended to be substituted with actual Lean code when the Markdown file is processed by a documentation tool or system. As those placeholders would fetch code from a database or a file system when processed, they aren't direct content that I can translate.

However, I can provide you with translation for the surrounding conceptual explanations and headings. Please note that any `{{...}}` placeholders will be left unchanged.

---

# Leanの`do`表記におけるさらなる特徴

Leanの`do`表記は、命令型プログラミング言語に似た構文で、モナドを使ったプログラミングを記述するための構文を提供します。
単にモナドを有効活用するための便利な構文を提供するだけではなく、特定のモナドトランスフォーマを使うための構文も`do`記法は提供しています。

## `if`の単一の分岐

モナドの中で働く際、特定の条件が真のときだけに副作用を起こすという一般的なパターンがあります。
たとえば、`countLetters`は母音または子音かどうかをチェックする処理が含まれており、どちらでもない文字は状態に影響しません。
以下の`countLettersModify`は`else`ブロックが`pure ()`を評価し、これには副作用がありません：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean countLettersModify}}
```

`do`ブロック内で式ではなくステートメントとしての`if`には、`else pure ()`を省略でき、Leanは自動的にこれを挿入します。
以下の`countLettersNoElse`の定義も完全に同等です：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean countLettersNoElse}}
```
状態モナドを使用して、モナディック条件を満たすリスト項目を数えるプログラムは以下のようになります：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean count}}
```

同様に、`if not E1 then STMT...`のかわりに`unless E1 do STMT...`と書くことができます。
条件を満たさないエントリーを数える`countNot`の逆の動作は、`if`の代わりに`unless`を使うことで実現できます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean countNot}}
```

単一の分岐`if`と`unless`を理解するには、モナドトランスフォーマについて考える必要はありません。
それらは単に欠けているブランチを`pure ()`で置き換えるだけです。
しかし、このセクションに残されている拡張機能はLeanが`do`ブロックを自動的に書き換え、そこに書かれたモナドの上に局所トランスフォーマーを追加するようになっています。

## 早期リターン

標準ライブラリには、ある条件を満たすリストの最初のエントリを返す`List.find?`という関数が含まれています。
この`Option`モナドを活用しない単純な実装は、リカーシブ(再帰的)関数を使用し、望ましいエントリが見つかったときにループを停止するために`if`を使います：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean findHuhSimple}}
```

命令型言語で、通常`return`キーワードは関数の実行を中断し、何らかの値を直ちに返すために使用されます。
Leanでは、`do`表記の中で`return`を使うことができ、`return`で`do`ブロックの実行を止めることができ、`return`の引数がモナドから返される値となります。
つまり、`List.find?`は以下のように書けたかもしれません：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean findHuhFancy}}
```

命令型言語の早期リターンは、現在のスタックフレームだけをアンワインドすることが可能な例外類似です。
早期リターンと例外はどちらもコードブロックの実行を停止させ、周囲のコードを投げられた値で置き換えます。
裏側では、Leanにおける早期リターンは`ExceptT`のバージョンを使用して実装されています。
早期リターンを使う各`do`ブロックは、関数`tryCatch`の意味の例外ハンドラーでラップされます。
早期リターンは例外として値を投げ、ハンドラは投げられた値をキャッチし、直ちに返します。
つまり、`do`ブロックの元の戻り値型は、例外型としても使われます。

より具体的にするために、ヘルパー関数`runCatch`は`ExceptT`の層をモナドトランスフォーマースタックの最上部から取り除きます。これは例外タイプと戻り値タイプが同じ場合に使用されます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean runCatch}}
```
早期リターンを使う`List.find?`の`do`ブロックは、`runCatch`を使用してラップされ、早期リターンを`throw`に置き換えることで、早期リターンを使用しない`do`ブロックに変換されます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean desugaredFindHuh}}
```

早期リターンが有用な別のシナリオは、引数や入力が不正な場合にコマンドラインアプリケーションを早期終了させる場合です。
多くのプログラムは、プログラムの本体に入る前に、引数や入力を検証するセクションから始まります。
以下の挨拶プログラム`hello-name`のバージョンはコマンドライン引数が提供されていないことをチェックします：
```lean
{{#include ../../../examples/early-return/EarlyReturn.lean:main}}
```
引数なしで実行し、名前`David`を入力すると、以前のバージョンと同じ結果が得られます。

名前をコマンドライン引数として提供するとエラーが発生します：

そして名前を提供しない場合は別のエラーが発生します：

早期リターンを使用するプログラムは、このような制御フローのネストを避けます。早期リターンを使用しないバージョンで行われているようにです：
```lean
{{#include ../../../examples/early-return/EarlyReturn.lean:nestedmain}}
```

Leanの早期リターンと命令型言語の早期リターンの重要な違いの一つは、Leanの早期リターンは現在の`do`ブロックにのみ影響を与えるということです。
関数の完全な定義が同じ`do`ブロックの内にある場合は、この違いはそれほど重要ではありません。
しかし、`do`が他の構造の下にある場合、その差異は明確になります。
例として、以下の`greet`の定義を考えましょう：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean greet}}
```
この式`{{#example_in Examples/MonadTransformers/Do.lean greetDavid}}`は`{{#example_out Examples/MonadTransformers/Do.lean greetDavid}}`として評価されます。注意しましょう、これは`"David"`ではありません。

## ループ

すべての書き換え可能な状態を持つプログラムは、状態を引数として渡すプログラムに置き換えることができるのと同様に、すべてのループも再帰関数に置き換えることができます。
一方で、`List.find?`は再帰関数として一番明確です。結局のところ、定義はリストの構造を反映しています：先頭がチェックを通過するならばそれを返し、そうでなければ尾部を検討します。エントリがなくなった場合、答えは`none`です。
別の視点からすると、`List.find?`はループとしてもっとも明確です。このプログラムは満足のいくエントリが見つかるまで、順にエントリを調べます。ループがなにも返さずに終わる場合、答えは`none`です。

### ForMを使用したループ

Leanには、特定のモナドの中でコンテナ型をループするための型クラスが含まれています。
これは`ForM`と呼ばれています：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean ForM}}
```
このクラスはとても一般的です。パラメータ`m`はいくつかのモナドであり求められる効果を持ち、`γ`はループされるコレクションで、`α`はコレクションからの要素の型です。
通常`m`はどのモナドでも良いですが、例えば`IO`でのみループをサポートするデータ構造を持つこともできます。
メソッド`forM`はコレクションと、コレクションの各要素に対してその効果のために実行されるモナドアクションをとり、アクションを実行します。

`List`のインスタンスも任意のモナドに対して`m`を許し、`γ`を`List α`に設定し、クラスの`α`はリストに見られる同じ`α`に設定します：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean ListForM}}
```
`doug`からの`doList`関数はリストに対する`forM`です。
`forM`は`do`ブロックで使われることを意図しているため、`Applicative`よりも`Monad`が使われます。
`forM`を使えば`countLetters`をかなり短くすることができます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean countLettersForM}}
```

`Many`のインスタンスも非常に似ています：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean ManyForM}}
```

`γ`は何でもよいので、`ForM`は非多様なコレクションもサポートすることができます。
特にシンプルなコレクションの一つは、反対バージョンの特定の数値よりも小さい自然数です：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean AllLessThan}}
```
その`forM`演算子は指定されたアクションをそれぞれの小さな`Nat`に適用します：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean AllLessThanForM}}
```
5より小さい数に`IO.println`を実行するのは`forM`を使って達成されます：
```lean
{{#example_in Examples/MonadTransformers/Do.lean AllLessThanForMRun}}
```
```output info
{{#example_out Examples/MonadTransformers/Do.lean AllLessThanForMRun}}
```

特定のモナドにのみ動作する`ForM`インスタンスの例は、例えば標準入力などのIOストリームから読まれた行をループするものです：
```lean
{{#include ../../../examples/formio/ForMIO.lean:LinesOf}}
```
`forM`の定義は、ストリームが有限でない可能性があるため`partial`としてマークされています。
この場合、`IO.FS.Stream.getLine`は`IO`モナドのみで動作するため、他のモナドではループすることができません。

この例のプログラムでは、人を除いた行をフィルタリングするためにこのループ構造が使われています：
```lean
{{#include ../../../examples/formio/ForMIO.lean:main}}
```
ファイル`test-data`には以下の内容が含まれています：

このプログラムを実行すると、`ForMIO.lean`に保存されると次の出力が得られます：

### 反復を停止する

`forM`で早めにループを止めるのは大変な場合があります。
`AllLessThan`と`Nat`をループし、「3」に到達したら反復をやめるだけの関数を書く場合、途中でループを止められる手段が必要です。
この目的を達成する1つの方法は、`OptionT`モナドトランスフォーマを`forM`と一緒に使うことです。
最初のステップは、`OptionT.exec`を定義することであり、これは変換された計算に基づく戻り値と成功したかどうかの情報の両方を破棄します：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean OptionTExec}}
```
次に、`OptionT`