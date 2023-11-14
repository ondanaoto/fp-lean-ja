# モナド型クラス

それぞれのモナド型に対して、`ok` や `andThen` のような演算子をインポートする必要がないように、Leanの標準ライブラリには、それらをオーバーロードできる型クラスが含まれており、_任意の_モナドに対して同じ演算子を使うことができます。
モナドには二つの操作があり、それは `ok` と `andThen` の同等物です：
```lean
{{#example_decl Examples/Monads/Class.lean FakeMonad}}
```
この定義はやや単純化されています。
Leanライブラリでの実際の定義はもう少し複雑で、後で紹介されます。

`Option` や `Except` の `Monad` インスタンスは、それぞれの `andThen` 操作の定義を適応することで作成できます：
```lean
{{#example_decl Examples/Monads/Class.lean MonadOptionExcept}}
```

例として、`firstThirdFifthSeventh` は `Option α` と `Except String α` の戻り値の型に対して別々に定義されました。
今は、_任意の_モナドに対して多相的に定義することができます。
しかしながら、異なるモナドが結果を見つけられない異なる方法について考慮するため、引数としてのルックアップ関数が必要です。
`bind` の中置バージョンは `>>=` で、これは例の中の `~~>` と同じ役割を果たします。
```lean
{{#example_decl Examples/Monads/Class.lean firstThirdFifthSeventhMonad}}
```

ゆっくり動く哺乳類と速く飛ぶ鳥の例としてのリストが与えられたとき、この実装の `firstThirdFifthSeventh` は `Option` と一緒に使うことができます：
```lean
{{#example_decl Examples/Monads/Class.lean animals}}

{{#example_in Examples/Monads/Class.lean noneSlow}}
```
```output info
{{#example_out Examples/Monads/Class.lean noneSlow}}
```
```lean
{{#example_in Examples/Monads/Class.lean someFast}}
```
```output info
{{#example_out Examples/Monads/Class.lean someFast}}
```

`Except` のルックアップ関数 `get` の名前をより具体的なものに変えると、同じ実装の `firstThirdFifthSeventh` を `Except` と一緒に使うこともできます：
```lean
{{#example_decl Examples/Monads/Class.lean getOrExcept}}

{{#example_in Examples/Monads/Class.lean errorSlow}}
```
```output info
{{#example_out Examples/Monads/Class.lean errorSlow}}
```
```lean
{{#example_in Examples/Monads/Class.lean okFast}}
```
```output info
{{#example_out Examples/Monads/Class.lean okFast}}
```
`m` が `Monad` インスタンスを持っていなければならないという事実は、`>>=` 操作と `pure` 操作が利用可能であることを意味します。

## 一般的なモナド操作

多くの異なる型がモナドであるため、_任意の_モナドに対して多相的な関数は非常に強力です。
例えば、関数 `mapM` は `Monad` を使用して関数を適用した結果を順序づけて組み合わせる `map` のバージョンです：
```lean
{{#example_decl Examples/Monads/Class.lean mapM}}
```
関数引数 `f` の戻り値の型はどの `Monad` インスタンスが使用されるかを決定します。
言い換えると、`mapM` はログを生成する関数、失敗する可能性がある関数、または可変状態を使用する関数に対して使用できます。
`f` の型が利用可能な効果を決定するため、APIデザイナーはそれらを厳密に制御することができます。

