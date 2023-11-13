# アプリカティブ契約

`Functor`、`Monad`、`BEq` や `Hashable` を実装する型と同様に、`Applicative` もすべてのインスタンスが従うべき一連の規則を持っています。

アプリカティブファンクターが従うべきルールは以下の四つです:
1. 結合律を尊重し、`pure id <*> v = v` であるべきです。
2. 関数の合成を尊重し、`pure (· ∘ ·) <*> u <*> v <*> w = u <*> (v <*> w)` であるべきです。
3. 純粋な操作のシーケンスは、ノーオペレーション（no-op）になるべきなので、`pure f <*> pure x = pure (f x)` です。
4. 純粋な操作の順序は関係なく、`u <*> pure x = pure (fun f => f x) <*> u` であるべきです。

これらを `Applicative Option` インスタンスについてチェックするには、`pure`を`s​​ome`に展開することから始めます。

最初のルールは、`some id <*> v = v`です。
`Option`の`seq`の定義によると、これは`id <$> v = v`と同じであり、これはすでに確認された`Functor`の規則の1つです。

二番目のルールは、`some (· ∘ ·) <*> u <*> v <*> w = u <*> (v <*> w)`です。
`u`、`v`、または`w`のいずれかが`none`の場合、両方の側が`none`になりますので、特性が保持されます。
`u`が`some f`であり、`v`が`some g`であり、`w`が`some x`であると仮定すると、これは`some (· ∘ ·) <*> some f <*> some g <*> some x = some f <*> (some g <*> some x)`ということになります。
両側を評価すると同じ結果が得られます：
```lean
{{#example_eval Examples/FunctorApplicativeMonad.lean OptionHomomorphism1}}

{{#example_eval Examples/FunctorApplicativeMonad.lean OptionHomomorphism2}}
```

三番目のルールは、`seq`の定義から直接導かれます：
```lean
{{#example_eval Examples/FunctorApplicativeMonad.lean OptionPureSeq}}
```

四番目のケースでは、`u`が`some f`であると仮定します。なぜならそれが`none`の場合、方程式の両側は`none`になるからです。
`some f <*> some x`は直接`some (f x)`に評価され、`some (fun g => g x) <*> some f`もそうなります。


## すべてのアプリカティブはファンクターです

`Applicative`の二つのオペレーターは、`map`を定義するのに十分です：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean ApplicativeMap}}
```

しかしながら、これが`Functor`を実装することに使うことは、`Applicative`の契約が`Functor`の契約を保証する場合に限られます。
`Functor`の最初のルールは、`id <$> x = x`であり、これは`Applicative`の最初のルールから直接導かれます。
`Functor`の二番目のルールは、`map (f ∘ g) x = map f (map g x)`です。
ここで`map`の定義を展開すると、`pure (f ∘ g) <*> x = pure f <*> (pure g <*> x)`になります。
純粋な操作のシーケンスがノーオペレーションであるという規則を使って、左側を`pure (· ∘ ·) <*> pure f <*> pure g <*> x`に書き換えることができます。
これはアプリカティブファンクターが関数の合成を尊重するという規則の一例です。

これは、`Applicative`が`Functor`を拡張している定義を正当化し、`pure`と`seq`を使って`map`のデフォルト定義を提供します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean ApplicativeExtendsFunctorOne}}
```

## すべてのモナドはアプリカティブファンクターです

`Monad`のインスタンスは既に`pure`の実装を必要としています。
これと`bind`と一緒に、`seq`を定義するのに十分です：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean MonadSeq}}
```
もう一度、`Monad`の契約が`Applicative`の契約を含意していることをチェックすれば、これを`Applicative`を拡張する`Monad`に対する`seq`のデフォルト定義として使うことができます。

この部分の残りは、`bind`に基づいて`seq`を実装するこの定義が実際に`Applicative`の契約を満たしているという主張で構成されています。
関数プログラミングの素晴らしいことの一つは、この種の主張を[式の評価に関する最初のセクション](../getting-to-know/evaluating.md)からの評価ルールを使って、紙と鉛筆で解き出すことができるということです。
これらの引数を読む際に操作の意味を考えると、理解に役立つことがあります。

`do`記法を明示的な`>>=`の使用に置き換えると、`Monad`のルールを適用しやすくなります：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean MonadSeqDesugar}}
```


