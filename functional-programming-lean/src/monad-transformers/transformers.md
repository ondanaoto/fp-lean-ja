# モナド構築キット

`ReaderT`は有用なモナド変換子であり、それが唯一存在するわけではありません。
このセクションでは、追加の変換子について説明します。
各モナド変換子は以下のもので構成されます：
 1. モナドを引数として受け取る定義またはデータ型 `T`。
    型は `(Type u → Type v) → Type u → Type v` のようになりますが、モナドの前に追加の引数を受け付けることがあります。
 2. `Monad m` のインスタンスに依存する `T m` の `Monad` インスタンス。これにより、変換されたモナドをモナドとして使用することができます。
 3. 任意のモナド `m` において、型 `m α` のアクションを型 `T m α` のアクションに変換する `MonadLift` インスタンス。これにより、基礎となるモナドのアクションを、変換されたモナドで使用できます。

さらに、変換子の `Monad` インスタンスは、少なくとも基礎となる `Monad` インスタンスがそうである場合、`Monad` の契約を守るべきです。
加えて、`monadLift (pure x)` は変換されたモナドにおける `pure x` と等価でなければならず、`monadLift` は `bind` 上で分配可能であるべきで、`monadLift (x >>= f)` は `monadLift x >>= fun y => monadLift (f y)` と等価でなければなりません。

多くのモナド変換子は、モナドにおいて使用可能な実際の効果を記述する `MonadReader` のスタイルの型クラスを追加で定義します。
これにより、より大きな柔軟性が提供されることがあります：プログラムはインターフェイスのみに依存するように記述され、特定の変換子によって基礎となるモナドが実装されることに制約されることはありません。
型クラスは、プログラムがその要件を表現する方法であり、モナド変換子はこれらの要件を満たす便利な方法です。

## `OptionT` を用いた失敗

`Option` モナドによって代表される失敗と、`Except` モナドによって代表される例外の両方には、対応する変換子があります。
`Option` の場合、失敗をモナドに追加することは、それ以外では型 `α` の値を含む箇所に型 `Option α` の値を含むようにすることで実現されます。
たとえば、`IO (Option α)` は常に型 `α` の値を返すわけではない `IO` アクションを表します。
これは、モナド変換子 `OptionT` の定義を示唆しています：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean OptionTdef}}
```

`OptionT` のアクションの例として、ユーザーに質問をするプログラムを考えます。
関数 `getSomeInput` は入力行を求め、両端から空白を削除します。
結果のトリムされた入力が空でない場合は、それが返されますが、空白文字がない場合は関数は失敗します：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean getSomeInput}}
```
この特定のアプリケーションは、ユーザーを名前と彼らの好きな甲虫の種類で追跡します：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean UserInfo}}
```
ユーザーに入力を求めることは、`IO` のみを使用する関数よりも煩雑ではありません：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean getUserInfo}}
```
しかし、関数が `IO` ではなく `OptionT IO` の文脈で実行されるため、最初の `getSomeInput` 呼び出しでの失敗は、甲虫に関する質問に到達することなく `getUserInfo` 全体の失敗を引き起こします。
メイン関数 `interact` は、`getUserInfo` を純粋な `IO` コンテキストで呼び出し、内部の `Option` にマッチングすることで、呼び出しが成功したか失敗したかを確認します：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean interact}}
```

### モナドインスタンス

モナドインスタンスを記述する際には難しさが明らかになります。
型に基づいて `pure` は基礎となるモナド `m` の `pure` と共に使用するべきです。
`Option` の `bind` が最初の引数に分岐し `none` を伝播するのと同様に、`OptionT` の `bind` は最初の引数で構成されるモナドアクションを実行し、結果に分岐した後 `none` を伝播するべきです。
この概略に従うと以下の定義が生じますが、Leanは受け入れません：
```lean
{{#example_in Examples/MonadTransformers/Defs.lean firstMonadOptionT}}
```
エラーメッセージには謎めいた型の不一致が表示されます：
```output error
{{#example_out Examples/MonadTransformers/Defs.lean firstMonadOptionT}}
```
ここでの問題は、Leanが`pure`の周辺使用に対する間違った`Monad`インスタンスを選択していることです。
型注釈を使用してLeanを正しい`Monad`インスタンスに導くために同様のエラーが`bind`の定義にも発生します：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean MonadOptionTAnnots}}
```
この解決策は機能しますが、洗練されておらずコードが少し騒がしくなります。

別の解決策は、正しいインスタンスにLeanを導く型シグネチャを持つ関数を定義することです。
実際には、`OptionT` を構造体として定義することも可能でした：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean OptionTStructure}}
```
これにより問題が解決されます。なぜなら、コンストラクタ `OptionT.mk` とフィールドアクセス `OptionT.run` が正しいインスタンスに型クラスインターフェースを導くからです。
ただし、構造体の値を使ったコードを実行するたびに、割り当ておよび取り消しが必要になり、それに対して直接の定義はコンパイル時のみの機能です。
最善の世界は `OptionT.mk` と `OptionT.run` の役割として機能するが、直接定義で機能する関数を定義することによって達成できます：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean FakeStructOptionT}}
```
これらの関数は入力されたものを変更せずに返しますが、`OptionT`のインターフェイスを提示することを意図したコードと、基礎となるモナド `m` のインターフェイスを提示することを意図したコードの間に境界を示しています。
これらの補助関数を使用して、`Monad`インスタンスがより読みやすくなります：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean MonadOptionTFakeStruct}}
```
ここでは `OptionT.mk` の使用が引数を `m` のインターフェイスを使用するコードとして見なすべきことを示し、これによってLeanは正しい `Monad` インスタンスを選択することができます。

モナドインスタンスを定義した後、モナド契約が満たされていることを確認することは良いアイデアです。
最初のステップは `bind (pure v) f` が `f v` と同じであることを示すことです。
以下がステップです：
{{#equations Examples/MonadTransformers/Defs.lean OptionTFirstLaw}}

二つ目の規則は `bind w pure` が `w` と同じであると述べています。
これを示すためには、`bind` と `pure` の定義を展開し、以下を示します：
```lean
OptionT.mk do
    match ← w with
    | none => pure none
    | some v => pure (some v)
