```markdown
# 完全な定義

ここに、Lean 標準ライブラリにおける `Functor`、`Applicative`、そして `Monad` の完全かつ正直な定義を記述しました。
理解のために、詳細は省略されていません。

## Functor

`Functor` クラスの完全な定義は、宇宙多相性 (universe polymorphism) とデフォルトメソッド実装を使用します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean HonestFunctor}}
```
この定義において、`Function.comp` は関数の合成であり、通常 `∘` 演算子で書かれます。
`Function.const` は_定数関数 (constant function)_ で、第二引数を無視する二つの引数を取る関数です。
この関数に一つの引数だけを適用すると、常に同じ値を返す関数が得られ、APIが関数を要求するものの、プログラムが異なる引数に対して異なる結果を計算する必要がない場合に便利です。
`Function.const` のシンプルなバージョンは次のように書けます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean simpleConst}}
```
これを `List.map` に対する関数引数として一つの引数で使用するとその有用性がわかります：
```lean
{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean mapConst}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean mapConst}}
```
実際の関数には以下のようなシグネチャがあります：
```output info
{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean FunctionConstType}}
```
ここで、型引数 `β` は明示的な引数であり、そのため `Functor.mapConst` のデフォルト定義は `_` 引数を提供し、プログラムがタイプチェックされるように `Function.const` に渡すユニークな型を Lean が見つけるよう指示します。
`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean unfoldCompConst}}` は `{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean unfoldCompConst}}` と同等です。

`Functor` 型クラスは `u+1` と `v` の大きな方にある宇宙に属しています。
ここで、`u` は `f` に対して引数として受け入れられる宇宙のレベル、`v` は `f` によって返される宇宙です。
`Functor` 型クラスを実装する構造が `u` よりも大きな宇宙になければならない理由を理解するためには、クラスの簡素化された定義から始めます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean FunctorSimplified}}
```
この型クラスの構造型は次の帰納型と等価です：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean FunctorDatatype}}
```
`Functor.mk` に引数として渡される `map` メソッドの実装には、`Type u` 内の二つの型を引数とする関数が含まれています。
これは関数自身の型が `Type (u+1)` 内にあることを意味するため、`Functor` も少なくとも `u+1` のレベルになければなりません。
同様に、関数の他の引数には `f` を適用して作られた型が含まれるため、`v` のレベルにも達している必要があります。
この節で共有されているすべての型クラスがこのプロパティを共有しています。

## Applicative

`Applicative` 型クラスは実際には、`pure` や `seq` などの関連メソッドを含むいくつかの小さなクラスから構成されています。
最初に `Pure` と `Seq` があり、それぞれ `pure` と `seq` を含んでいます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean Pure}}

{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean Seq}}
```

これに加えて、`Applicative` は `SeqRight` と類似の `SeqLeft` クラスにも依存しています：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean SeqRight}}

