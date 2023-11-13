# 代替案


## 失敗からの回復

`Validate`は、入力が受け入れられる方法が複数ある状況でも使用できます。
入力フォーム`RawInput`に対して、レガシーシステムからの慣習を実装したビジネスルールの別のセットは以下の通りです：

 1. すべての人間のユーザーは、4桁の生年を提供しなければなりません。
 2. 1970年以前に生まれたユーザーは、不完全な旧記録のため、名前を提供する必要はありません。
 3. 1970年以降に生まれたユーザーは、名前を提供しなければなりません。
 4. 企業は、出生年として`"FIRM"`を入力し、企業名を提供する必要があります。
 
1970年に生まれたユーザーについては特に規定がなされていません。
彼らには、諦めるか、出生年を偽るか、または電話をかけることが期待されています。
会社はこれをビジネスを行う上での許容可能なコストと考えています。

以下の帰納的型は、これらの規則から生成される値を捉えます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean LegacyCheckedInput}}
```

しかし、これらのルールのバリデータはより複雑であり、３つのケースすべてに対処しなければなりません。
それは一連のネストした`if`式として書くこともできますが、３つのケースを独立して設計し、その後に統合する方が簡単です。
これには、エラーメッセージを保持しながら、失敗からの回復が可能な手段が必要です：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean ValidateorElse}}
```

この失敗からの回復パターンは十分に一般的であり、Leanにはそれ用の組み込み構文があります。これは`OrElse`という型クラスに付随しています：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean OrElse}}
```
式`{{#example_in Examples/FunctorApplicativeMonad.lean OrElseSugar}}`は`{{#example_out Examples/FunctorApplicativeMonad.lean OrElseSugar}}`の略です。
`Validate`の`OrElse`インスタンスにより、この構文をエラー回復に使用することができます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean OrElseValidate}}
```

`LegacyCheckedInput`のバリデータは、各コンストラクタのバリデータから構築することができます。
企業のルールは、出生年は文字列`"FIRM"`でなければならず、名前は非空でなければなりません。
しかし、コンストラクタ`LegacyCheckedInput.company`には出生年の表現がまったくないため、`<*>`を使ってこれを実行する簡単な方法はありません。
重要なのは、その引数を無視する関数を`<*>`と一緒に使用することです。

型にこの事実を何も記録せずに真偽条件が成り立つことを確認するには、`checkThat`を使用できます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean checkThat}}
```
この`checkCompany`の定義は`checkThat`を使用して、その結果の`Unit`値を廃棄します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean checkCompanyProv}}
```

しかしながら、この定義はかなり雑音が多いです。
２つの方法で簡略化できます。
最初の方法は、最初の`<*>`を使う代わりに、最初の引数によって返される値を自動的に無視する特殊バージョンである`*>`を使用することです。
この演算子もまた、`SeqRight`という型クラスによって制御されており、`{{#example_in Examples/FunctorApplicativeMonad.lean seqRightSugar}}`は`{{#example_out Examples/FunctorApplicativeMonad.lean seqRightSugar}}`の糖衣構文です：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean ClassSeqRight}}
```
`seqRight`のデフォルト実装は`seq`の観点からです：`seqRight (a : f α) (b : Unit → f β) : f β := pure (fun _ x => x) <*> a <*> b ()`。

`seqRight`を使用すると、`checkCompany`はよりシンプルになります：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean checkCompanyProv2}}
```
さらに簡略化することが可能です。
すべての`Applicative`において、`pure F <*> E`は`f <$> E`と等価です。
言い換えれば、`pure`を使って`Applicative`型に置かれた関数を適用するために`seq`を使用することは過剰であり、その関数は`Functor.map`を使って適用できたはずです。
この簡略化により：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean checkCompany}}
```

`LegacyCheckedInput`の残りの２つのコンストラクタは、そのフィールドにサブタイプを使用しています。
サブタイプをチェックするための汎用ツールは、これらを読みやすくするでしょう：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean checkSubtype}}
```
関数の引数リストでは、型クラス`[Decidable (p v)]`が引数`v`と`p`の指定の後で発生することが重要です。
さもなければ、これは手動で提供された値ではなく、追加の一連の自動的な暗黙の引数を指すことになるでしょう。
`Decidable`インスタンスは、`if`を使用して命題`p v`をチェックすることを可能にするものです。

２つの人間のケースは追加のツールを必要としません：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean checkHumanBefore1970}}

{{#example_decl Examples/FunctorApplicativeMonad.lean checkHumanAfter1970}}
```

３つのケースのバリデータは`<|>`を使用して組み合わせることができます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean checkLegacyInput}}
```

成功したケースは予想通り`LegacyCheckedInput`のコンストラクタを返します：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean trollGroomers}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean trollGroomers}}
```
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean johnny}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean johnny}}
```
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean johnnyAnon}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean johnnyAnon}}
```

最悪の入力はすべての失敗を返します：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean allFailures}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean allFailures}}
```


## `Alternative` クラス

多くの型は、失敗と回復の概念をサポートしています。
[さまざまなモナドで算術表現を評価する](../monads/arithmetic.md#nondeterministic-search)セクションの`Many`モナドはこのような型の一つですし、`Option`もそうです。
両方とも失敗をサポートしていますが、理由を提供することなく（例えば`Except`や`Validate`のように、何が間違っているかの何らかの指示が必要です）。

`Alternative`クラスは、失敗と回復のための追加の演算子を持つ適用関数型子を記述します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean FakeAlternative}}
```
`Add α`の実装者が自動的に`HAdd α α α`インスタンスを得るのと同様に、`Alternative`の実装者は`OrElse`インスタンスを無料で取得します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean AltOrElse}}
```

`Option`の`Alternative`の実装は、最初のnone-`none`引数を保持します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean AlternativeOption}}
```
同様に、`Many`の実装は`Many.union`の一般的な構造に従いますが、怠惰を誘発する`Unit`パラメータの配置が異なるために若干の違いがあります：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean AlternativeMany}}
```

他の型クラスと同様に、`Alternative`は`Alternative`を実装する任意の適用関数型子に対して動作するさまざまな操作の定義を可能にします。
最も重要なのは、否定的な命題が偽であるときに`failure`を引き起こす`guard`です：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean guard}}
```
これはモナドプログラムで実行を早期に終了させるために非常に有効です。
`Many`では、自然数のすべての偶数の除数を計算する以下のプログラムで、検索の全枝をフィルタリングするために使用できます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean evenDivisors}}
```
`20`で実行すると予想される結果が得られます：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean evenDivisors20}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean evenDivisors20}}
```


## 練習問題

### バリデーションの親しみやすさの向上

`<|>`を使用する`Validate`プログラムから返されるエラーは、リストに含まれるエラーは単に_いくつかの_コードパスを通じて到達可能であることを意味するため、読むのが難しいことがあります。
より構造化されたエラーレポートを使用することで、ユーザーをより正確にガイドすることができます：

 * `Validate.error`の`NonEmptyList`を素の型変数に置き換え、そして`Applicative (Validate ε)`および`OrElse (Validate ε α)`インスタンスの定義を更新して、`Append ε`インスタンスが利用可能であるだけを要求します。
 * `Validate.mapErrors : Validate ε α → (ε → ε') → Validate ε' α`という関数を定義し、検証実行中のすべてのエラーを変換します。
 * エラーを代表するデータ型`TreeError`を使い、３つの代替案を通じたパスを追跡するようにレガシーバリデーションシステムを書き換えます。
 * `report : TreeError → String`という関数を書き、`TreeError`の蓄積された警告とエラーのユーザーフレンドリーなビューを出力します。

```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean TreeError}}
```