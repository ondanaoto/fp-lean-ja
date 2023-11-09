# モナド

C#やKotlinでは、`?.`オペレーターはnullかもしれない値のプロパティをルックアップしたりメソッドを呼び出したりする方法です。
もしレシーバが`null`なら、その全ての式はnullになります。
そうでなければ、基本となる非`null`の値が呼び出しを受けます。
`?.`の使用は連鎖的に可能で、その場合最初のnull結果がルックアップの鎖を終了させます。
このようにnullチェックを連鎖させるのは、深くネストされた`if`文を書いて維持していくよりもはるかに便利です。

同様に、エラーコードの手動でのチェックと伝播よりも例外はかなり便利です。
同時に、ロギングは専用のロギング・フレームワークを持つことにより、関数がそのログ結果と戻り値を両方返すことよりも実現が最も簡単です。
連鎖するnullチェックと例外は通常、言語設計者がこの使用例を予期することを必要とし、一方でロギング・フレームワークは、ログの蓄積からログを記録するコードを分離するために副作用を使用することが一般的です。

これらの機能やその他の多くの機能は、`モナド`と呼ばれる共通APIのインスタンスとしてライブラリコードで実装することができます。
LeanはこのAPIを使用するのに便利な専用の構文を提供していますが、背後で何が起こっているのかを理解する際には邪魔になることがあります。
この章は、手動でnullチェックをネストする詳細なプレゼンテーションから始まり、そこから便利で一般的なAPIへと拡張されます。
その間は、信じがたいことに耐えてください。

## `none`のチェック：繰り返しを避ける

Leanでは、パターンマッチングを使ってnullのチェックを連鎖させることができます。
リストから最初のエントリを取得するには、オプショナルなインデックス記法を使うだけです：
```lean
{{#example_decl Examples/Monads.lean first}}
```
結果は`Option`である必要があります。なぜなら空のリストには最初のエントリがないからです。
最初と3番目のエントリを抽出するには、それぞれが`none`ではないというチェックが必要です：
```lean
{{#example_decl Examples/Monads.lean firstThird}}
```
同様に、最初、3番目、5番目のエントリを抽出するには、さらに値が`none`でないというチェックが必要です：
```lean
{{#example_decl Examples/Monads.lean firstThirdFifth}}
```
そして、このシーケンスに7番目のエントリを追加すると、管理がかなり困難になり始めます：
```lean
{{#example_decl Examples/Monads.lean firstThirdFifthSeventh}}
```


このコードの基本的な問題は、数値を抽出することと全てが存在することを確認することの2つの関心を扱っていることですが、2番目の関心は`none`の場合のコードをコピー＆ペーストすることで対応されています。
繰り返されるセグメントをヘルパー関数に持ち上げることは、しばしば良いスタイルです：
```lean
{{#example_decl Examples/Monads.lean andThenOption}}
```
このヘルパーは、C#やKotlinでの`?.`と同様に、`none`の値を伝播することを担当します。
これは、オプショナルな値と、その値が`none`でない場合に適用される関数の2つの引数をとります。
最初の引数が`none`の場合、ヘルパーは`none`を返します。
最初の引数が`none`でない場合、関数は`some`コンストラクタの内容に適用されます。

これで、`firstThird`をパターンマッチングの代わりに`andThen`を使って書き直すことができます：
```lean
{{#example_decl Examples/Monads.lean firstThirdandThen}}
```
Leanでは、関数は引数として渡されるときに括弧で囲む必要はありません。
以下の同等の定義は、より多くの括弧を使用し、関数のボディをインデントしています：
```lean
{{#example_decl Examples/Monads.lean firstThirdandThenExpl}}
```
`andThen`ヘルパーは値が流れる一種の「パイプライン」を提供し、インデントが少し変わったバージョンの方がこの事実をより暗示しています。
`andThen`を書くための構文を改善することで、これらの計算をさらに理解しやすくすることができます。

### 中置演算子

Leanでは、中置演算子は`infix`、`infixl`、`infixr`コマンドを使用して宣言され、それぞれ非結合、左結合、右結合の演算子を作成します。
連続して使用される場合、_左結合_の演算子は開き括弧を式の左側に積み上げます。
加算演算子`+`は左結合なので、`{{#example_in Examples/Monads.lean plusFixity}}`は`{{#example_out Examples/Monads.lean plusFixity}}`と同値です。
べき乗演算子`^`は右結合なので、`{{#example_in Examples/Monads.lean powFixity}}`は`{{#example_out Examples/Monads.lean powFixity}}`と同値です。
`<`のような比較演算子は非結合なので、`x < y < z`は構文エラーであり、手動で括弧を付ける必要があります。

次の宣言は`andThen`を中置演算子にします：
```lean
{{#example_decl Examples/Monads.lean andThenOptArr}}
```
コロンの後の数字は、新しい中置演算子の_優先順位_を宣言します。
通常の数学記法では、`{{#example_in Examples/Monads.lean plusTimesPrec}}`は`{{#example_out Examples/Monads.lean plusTimesPrec}}`と等価ですが、`+`と`*`は共に左結合です。
Leanでは、`+`は優先順位65、`*`は優先順位70を持ちます。
優先度の高い演算子は、優先度の低い演算子よりも先に適用されます。
`~~>`の宣言に従うと、`+`と`*`は両方ともより高い優先順位を持ち、つまり先に適用されます。
通常、一連の演算子の最も便利な優先順位を決定するには、いくつかの実験と多くの例が必要になります。