```
このパターンマッチで、両方のケースの結果はパターンに対して同じですが、`pure` がそれを取り巻いています。
つまり、これは `w >>= fun y => pure y` に相当し、これは `m` の二つ目のモナドルールのインスタンスです。

最終ルールは `bind (bind v f) g` が `bind v (fun x => bind (f x) g)` と同じであると述べています。
これも同様の方法で `bind` と `pure` の定義を展開して確認することができ、その後基礎となるモナド `m` に委ねます。

### `Alternative` インスタンス

`OptionT` を使用する便利な方法の一つは、`Alternative` 型クラスを通じてです。
成功した返り値はすでに `pure` によって示されており、`Alternative` の `failure` および `orElse` メソッドは、複数のサブプログラムから最初の成功した結果を返すプログラムを記述する方法を提供します：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean AlternativeOptionT}}
```


### リフティング

`m` から `OptionT m` へのアクションをリフティングすることは、計算結果に `some` をラップするだけで必要とされます：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean LiftOptionT}}
```


## 例外

`Except` のモナド変換子バージョンは `Option` のモナド変換子バージョンに非常に似ています。
型 `ε` の例外を型 `m α` のモナドアクションに追加することは `α` に例外を追加し、型 `m (Except ε α)` を生み出すことで実現されます：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean ExceptT}}
```
`OptionT` は型チェッカーを正しい `Monad` インスタンスに導く `mk` と `run` 関数を提供します。
このテクニックは `ExceptT` にも役立ちます：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean ExceptTFakeStruct}}
```
`ExceptT` の `Monad` インスタンスも `OptionT` のインスタンスと非常に似ています。
唯一の違いは、特定のエラー値を伝播すること、そして `none` ではないことです：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean MonadExceptT}}
```

`ExceptT.mk` と `ExceptT.run` の型シグネチャには、`α` と `ε` の宇宙レベルを明示的に注釈しているという微妙な詳細が含まれています。
それらが明示的に注釈されていない場合、Leanはそれらに異なる多型宇宙変数があるより一般的な型シグネチャを生成し、しかし `ExceptT` の定義は `m` に引数として両方を与えることができるために、それらが同じ宇宙にあることを期待しています。
これにより `Monad` インスタンスで宇宙レベルのソルバーが機能するソリューションを見つけることができずに問題が発生する可能性があります：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean ExceptTNoUnis}}

{{#example_in Examples/MonadTransformers/Defs.lean MonadMissingUni}}
```
```output error
{{#example_out Examples/MonadTransformers/Defs.lean MonadMissingUni}}
```
この種のエラーメッセージは一般に、制約が不足している宇宙変数によって引き起こされます。
診断するのはトリッキーなことがありますが、最初のステップとしては、一部の定義で再利用される宇宙変数が他の定義では再利用されないことを探すことです。

`Except` と異なり、`Except` データ型は通常データ構造として使用されません。
それは常にその `Monad` インスタンスを使った制御構造として使用されます。
これは `Except ε` アクションを `ExceptT ε m` にリフティングすることが合理的であることを意味し、 `m` のアクションから `m` のエフェクトがないため、`m` の `pure` でそれらをラップする必要があります：
```lean
{{#example_decl Examples/MonadTransformers/Defs.lean ExceptTLiftExcept}}
```
`m` からのアクションには例外が含まれていないため、その値は `Except.ok` にラップされるべきです。
これは、`Functor` が `Monad` のスーパークラスであるという事実を利用して、任意の