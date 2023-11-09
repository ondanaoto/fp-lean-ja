# IOとReaderの組み合わせ

あるケースでリーダーモナドが便利とされるのは、アプリケーションの「現在の設定」の概念が多くの再帰呼び出しに渡される場合です。
このようなプログラムの例は、現在のディレクトリとそのサブディレクトリのファイルを再帰的に出力し、そのツリー構造を文字で示す`tree`です。
この章の`tree`バージョンは、北米西海岸を飾る偉大なダグラスファーの木にちなんで`doug`と呼ばれ、ディレクトリ構造を示すときにユニコードの枠線文字またはそのASCII同等物の選択肢を提供します。

たとえば、以下のコマンドは`doug-demo`というディレクトリにディレクトリ構造といくつかの空ファイルを作成します：
```
$ cd doug-demo
$ {{#command {doug-demo} {doug} {mkdir -p a/b/c} }}
$ {{#command {doug-demo} {doug} {mkdir -p a/d} }}
$ {{#command {doug-demo} {doug} {mkdir -p a/e/f} }}
$ {{#command {doug-demo} {doug} {touch a/b/hello} }}
$ {{#command {doug-demo} {doug} {touch a/d/another-file} }}
$ {{#command {doug-demo} {doug} {touch a/e/still-another-file-again} }}
```
`doug`を実行すると、以下のようになります：
```
$ {{#command {doug-demo} {doug} {doug} }}
{{#command_out {doug} {doug} }}
```

## 実装

内部的に、`doug`は設定値をディレクトリ構造を再帰的にトラバースする際に下方向へ渡します。
この設定には二つのフィールドが含まれます：`useASCII`はユニコードの枠線文字を使うかASCII縦線やダッシュ文字を使うかを決定し、`currentPrefix`は出力の各行に前置する文字列を含みます。
現在のディレクトリが深くなるにつれて、プレフィックス文字列はディレクトリ内にあることを示すマーカーが蓄積されます。
設定は以下の構造です：
```lean
{{#example_decl Examples/MonadTransformers.lean Config}}
```
この構造は両方のフィールドにデフォルト定義を持っています。
デフォルトの`Config`はプレフィックスなしでユニコード表示を使用します。

`doug`を呼び出すユーザーはコマンドライン引数を提供する必要があります。
使用情報は以下の通りです：
```lean
{{#example_decl Examples/MonadTransformers.lean usage}}
```
それに応じて、コマンドライン引数のリストを検査して設定を構築することができます：
```lean
{{#example_decl Examples/MonadTransformers.lean configFromArgs}}
```

`main`関数は内部ワーカーである`dirTree`をラップするもので、設定を使用してディレクトリの内容を表示します。
`dirTree`を呼び出す前に、`main`はコマンドライン引数を処理する責任を持ちます。
また、適切な終了コードをオペレーティングシステムに返す必要があります：
```lean
{{#example_decl Examples/MonadTransformers.lean OldMain}}
```

ディレクトリツリーにすべてのパスを表示すべきではありません。
特に、`.`または`..`という名前のファイルはスキップされるべきです。それらは実際にはナビゲーションに使用される機能であり、それ自体がファイルではありません。
表示すべきファイルには二つのタイプがあります：通常のファイルとディレクトリです：
```lean
{{#example_decl Examples/MonadTransformers.lean Entry}}
```
ファイルを表示すべきかどうかを決定し、どの種類のエントリであるか、`doug`は`toEntry`を使用します：
```lean
{{#example_decl Examples/MonadTransformers.lean toEntry}}
```
`System.FilePath.components`はパスをコンポーネントのリストに変換し、ディレクトリセパレータで名前を分割します。
最後のコンポーネントがない場合、パスはルートディレクトリです。
最後のコンポーネントが特別なナビゲーションファイル(`.`または`..`)である場合、そのファイルは除外されるべきです。
それ以外の場合、ディレクトリとファイルはそれぞれのコンストラクタでラップされます。