新しい中置演算子の後には`=>`の二重矢印があり、中置演算子に使用される名前付き関数を指定します。
Leanの標準ライブラリはこの機能を使用して、型クラスを使用して中置演算子をオーバーロードする`HAdd.hAdd`と`HMul.hMul`をそれぞれ指す中置演算子として`+`と`*`を定義しています。
しかし、ここでの`andThen`はただの通常の関数です。

`andThen`を中置演算子として定義することにより、`firstThird`は「パイプライン」の感じを前面に出す方法で書き直されます：
```lean
{{#example_decl Examples/Monads.lean firstThirdInfix}}
```
このスタイルは、より大きな関数を書く際にはるかに簡潔です：
```lean
{{#example_decl Examples/Monads.lean firstThirdFifthSeventInfix}}
```

## エラーメッセージの伝播

純粋な関数型言語であるLeanにはエラーハンドリングのための組み込みの例外メカニズムはありません。というのも、例外を投げるまたはキャッチするのは式のステップごとの評価モデルの外にあります。
しかし、関数型プログラムには確かにエラーを処理する必要があります。
`firstThirdFifthSeventh`の場合、リストがどのくらい長かったのか、どこでルックアップが失敗したのかをユーザに知らせることが関連している可能性が高いです。

これは通常、エラーまたは結果を持つことができるデータ型を定義し、例外を持つ関数をこのデータ型を返す関数に変換することによって実現されます：
```lean
{{#example_decl Examples/Monads.lean Except}}
```
型変数`ε`は、関数によって生成されるエラーの型を意味します。
呼び出し側はエラーと成功の両方を処理することが期待され、これによって型変数`ε`はJavaのチェックされた例外のリストと似たような役割を果たします。

`Option`のように、`Except`はリストでエントリが見つからなかったことを表すために使用できます。
このケースでは、エラータイプは`String`です：
```lean
{{#example_decl Examples/Monads.lean getExcept}}
```
範囲内の値のルックアップは`Except.ok`を生み出します：
```lean
{{#example_decl Examples/Monads.lean ediblePlants}}

{{#example_in Examples/Monads.lean success}}
```
```output info
{{#example_out Examples/Monads.lean success}}
```
範囲外の値のルックアップは`Except.error`を生み出します：
```lean
{{#example_in Examples/Monads.lean failure}}
```
```output info
{{#example_out Examples/Monads.lean failure}}
```

単一のリストのルックアップは便利に値またはエラーを返すことができます：
```lean
{{#example_decl Examples/Monads.lean firstExcept}}
```
しかし、2つのリストのルックアップを実行するには失敗の可能性を扱う必要があります：
```lean
{{#example_decl Examples/Monads.lean firstThirdExcept}}
```
関数に別のリストのルックアップを追加するには、さらに多くのエラーハンドリングが必要です：
```lean
{{#example_decl Examples/Monads.lean firstThirdFifthExcept}}
```
もう一つのリストのルックアップを追加すると、かなり取り扱いが困難になります：
```lean
{{#example_decl Examples/Monads.lean firstThirdFifthSeventhExcept}}
```

再び、共通のパターンをヘルパーに切り出すことができます。
関数を通じての各ステップではエラーがチェックされ、結果が成功だった場合にのみ残りの計算が続行されます。
`Except`のための新しいバージョンの`andThen`を定義できます：
```lean
{{#example_decl Examples/Monads.lean andThenExcept}}
```
`Option`の場合と同じように、このバージョンの`andThen`は`firstThird`のより簡潔な定義を可能にします：
```lean
{{#example_decl Examples/Monads.lean firstThirdAndThenExcept}}
```

`Option`と`Except`の両方のケースでは、2つの繰り返しパターンがあります：それぞれのステップで中間結果をチェックすること、そして`andThen`に切り出されたもの、また最終的な成功結果はそれぞれ`some`もしくは`Except.ok`です。
利便性のために、成功は`ok`というヘルパーに切り出すことができます：
```lean
{{#example_decl Examples/Monads.lean okExcept}}
```
同様に、失敗は`fail`というヘルパーに切り出すことができます：
```lean
{{#example_decl Examples/Monads.lean failExcept}}
```
`ok`と`fail`を使用することで、`get`を少し読みやすくすることができます：
```lean
{{#example_decl Examples/Monads.lean getExceptEffects}}
```


`andThen`の中置宣言を追加した後、`firstThird`は`Option`を返すバージョンと同じくらい簡潔にすることができます：
```lean
{{#example_decl Examples/Monads.lean andThenExceptInfix}}

{{#example_decl Examples/Monads.lean firstThirdInfixExcept}}
```
この技術は大きな関数に似たように拡大されます：
```lean
{{#example_decl Examples/Monads.lean firstThirdFifthSeventInfixExcept}}
```

## ロギング

2で割ったときに余りが出ない場合、数は偶数です：
```lean
{{#example_decl Examples/Monads.lean isEven}}
```
関数`sumAndFindEvens`は、途中で遭遇した偶数を記憶しながらリストの合計を計算します：
```lean
{{#example_decl Examples/Monads.lean sumAndFindEvensDirect}}
```
この関数は一般的なパターンの単純化された例です。
多くのプログラムはデータ構造を一度だけトラバースする必要があり、メインの結果を計算するとともに、何らかの二次的な追加結果を蓄積します。
この一つの例がロギングです：プログラムがIOアクションであれば、常にディスク上のファイルにログを記録できますが、ディスクが