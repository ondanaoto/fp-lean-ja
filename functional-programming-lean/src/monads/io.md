# IOモナド

`IO`をモナドとして理解するには、[プログラムの実行](../hello-world/running-a-program.md)セクションで説明された二つの視点があります。これらはそれぞれ`pure`および`bind`の`IO`での意味を理解するのに役立ちます。

一つ目の視点では、`IO`アクションはLeanのランタイムシステムへの命令として理解できます。例えば、その命令は「このファイルディスクリプタから文字列を読み取り、その文字列を使って純粋なLeanコードを再実行する」といったものになるかもしれません。この視点は_外部的_なもので、オペレーティングシステムの視点からプログラムを見るものです。この場合、`pure`はRTSに何の効果も要請しない`IO`アクションであり、`bind`はRTSに潜在的に影響を及ぼす操作を最初に実行するよう指示し、その後で結果の値を使ってプログラムの残りの部分を呼び出します。

二つ目の視点では、`IO`アクションは世界全体を変換します。`IO`アクションは実際には純粋であり、唯一無二の世界を引数として受け取り、その変更された世界を返します。この視点は_内部的_なもので、Leanの内部での`IO`の表現に合っています。Lean内で世界はトークンとして表現され、`IO`モナドは各トークンが正確に一度だけ使用されるように構造化されています。

これがどのように機能するのかを見るためには、定義を一つずつ剥がしていくと役立ちます。`#print`コマンドは、Leanデータ型や定義の内部情報を明らかにします。たとえば、
```lean
{{#example_in Examples/Monads/IO.lean printNat}}
```
は以下の結果になります。
```output info
{{#example_out Examples/Monads/IO.lean printNat}}
```
また、
```lean
{{#example_in Examples/Monads/IO.lean printCharIsAlpha}}
```
は以下の結果になります。
```output info
{{#example_out Examples/Monads/IO.lean printCharIsAlpha}}
```

時には、`#print`の出力には、まだこの本で紹介されていないLeanの機能が含まれています。例えば、
```lean
{{#example_in Examples/Monads/IO.lean printListIsEmpty}}
```
は以下の出力を生成します。
```output info
{{#example_out Examples/Monads/IO.lean printListIsEmpty}}
```
これには、定義名の後に`.{u}`が含まれ、型が単に`Type`ではなく`Type u`と注釈されています。これは今のところ無視して構いません。

`IO`の定義をプリントすると、それがより単純な構造に基づいて定義されているのがわかります。
```lean
{{#example_in Examples/Monads/IO.lean printIO}}
```
```output info
{{#example_out Examples/Monads/IO.lean printIO}}
```
`IO.Error`は、`IO`アクションによってスローされ得るすべてのエラーを表しています。
```lean
{{#example_in Examples/Monads/IO.lean printIOError}}
```
```output info
{{#example_out Examples/Monads/IO.lean printIOError}}
```
`EIO ε α`は、`ε`型のエラーで終了するか、または`α`型の値で成功する`IO`アクションを表します。これは、`Except ε`モナドと同様に、`IO`モナドにはエラーハンドリングと例外の定義能力が含まれていることを意味します。

さらに層を剥がすと、`EIO`自体がより単純な構造によって定義されています。
```lean
{{#example_in Examples/Monads/IO.lean printEIO}}
```
```output info
{{#example_out Examples/Monads/IO.lean printEIO}}
```
`EStateM`モナドはエラーと状態の両方を含んでおり、`Except`と`State`の組み合わせです。それは別の型、`EStateM.Result`を使用して定義されます。
```lean
{{#example_in Examples/Monads/IO.lean printEStateM}}
```
```output info
{{#example_out Examples/Monads/IO.lean printEStateM}}
```
つまり、`EStateM ε σ α`型のプログラムは、初期状態`σ`を受け取る関数であり、`EStateM.Result ε σ α`を返します。

`EStateM.Result`は`Except`の定義に非常に似ており、成功した終了を示すコンストラクタとエラーを示すコンストラクタがあります。
```lean
{{#example_in Examples/Monads/IO.lean printEStateMResult}}
```
```output info
{{#example_out Examples/Monads/IO.lean printEStateMResult}}
```
`Except ε α`のように、`ok`コンストラクタは型`α`の結果を含み、`error`コンストラクタは型`ε`の例外を含みます。しかし`Except`と違って、どちらのコンストラクタも計算の最終状態を含む追加の状態フィールドがあります。

`EStateM ε σ`の`Monad`インスタンスには`pure`と`bind`が必要です。`State`の場合と同様に、`EStateM`の`pure`の実装は初期状態を受け取り、それを変更せずに返します。そして、`Except`の場合と同様に、その引数を`ok`コンストラクタで返します：
```lean
{{#example_in Examples/Monads/IO.lean printEStateMpure}}
```
```output info
{{#example_out Examples/Monads/IO.lean printEStateMpure}}
```
`protected`は、`EStateM`ネームスペースが開かれていても、完全な名前`EStateM.pure`が必要であることを意味します。

同様に、`EStateM`の`bind`も初期状態を引数として取ります。これは初期状態を最初のアクションに渡します。`Except`の`bind`のように、結果がエラーかどうかをチェックします。もしエラーだった場合、そのエラーは変更されずに返され、`bind`の二番目の引数は使用されません。成功した結果であれば、二番目の引数は返された値と生成された状態の両方に適用されます。
```lean
{{#example_in Examples/Monads/IO.lean printEStateMbind}}
```
```output info
{{#example_out Examples/Monads/IO.lean printEStateMbind}}
```

これらすべてをまとめると、`IO`は同時に状態とエラーを追跡するモナドです。利用可能なエラーのコレクションは、プログラムで発生する多くの問題を説明するコンストラクタを持つデータ型`IO.Error`によって与えられます。状態は現実の世界を表す型であり、`IO.RealWorld`と呼ばれます。各基本`IO`アクションはこの現実の世界を受け取り、エラーまたは結果とペアにされた別の世界を返します。`IO`では、`pure`は世界を変更せずに返し、`bind`は一つのアクションから変更された世界を次のアクションに渡します。

宇宙全体はコンピュータのメモリに収まらないので、渡されている世界はただの表現です。世界のトークンが再利用されない限り、この表現は安全です。つまり、世界のトークンには一切のデータを含む必要はありません：
```lean
{{#example_in Examples/Monads/IO.lean printRealWorld}}
```
```output info
{{#example_out Examples/Monads/IO.lean printRealWorld}}
```