{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean SeqLeft}}
```

[代替と検証に関する節](alternative.md)で紹介された `seqRight` 関数は、効果の観点から理解するのが最も簡単です。
`{{#example_in Examples/FunctorApplicativeMonad.lean seqRightSugar}}` は `{{#example_out Examples/FunctorApplicativeMonad.lean seqRightSugar}}` に展開され、`E1` を実行してから `E2` を実行し、`E2` の結果だけを結果として得ると理解できます。
`E1` の効果によって `E2` が実行されなかったり、複数回実行されたりすることがあります。
確かに、`f` が `Monad` のインスタンスを持っているならば、`E1 *> E2` は `do let _ ← E1; E2` と同等ですが、`seqRight` は `Validate` のようなモナドではない型で使用できます。

そのいとこ `seqLeft` も非常に似ていますが、左側の式の値が返されます。
`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean seqLeftSugar}}` は `{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean seqLeftSugar}}` に展開されます。
`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean seqLeftType}}` の型は `{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean seqLeftType}}` となり、これは `seqRight` と同じですが、`f α` を返すという点だけが異なります。
`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean seqLeftSugar}}` は `E1` を最初に実行し、次に `E2` を実行し、`E1` に対する元の結果を返すプログラムとして理解できます。
`f` が `Monad` のインスタンスを持っているならば、`E1 <* E2` は `do let x ← E1; _ ← E2; pure x` と同等です。
一般的に、`seqLeft` は、値自体を変更することなく、検証やパーサーのようなワークフローにおいて値に追加の条件を指定するために役立ちます。

`Applicative` の定義はこれらのクラスすべてを拡張しますが、`Functor` も含んでいます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean Applicative}}
```
`Applicative` の完全な定義には `pure` と `seq` の定義だけが必要です。
これは `Functor`、`SeqLeft`、`SeqRight` のすべてのメソッドにデフォルトの定義があるためです。
`Functor` の `mapConst` メソッドには `Functor.map` に基づく独自のデフォルト実装があります。
これらのデフォルト実装は、振る舞いが同等でありながら効率的な新たな関数に置き換えられるべきです。
デフォルトの実装は、正確性の仕様と同時に自動生成されたコードとして見るべきです。

`seqLeft` のデフォルト実装は非常にコンパクトです。
いくつかの名前を構文糖やそれらの定義に置き換えることで別の視点が得られるため：
```lean
{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean unfoldMapConstSeqLeft}}
```
になります
```lean
{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean unfoldMapConstSeqLeft}}
```
`(fun x _ => x) <$> a` はどのように理解すべきでしょうか？
ここでは、`a` は型 `f α` を持ち、`f` はファンクターです。
`f` が `List` である場合、`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstList}}` は `{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstList}}` と評価されます。
`f` が `Option` である場合、`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstOption}}` は `{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstOption}}` と評価されます。
どちらの場合でも、ファンクター内の値は引数を無視して元の値を返す関数に置き換えられます。
`seq` と組み合わせると、この関数は `seq` の第二引数の値を破棄します。

`seqRight` のデフォルト実装も非常に類似していますが、`const` には追加の引数 `id` があります。
この定義も、いくつかの標準的な構文糖を導入し、それからいくつかの名前をそれらの定義に置き換えることで同様に理解することができます：
```lean
{{#example_eval Examples/FunctorApplicativeMonad/ActualDefs.lean unfoldMapConstSeqRight}}
```
`(fun _ x => x) <$> a` はどのように理解すべきでしょうか？
やはり、例が役立ちます。
`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstIdList}}` は `{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstIdList}}` と同等であり、`{{#example_in Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstIdOption}}` は `{{#example_out Examples/FunctorApplicativeMonad/ActualDefs.lean mapConstIdOption}}` と同等です。
言い換えれば、`(fun _ x => x) <$> a` は `a` の全体的な形を維持しつつ、各値がアイデンティティ関数で置き換えられます。
効果の観点から、`a` の副作用は起こりますが、それが `seq` で使われるときに値は捨てられます。

## Monad

`Applicative` の構成操作がそれぞれ独自の型クラスに分割されているように、`Bind` も独自のクラスを持っています：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean Bind}}
```
`Monad` は `Applicative` と `Bind` で拡張されます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad/ActualDefs.lean Monad}}
```
インスタンスが要求するメソッドの集合と、階層全体から得られるデフォルトメソッドを追跡すると、`Monad` インスタンスには `bind` と `pure` の実装だけが必要です。
言い換えれば、`Monad` インスタンスは自動的に `seq`、`seqLeft`、`seqRight`、`map`、`mapConst` の実装を導出します。
API 境界の観点から、`Monad` のインスタンスを持つ任意の型は `Bind`、`Pure`、`Seq`、`Functor`、`SeqLeft`、`SeqRight` のインスタンスを得ます。


## エクササイズ

 1. `Monad` における `map`、`seq`、`seqLeft`、`seqRight` のデフォルト実装を、`Option` や `Except` などの例を通じて理解してみてください。つまり、その定義を `bind` と `pure` に代替してデフォルト定義に代入し、手書きされるであろうバージョンの `map`、`seq`、`seqLeft`、`seqRight` に単純化してください。
 2. 紙に書いたりテキストファイルに記述したりして、`map` と `seq` のデフォルト実装が `Functor` および `Applicative` の契約を満たすことを自分自身に証明してみてください。この議論で使用するのは `Monad` 契約からの規則と通常の式の評価です。
```