[この章のイントロダクション](../monads.md#numbering-tree-nodes)で記述されているように、`State σ α` は可変変数の型 `σ` を使用し、型 `α` の値を返すプログラムを表します。
これらのプログラムは実際には、開始状態から値と最終状態のペアへの関数です。
`Monad` クラスでは、そのパラメーターが単一の型引数を期待することが必要です。つまり、`Type → Type` であるべきです。
これは、`State` のインスタンスは状態型 `σ` を言及し、インスタンスへのパラメーターになるべきだということを意味します：
```lean
{{#example_decl Examples/Monads/Class.lean StateMonad}}
```
これは、`get` と `set` の呼び出し之间を `bind` を使用してシーケンスするときに、状態の型は変化することはないという合理的なルールです。
オペレータ `increment` は与えられた量だけ保存された状態を増加させ、古い値を返します:
```lean
{{#example_decl Examples/Monads/Class.lean increment}}
```

`mapM` を `increment` と一緒に使用すると、リストのエントリーの総和を計算するプログラムになります。
より具体的には、可変変数はこれまでの総和を含み、結果としてのリストは逐次的な総和を含んでいます。
言い換えると、`{{#example_in Examples/Monads/Class.lean mapMincrement}}`は型`{{#example_out Examples/Monads/Class.lean mapMincrement}}`を持ち、`State`の定義を展開すると`{{#example_out Examples/Monads/Class.lean mapMincrement2}}`となります。
初期総和としての引数を取り、これは `0` であるべきです:
```lean
{{#example_in Examples/Monads/Class.lean mapMincrementOut}}
```
```output info
{{#example_out Examples/Monads/Class.lean mapMincrementOut}}
```

[ログ効果](../monads.md#logging)は`WithLog` を使用することで表現できます。
`State`と同じように、その `Monad` インスタンスはログデータの型に対して多态的です:
```lean
{{#example_decl Examples/Monads/Class.lean MonadWriter}}
```
`saveIfEven` は偶数をログに記録するが、引数を変更せずに返す関数です:
```lean
{{#example_decl Examples/Monads/Class.lean saveIfEven}}
```
この関数を `mapM` と一緒に使用すると、偶数が含まれるログと変更されていない入力リストのペアが得られます:
```lean
{{#example_in Examples/Monads/Class.lean mapMsaveIfEven}}
```
```output info
{{#example_out Examples/Monads/Class.lean mapMsaveIfEven}}
```



## アイデンティティモナド

モナドは、失敗、例外、ログのような効果をデータと関数としての明示的な表現にエンコードします。
しかし、たまにAPIが多様性のためにモナドを使用して記述されているが、APIのクライアントはエンコードされた効果を必要としない場合があります。
_アイデンティティモナド_ は、効果のないモナドであり、純粋なコードをモナディックなAPIとともに使用させることができます：
```lean
{{#example_decl Examples/Monads/Class.lean IdMonad}}
```
`pure` の型は `α → Id α` であるべきですが、`Id α` は単純に `α` にまで簡約されます。
同様に、`bind` の型は `α → (α → Id β) → Id β` であるべきです。
これは `α → (α → β) → β` にまで簡約されるため、第二引数は第一引数に適用して結果を見つけることができます。

アイデンティティモナドを使うと、`mapM` は `map` に等価になります。
しかし、Leanでは使用されるモナドが `Id` であるというヒントが必要です：
```lean
{{#example_in Examples/Monads/Class.lean mapMId}}
```
```output info
{{#example_out Examples/Monads/Class.lean mapMId}}
```
ヒントがないとエラーになります：
```lean
{{#example_in Examples/Monads/Class.lean mapMIdNoHint}}
```
```output error
{{#example_out Examples/Monads/Class.lean mapMIdNoHint}}
```
このエラーでは、1つのメタ変数を別のメタ変数に適用することは、Leanが型レベルの計算を後方に実行しないことを示しています。
関数の戻り値の型は何らかの他の型に適用されたモナドであることが期待されます。
同様に、どのモナドを使用しようとも特定のヒントを提供しない型の関数と `mapM` を使用すると、「インスタンス問題が詰まる」というメッセージが表示されます：
```lean
{{#example_in Examples/Monads/Class.lean mapMIdId}}
```
```output error
{{#example_out Examples/Monads/Class.lean mapMIdId}}
```


## モナドの契約
`BEq` インスタンスと `Hashable` インスタンスのすべてのペアが等しい値を同じハッシュにするべきだというように、`Monad` のインスタンスごとにモナド契約が遵守されるべきです。
最初に、`pure` は `bind` の左同一性でなければならない。
つまり、`bind (pure v) f` は `f v` と同じであるべきです。
次に、`pure` は `bind` の右同一性であり、したがって `bind v pure` は `v` と同じであるべきです。
最後に、`bind` は結合的であるため、`bind (bind v f) g` は `bind v (fun x => bind (f x) g)` と同じであるべきです。

この契約は、より一般的に効果を持つプログラムの期待される特性を指定しています。
`pure` には効果がないので、`bind` でその効果をシーケンス化することは結果を変えるべきではありません。
`bind` の結合性は基本的に、シーケンスブックキーピング自体は問題ではないということを言っています。つまり、事象が発生する順序が保持される限り。

## 練習問題

### 木へのマッピング

関数 `BinTree.mapM` を定義してください。
リストに対する `mapM` との類例により、この関数はモナディック関数をツリーの各データエントリに前順で適用するべきです。
型シグネチャは以下のようになるべきです：
```
def BinTree.mapM [Monad m] (f : α → m β) : BinTree α → m (BinTree β)
```


### Optionモナド契約

最初に、`Option` の `Monad` インスタンスがモナド契約を満たしていることを説得的に証明してください。
それから、以下のインスタンスを考えます：
```lean
{{#example_decl Examples/Monads/Class.lean badOptionMonad}}
```
両方のメソッドは正しい型を持っています。
なぜこのインスタンスがモナド契約に違反するのか説明してください。