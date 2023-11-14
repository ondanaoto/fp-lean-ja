# モナド用do記法

APIがモナドに基づいていることは非常に強力ですが、匿名関数と共に`>>=`の明示的な使用はまだ少しわかりにくいものがあります。
`HAdd.hAdd`への明示的な呼び出しの代わりに中置演算子が使われるのと同様に、Leanには_`do`記法_と呼ばれるモナド用の構文があり、これによりモナドを使用するプログラムの読み書きが容易になります。
これは、`IO`プログラムを記述する際に使用される全く同じ`do`記法であり、`IO`もまたモナドです。

[Hello, World!](../hello-world.md)では、`do`記法が`IO`アクションを組み合わせるために使用されていますが、これらのプログラムの意味は直接説明されています。
モナドを使ってプログラミングする方法を理解することは、`do`を基本的なモナド演算子の使用方法に基づいて説明することを意味しています。

`do`の最初の翻訳は、`do`内の唯一の文が単一の式`E`である場合に使用されます。
このケースでは、`do`は取り除かれ、
```lean
{{#example_in Examples/Monads/Do.lean doSugar1}}
```
は以下のように翻訳されます。
```lean
{{#example_out Examples/Monads/Do.lean doSugar1}}
```

2番目の翻訳は、`do`の最初の文が矢印を持つ`let`であり、ローカル変数を束縛する場合に使用されます。
これは、その同じ変数を束縛する関数と共に`>>=`の使用に翻訳されるので、
```lean
{{#example_in Examples/Monads/Do.lean doSugar2}}
```
は以下のように翻訳されます。
```lean
{{#example_out Examples/Monads/Do.lean doSugar2}}
```

`do`ブロックの最初の文が式である場合、それは`Unit`を返すモナドアクションと見なされ、そのため関数は`Unit`コンストラクタにマッチし、
```lean
{{#example_in Examples/Monads/Do.lean doSugar3}}
```
は以下のように翻訳されます。
```lean
{{#example_out Examples/Monads/Do.lean doSugar3}}
```

最後に、`do`ブロックの最初の文が`:=`を使用する`let`である場合、翻訳された形式は通常の`let`式で、
```lean
{{#example_in Examples/Monads/Do.lean doSugar4}}
```
は以下のように翻訳されます。
```lean
{{#example_out Examples/Monads/Do.lean doSugar4}}
```

`Monad`クラスを使用して書かれた`firstThirdFifthSeventh`の定義は以下のようになります。
```lean
{{#example_decl Examples/Monads/Class.lean firstThirdFifthSeventhMonad}}
```
`do`記法を使用すると、それはかなり読みやすくなります：
```lean
{{#example_decl Examples/Monads/Do.lean firstThirdFifthSeventhDo}}
```

`Monad`型クラスなしでは、木のノードに番号を付ける関数`number`が以下のように書かれました：
```lean
{{#example_decl Examples/Monads.lean numberMonadicish}}
```
`Monad`と`do`を使用すると、その定義はかなり煩わしさがなくなります：
```lean
{{#example_decl Examples/Monads/Do.lean numberDo}}
```


`IO`での`do`のすべての便利さは、他のモナドで使う場合にも利用可能です。
例えば、ネストされたアクションはどんなモナドでも動作します。
`mapM`の元の定義は以下のようでした：
```lean
{{#example_decl Examples/Monads/Class.lean mapM}}
```
`do`記法を使うと、以下のように書くことができます：
```lean
{{#example_decl Examples/Monads/Do.lean mapM}}
```
ネストされたアクションを使用することで、それはオリジナルの非モナド`map`とほぼ同じ短さになります：
```lean
{{#example_decl Examples/Monads/Do.lean mapMNested}}
```
ネストされたアクションを使って、`number`関数をより簡潔にすることができます：
```lean
{{#example_decl Examples/Monads/Do.lean numberDoShort}}
```



## 練習問題

 * `evaluateM`、そのヘルパー、および異なる特定の使用例を`>>=`を明示的に呼び出す代わりに`do`記法を使用して書き直してください。
 * `firstThirdFifthSeventh`をネストされたアクションを使って書き直してください。