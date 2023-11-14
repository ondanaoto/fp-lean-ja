# 構造と継承

`Functor`、`Applicative`、そして `Monad` の完全な定義を理解するために、Lean の別の機能が必要です：構造体継承です。
構造体継承を使用すると、一つの構造体タイプが、追加のフィールドとともに、別のもののインターフェースを提供することができます。
これは、明白な分類関係を持つ概念をモデル化する際に役立ちます。
例えば、神話上の生き物のモデルを考えてみましょう。
その中には大きなものも小さなものもいます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean MythicalCreature}}
```
背後では、`MythicalCreature` 構造体の定義によって、`mk` と呼ばれる単一のコンストラクターを持つ帰納型が作成されます：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean MythicalCreatureMk}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean MythicalCreatureMk}}
```
同様に、実際にコンストラクターからフィールドを取り出す `MythicalCreature.large` 関数が作成されます：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean MythicalCreatureLarge}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean MythicalCreatureLarge}}
```

ほとんどの古い物語では、どんなモンスターでも何かしらの方法で倒すことができます。
モンスターの説明には、それが大きいかどうかと共に、この情報を含める必要があります：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean Monster}}
```
見出しの `extends MythicalCreature` は、すべてのモンスターもまた神話上の生き物であることを示しています。
`Monster` を定義するためには、`MythicalCreature` および `Monster` からのフィールドを両方提供する必要があります。
トロールは、日光に弱い大きなモンスターです：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean troll}}
```

内部では、継承はコンポジションを使用して実装されます。
コンストラクター `Monster.mk` は `MythicalCreature` を引数として取ります：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean MonsterMk}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean MonsterMk}}
```
各新しいフィールドの値を抽出する関数を定義する他に、`{{#example_in Examples/FunctorApplicativeMonad.lean MonsterToCreature}}` という型の関数 `{{#example_out Examples/FunctorApplicativeMonad.lean MonsterToCreature}}` が定義されます。
これは、基になる生き物を抽出するために使用できます。

Lean での継承階層の移動は、オブジェクト指向言語におけるアップキャストと同じものではありません。
アップキャスト演算子は、派生クラスの値を親クラスのインスタンスとして扱うものですが、値はそのアイデンティティと構造を保持します。
しかし、Lean では、継承階層を上に移動することが、実際には基になる情報を消去することを意味します。
このことは、`troll.toMythicalCreature` を評価した結果を考慮することで明らかになります：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean evalTrollCast}}
```
```output info
{{#example_out Examples/FunctorApplicativeMonad.lean evalTrollCast}}
```
`MythicalCreature` のフィールドのみが残ります。


`where` 構文と同様に、構造体継承でもフィールド名を伴う波括弧記法が機能します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean troll2}}
```
しかし、基になるコンストラクターに委譲する匿名の山括弧記法は、内部の詳細を明らかにします：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean wrongTroll1}}
```
```output error
{{#example_out Examples/FunctorApplicativeMonad.lean wrongTroll1}}
```
必要なのは追加の1組の山括弧であり、これにより `true` に `MythicalCreature.mk` が呼び出されます：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean troll3}}
```


Leanのドット記法は継承を考慮に入れることができます。
つまり、既存の `MythicalCreature.large` を `Monster` で使用でき、Lean は自動的に `{{#example_in Examples/FunctorApplicativeMonad.lean MonsterToCreature}}` を `MythicalCreature.large` 呼び出しの前に挿入します。
しかしこれは、ドット記法を使用している場合のみ発生し、フィールドルックアップ関数を通常の関数呼び出し構文を使用して適用すると、型エラーになります：
```lean
{{#example_in Examples/FunctorApplicativeMonad.lean trollLargeNoDot}}
```
```output error
{{#example_out Examples/FunctorApplicativeMonad.lean trollLargeNoDot}}
```
ユーザー定義の関数に対しても、ドット記法は小さな生き物、それは大きくないものを指します：
```lean
{{#example_decl Examples/FunctorApplicativeMonad.lean small}}
```
`{{#example_in Examples/FunctorApplicativeMonad.lean smallTroll}}` を評価すると `{{#example_out Examples/FunctorApplicativeMonad.lean smallTroll}}` が出力され、`{{#example_in Examples/FunctorApplicativeMonad.lean smallTrollWrong}}` を評価しようとすると以下のような結果になります：
```output error
{{#example_out Examples/FunctorApplicativeMonad.lean smallTrollWrong}}
```