Leanの論理は、ディレクトリツリーが有限であると知る方法がありません。
実際、いくつかのシステムは循環的なディレクトリ構造の構築を許可しています。
したがって、`dirTree`は`partial`と宣言されます：
```lean
{{#example_decl Examples/MonadTransformers.lean OldDirTree}}
```
`toEntry`への呼び出しは[ネストされたアクション](../hello-world/conveniences.md#nested-actions)です—カッコは矢印に他の意味を持たせることができない位置で省略可能です、例えば`match`。
ファイル名がツリー内のエントリに該当しない場合（例えば、`..`のため）、`dirTree`は何もしません。
ファイル名が通常のファイルを指している場合、`dirTree`は現在の設定とともに表示するヘルパーを呼び出します。
ディレクトリを指している場合、ヘルパーで表示され、その後その内容は新しいディレクトリ内にいることを考慮した拡張されたプレフィックスで新しい設定で再帰的に表示されます。

ファイルとディレクトリの名前を表示することは、`showFileName`と`showDirName`とともに達成されます：
```lean
{{#example_decl Examples/MonadTransformers.lean OldShowFile}}
```
これら両方のヘルパーはASCII対ユニコードの設定を考慮に入れる`Config`上の関数に委任しています：
```lean
{{#example_decl Examples/MonadTransformers.lean filenames}}
```
同様に、`Config.inDirectory`はディレクトリマーカーでプレフィックスを拡張します：
```lean
{{#example_decl Examples/MonadTransformers.lean inDirectory}}
```

ディレクトリ内容のリストに対しIOアクションを繰り返すことは`doList`を使用して達成されます。
`doList`はリスト内の全アクションを実行し、いかなるアクションから返された値に基づいて制御フローの決定をしませんので、`Monad`の全能力は必要なく、任意の`Applicative`に対して動作します：
```lean
{{#example_decl Examples/MonadTransformers.lean doList}}
```

## カスタムモナドの使用

この`doug`の実装は機能しますが、設定を手動で渡すことは冗長でエラーが発生しやすいものです。
タイプシステムでは、誤った設定が下方向に渡されても捉えられません。
リーダー効果は同じ設定をすべての再帰呼び出しに渡すことを保証し、コードを簡潔にするのに役立ちます。

`IO`のバージョンを作成することも`Config`のリーダーですが、最初にタイプとその`Monad`インスタンスを定義します。[評価者の例](../monads/arithmetic.md#custom-environments)からのレシピに従ってください：
```lean
{{#example_decl Examples/MonadTransformers.lean ConfigIO}}
```
この`Monad`インスタンスと`Reader`のためのものの違いは、`bind`が返す関数の本体として`IO`モナドの`do`記法を使うことです、`next`を直接`result`から返された値に適用する代わりに。
`result`によって実行される任意の`IO`効果は`next`が呼び出される前に発生しなくてはなりません。これは`IO`モナドの`bind`オペレータによって保証されます。
`ConfigIO`は基盤となる`IO`タイプもユニバース多型ではないため、ユニバース多型ではありません。

`ConfigIO`アクションを実行することは、それに設定を提供することによって`IO`アクションに変換することを含みます：
```lean
{{#example_decl Examples/MonadTransformers.lean ConfigIORun}}
```
この関数は本当に必要ありません。なぜなら呼び出し側は単純に設定を直接提供することもできるからです。
しかし、操作に名前を付けることで、コードのどの部分がどのモナドで実行されることを意図しているかを理解しやすくすることができます。

次のステップは、現在の設定へのアクセス手段を`ConfigIO`の一部として定義することです：
```lean
{{#example_decl Examples/MonadTransformers.lean currentConfig}}
```
これは[評価者の例](../monads/arithmetic.md#custom-environments)の`read`と同じですが、それの値を直接返すのではなく、`IO`の`pure`を使用して返します。
ディレクトリに入ることは、再帰呼び出しのスコープのために現在の設定を変更するので、設定を上書きする方法が必要となります：
```lean
{{#example_decl Examples/MonadTransformers.lean locally}}
```

`doug`で使用されるコードの多くは設定を必要としません。`doug`は標準ライブラリから通常のLean `IO`アクションを呼び出しますが、それらは確かに`Config`を必要としません。
通常の`IO`アクションは`runIO`を使用して実行できます。これは設定引数を無視します：
```lean
{{#example_decl Examples/MonadTransformers.lean runIO}}
```

これらのコンポーネントを持っていれば、`showFileName`と`showDirName`は`ConfigIO`モナドを通じて暗黙的に設定引数を取ります。
それらは設定を取得するために[ネストされたアクション](../hello-world/conveniences.md#nested-actions)を使用し、`runIO`を使用して`IO.println`への呼び出しを実行します：
```lean
{{#example_decl Examples/MonadTransformers.lean MedShowFileDir}}
```

新バージョンの`dirTree`では、`toEntry`と`System.FilePath.readDir`への呼び出しは`runIO`でラップされます。
また、新しい設定を作成して再帰呼び出しにプログラマがどれを渡すかを追跡する代わりに、それは`locally`を使用して変更された設定をプログラムの小さな領域に自然に限定し、それが唯一有効な設定となります：
```lean
{{#example_decl Examples/MonadTransformers.lean MedDirTree}}
```

新バージョンの`main`は`ConfigIO.run`を使用して初期設定とともに`dirTree`を呼び出します：
```lean
{{#example_decl Examples/MonadTransformers.lean MedMain}}
```

このカスタムモナドは設定を手動で渡すことに対して多くの利点があります：

 1. 変更を望む場合を除いて、設定が変更されずに渡されることを保証する方が簡単です
 2. 設定を次に渡すという懸念が、ディレクトリ内容を印刷する懸念からより明確に分離されます
 3. プログラムが拡大するにつれて、設定の論理を変更してもこれらの中間層を書き換える必要はなくなります

ただし、明らかなデメリットもあります：

 1. プログラムが進化しモナドがより多くの機能を必要とするにつれて、`locally`や`currentConfig`などの基本オペレーターを更新する必要があります
 2. 通常の`IO`アクションを`runIO`にラップすることは騒々しく、プログラムの流れから気を散らせます
 3. 手作業によるモナドインスタンスの作成は繰り返しであり、リーダー効果を別のモナドに追加するためのテクニックは、文書化とコミュニケーションのオーバーヘッドを必要とするデザインパターンです

_モナド変換器_と呼ばれるテクニックを使用することで、これらのデメリットに対処できます。
モナド変換器はモナドを引数として取り、新しいモナドを返します。
モナド変換器は以下を含みます：
 1. 変換器自体の定義であり、通常はタイプからタイプへの関数です
 2. 内部タイプが既にモナドであると仮定する`Monad`インスタンス
