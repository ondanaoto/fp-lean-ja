# より多くの `do` 特徴

Leanの`do`記法は、命令型プログラミング言語に似た構文でモナドを使用したプログラミングを記述するための構文を提供します。
モナドを用いたプログラムを便利に記述するための構文を提供するだけでなく、`do`記法は特定のモナドトランスフォーマーを使用するための構文も提供しています。

## 単一分岐の `if`

モナドの中で働く際に、ある条件が真であるならば副作用を行うという一般的なパターンがあります。
例えば、`countLetters`には母音や子音をチェックする処理が含まれており、どちらでもない文字は状態に影響を与えません。
これは`else`ブランチが`pure ()`を評価することによって捉えられ、これには副作用がありません：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean countLettersModify}}
```

`do`ブロック内の`if`が式ではなくステートメントの場合、`else pure ()`は省略可能であり、Leanはこれを自動的に挿入します。
以下の`countLetters`の定義は全く同等です：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean countLettersNoElse}}
```
モナディックチェックを満たすリスト内のエントリを数えるために州モナドを使用するプログラムは次のように記述されます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean count}}
```

同様に、`if not E1 then STMT...`の代わりに`unless E1 do STMT...`と書くことができます。
モナディックチェックを満たさないエントリを数える`count`の逆のものは、`if`を`unless`に置き換えることで書くことができます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean countNot}}
```

単一分岐の`if`と`unless`を理解するためには、モナドトランスフォーマーについて考える必要はありません。
それらは単純に欠けているブランチを`pure ()`で置き換えます。
しかし、このセクションの残りの拡張は、Leanが`do`ブロックを自動的に書き換えて、その中で書かれているモナドの上にローカルトランスフォーマーを追加する必要があります。

## 早期リターン

標準ライブラリには、あるチェックを満たすリストの最初のエントリを返す`List.find?`関数が含まれています。
`Option`がモナドであることを利用しない単純な実装は、リストをリカーサイブ関数を使用してループし、望ましいエントリが見つかったときにループを止めるために`if`を使用します：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean findHuhSimple}}
```

命令型言語では、`return`キーワードは通常関数の実行を中断し、呼び出し元に何らかの値を直ちに返すために使われます。
Leanでは、`do`記法の中で`return`が利用可能で、`return`は`do`ブロックの実行を停止し、`return`の引数がモナドから返される値です。
言い換えると、`List.find?`は次のように書くことができたでしょう：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean findHuhFancy}}
```

命令型言語における早期リターンは、現在のスタックフレームのみをアンワインド（巻き戻し）することができる例外のようなものです。
早期リターンと例外の両方がコードのブロックの実行を終了させ、周囲のコードを投げられた値で置き換えます。
裏では、Leanにおける早期リターンは`ExceptT`のバージョンを使用して実装されています。
早期リターンを使用する各`do`ブロックは、関数`tryCatch`の意味において、例外ハンドラーでラップされます。
早期リターンは例外として値を投げることに変換され、ハンドラーは投げられた値を捕捉し、直ちにそれを返します。
言い換えると、`do`ブロックの元の戻り値タイプは、例外タイプとしても使用されます。

これをより具体的にするために、ヘルパー関数`runCatch`は`ExceptT`の層をモナドトランスフォーマースタックの頂上から剥がし、例外タイプと戻り値タイプが同じ場合に使われます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean runCatch}}
```
早期リターンを使用する`List.find?`の`do`ブロックは、`runCatch`を使用してラップし、早期リターンを`throw`に置き換えることによって、早期リターンを使用しない`do`ブロックに変換されます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean desugaredFindHuh}}
```

早期リターンが役立つ別の状況は、引数や入力が正しくない場合に早期終了するコマンドラインアプリケーションです。
多くのプログラムは、プログラムの主体に進む前に引数と入力を検証するセクションから始まります。
以下の[挨拶プログラム`hello-name`](../hello-world/running-a-program.md)のバージョンはコマンドライン引数が提供されていなかったことをチェックします：
```lean
{{#include ../../../examples/early-return/EarlyReturn.lean:main}}
```
引数なしで実行し、名前`David`を入力すると、以前のバージョンと同じ結果が得られます：
```
$ {{#command {early-return} {early-return} {./run} {lean --run EarlyReturn.lean}}}
{{#command_out {early-return} {./run} }}
```

代わりにコマンドライン引数として名前を提供するとエラーが発生します：
```
$ {{#command {early-return} {early-return} {./too-many-args} {lean --run EarlyReturn.lean David}}}
{{#command_out {early-return} {./too-many-args} }}
```

そして名前を提供しない場合は他のエラーが発生します：
```
$ {{#command {early-return} {early-return} {./no-name} {lean --run EarlyReturn.lean}}}
{{#command_out {early-return} {./no-name} }}
```