結合律を尊重しているかをチェックするには、`seq (pure id) (fun () => v) = v`であることを確認します。
左側は`pure id >>= fun g => (fun () => v) () >>= fun y => pure (g y)`に相当します。
中央の単位関数はすぐに削除でき、`pure id >>= fun g => v >>= fun y => pure (g y)`になります。
`pure`が`>>=`の左単位元であるという事実を使用すると、これは`v >>= fun y => pure (id y)`、つまり`v >>= fun y => pure y`と同じです。
`fun x => f x`が`f`と同じなので、これは`v >>= pure`と同じであり、`pure`が`>>=`の右単位元であるという事実を使用して`v`となります。

この種の非公式な推論は、少しのフォーマッティングでより読みやすくすることができます。
以下のチャートでは、「EXPR1 ={ REASON }= EXPR2」を「EXPR1はREASONのためEXPR2と同じ」と読みます：
{{#equations Examples/FunctorApplicativeMonad.lean mSeqRespId}}


それが関数の合成を尊重しているかをチェックするには、「`pure (· ∘ ·) <*> u <*> v <*> w = u <*> (v <*> w)`」であることを確認します。
最初のステップは`<*>`をこの定義の`seq`で置き換えることです。
その後、`Monad`の契約から恒等性と結合性のルールを使用した（やや長い）一連のステップで一方から他方への十分な取得です：
{{#equations Examples/FunctorApplicativeMonad.lean mSeqRespComp}}

純粋な操作のシーケンスがノーオペレーションであることをチェックするには：
{{#equations Examples/FunctorApplicativeMonad.lean mSeqPureNoOp}}

最後に、純粋な操作の順序が関係ないことをチェックするには：
{{#equations Examples/FunctorApplicativeMonad.lean mSeqPureNoOrder}}

これにより、`Applicative`を拡張する`Monad`の定義が正当化され、`seq`のデフォルト定義が提供されます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean MonadExtends}}
```
`Applicative`自身の`map`のデフォルト定義により、すべての`Monad`インスタンスは自動的に`Applicative`と`Functor`のインスタンスも生成します。

## 追加の定義

各型クラスに関連する個々の契約に準拠することに加えて、`Functor`、`Applicative`、および`Monad`の組み合わせ実装は、これらのデフォルト実装に相当するように動作するべきです。
言い換えれば、`Applicative`と`Monad`の両方のインスタンスを提供する型は、バージョンが`Monad`インスタンスがデフォルト実装として生成するバージョンとは異なる動作をする`seq`の実装を持つべきではありません。
これは重要です。なぜなら、多相関数は`>>=`の使用を`<*>`の同等の使用に置き換える、あるいは`<*>`の使用を`>>=`の同等の使用に置き換えるためにリファクタリングされるかもしれないからです。
このリファクタリングは、このコードを使用するプログラムの意味を変更するべきではありません。

このルールは、`Validate.andThen`を`Monad`インスタンスで`bind`を実装するために使用すべきではない理由を説明しています。
それ自体はモナド契約に従います。
ただし、それを`seq`の実装に使用した場合、動作は`seq`自体と同等ではありません。
どこが異なるかを見るために、両方がエラーを返す二つの計算の例を取ります。
関数を検証する際に二つのエラーが返されるべきケースの例を出発点としてください（これは関数への以前の引数から得られたものと同様になるかもしれません）、および引数を検証する際のエラーです：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean counterexample}}
```

`Validate`の`Applicative`インスタンスからの`<*>`のバージョンでそれらを組み合わせると、ユーザーに両方のエラーが報告されます：
```lean
{{#example_eval Examples/FunctorApplicativeMonad.lean realSeq}}
```

`>>=`に書き換えられた`seq`のバージョンを使って実装され、ここでは`andThen`に書き換えられたバージョンを使うと、最初のエラーのみが利用可能になります：
```lean
{{#example_eval Examples/FunctorApplicativeMonad.lean fakeSeq}}
```