早期リターンを使用するプログラムは、早期リターンを使用しないこのバージョンで行われているような制御フローのネストを回避します：
```lean
{{#include ../../../examples/early-return/EarlyReturn.lean:nestedmain}}
```

Leanの早期リターンと命令型言語の早期リターンの重要な違いの一つは、Leanの早期リターンは現在の`do`ブロックにのみ適用されるということです。
関数の全定義が同じ`do`ブロックの中にある場合、この違いは重要ではありません。
しかし、`do`が他の構造の下に存在する場合、違いは明白になります。
例として、以下のような`greet`の定義を考えてみましょう：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean greet}}
```
この式`{{#example_in Examples/MonadTransformers/Do.lean greetDavid}}`は`{{#example_out Examples/MonadTransformers/Do.lean greetDavid}}`として評価されます。ちなみに`"David"`ではないです。

## ループ

すべての書き換え可能な状態を伴うプログラムは、状態を引数として渡すプログラムに書き換えることができるように、すべてのループも再帰関数に書き換えることができます。
ある視点からは、`List.find?`は再帰関数として最も明確です。
結局のところ、その定義はリストの構造を映し出しています：もし先頭がチェックを通過するならばそれを返すべきで、そうでなければ尾部で見ます。
エントリがなくなったときに、答えは`none`です。
別の視点からは、`List.find?`はループとして最も明確です。
結局のところ、プログラムは満足なエントリが見つかるまで順番にエントリを参照します。その時点で終了します。
ループが返すことなく終了した場合、答えは`none`です。

### ForMを使用したループ

Leanには、一部のモナドの中でコンテナタイプをループすることを記述する型クラスが含まれています。
このクラスは`ForM`と呼ばれています：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean ForM}}
```
このクラスは非常に一般的です。
パラメーター`m`は求められる効果を持ついくつかのモナドで、`γ`はループされるコレクションで、`α`はコレクションからの要素のタイプです。
通常、`m`は任意のモナドになることが許されますが、例えば、ループを`IO`でのみサポートするデータ構造を持つことが可能です。
メソッド`forM`はコレクションと、コレクションの各要素に対してその効果のために実行されるモナドアクションをとり、次いでアクションを実行します。

`List`のインスタンスは任意のモナドを`m`に許可し、`γ`を`List α`に設定し、クラスの`α`をリストに見られる同じ`α`に設定します：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean ListForM}}
```
`doug`からの関数`doList`はリストのための`forM`です。
`forM`は`do`ブロックで使われることを意図しているため、`Applicative`ではなく`Monad`が使用されます。
`forM`を使用すると`countLetters`をとても短くすることができます：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean countLettersForM}}
```


`Many`のインスタンスも非常に似ています：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean ManyForM}}
```

`γ`はどのようなタイプでもよいので、`ForM`は非多相的なコレクションもサポートできます。
非常に単純なコレクションの一つは、逆順の特定の数値より小さい自然数です：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean AllLessThan}}
```
その`forM`演算子は提供されたアクションを各より小さい`Nat`に適用します：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean AllLessThanForM}}
```
五より小さい各数に`IO.println`を実行することは`forM`を使って完成されます：
```lean
{{#example_in Examples/MonadTransformers/Do.lean AllLessThanForMRun}}
```
```output info
{{#example_out Examples/MonadTransformers/Do.lean AllLessThanForMRun}}
```

特定のモナドでのみ動作する`ForM`インスタンスの例は、標準入力などのIOストリームから読み取られた行をループするものです：
```lean
{{#include ../../../examples/formio/ForMIO.lean:LinesOf}}
```
`forM`の定義は、ストリームが有限であるとは限らないため`partial`としてマークされています。
この場合、`IO.FS.Stream.getLine`は`IO`モナドでのみ動作するため、ループに他のモナドは使用できません。

この例のプログラムは、文字を含まない行をフィルタリングするためにこのループ構造を使用します：
```lean
{{#include ../../../examples/formio/ForMIO.lean:main}}
```
ファイル`test-data`は以下を含みます：
```
{{#include ../../../examples/formio/test-data}}
```
このプログラムを呼び出し、`ForMIO.lean`に保存されていると、次の出力が得られます：
```
$ {{#command {formio} {formio} {lean --run ForMIO.lean < test-data}}}
{{#command_out {formio} {lean --run ForMIO.lean < test-data} {formio/expected}}}
```

### 反復を停止する

`forM`で早期にループを終了することは難しいです。
`AllLessThan`の`Nat`をループして`3`に到達するまでに反復するだけの関数を書くには、途中でループを止める手段が必要です。
これを実現する1つの方法は、`OptionT`モナドトランスフォーマーを`forM`とともに使うことです。
最初のステップは、`OptionT.exec`を定義することであり、これは変換された計算に関する戻り値と成功したかどうかの情報の両方を破棄します：
```lean
{{#example_decl Examples/MonadTransformers/Do.lean OptionTExec}}
```
次に、`OptionT