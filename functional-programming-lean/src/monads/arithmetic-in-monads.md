# 例: モナドでの算術

モナドは、副作用を含むプログラムを副作用を持たない言語でエンコードする方法です。
これを読むと、純粋な関数型プログラムが重要な何かを欠いており、通常のプログラムを書くためにプログラマが苦労して飛び跳ねなければならないという認識にすぐさま飛びつきそうです。
しかしながら、`Monad` APIを使用することは、プログラムに構文上のコストを課しますが、2つの重要な利益をもたらします：
 1. プログラムは、それが使用する副作用について、その型で正直でなければなりません。型シグネチャを一目見るだけで、プログラムが行うことの_全て_が分かり、入力内容や返り値だけではありません。
 2. 全ての言語が同じ副作用を提供するわけではありません。例えば、例外を持つ言語だけ、その他の言語は[Iconの複数の値にわたって検索する](https://www2.cs.arizona.edu/icon/)など独特のエキゾチックな副作用や、SchemeやRubyの継続を持っています。モナドは_任意_の副作用をエンコードできるため、プログラマは、言語開発者が提供したものに固執するのではなく、特定のアプリケーションに最適なものを選択できます。

様々なモナドで意味をなすプログラムの一例は算術表現の評価者です。

### 算術表現

算術表現は、リテラルな整数または2つの表現に適用された原始的な２項演算子のいずれかです。演算子は加算、減算、乗算、および除算です：
```lean
{{#example_decl Examples/Monads/Class.lean ExprArith}}
```
表現 `2 + 3` は以下のように表されます：
```lean
{{#example_decl Examples/Monads/Class.lean twoPlusThree}}
```
そして `14 / (45 - 5 * 9)` は以下のように表されます：
```lean
{{#example_decl Examples/Monads/Class.lean exampleArithExpr}}
```

### 表現の評価

表現には除算が含まれるため、ゼロでの除算は未定義のため、評価は失敗する可能性があります。
失敗を表す一つの方法は `Option` を使用することです：
```lean
{{#example_decl Examples/Monads/Class.lean evaluateOptionCommingled}}
```
この定義は二項演算子の両方の分岐の評価から失敗を伝播するために `Monad Option` インスタンスを使用しています。
しかしながら、関数は副表現の評価と副表現の結果に二項演算子を適用する２つの関心事を混在させています。
これは2つの関数に分割することで改善できます：
```lean
{{#example_decl Examples/Monads/Class.lean evaluateOptionSplit}}
```

`{{#example_in Examples/Monads/Class.lean fourteenDivOption}}` を実行すると `{{#example_out Examples/Monads/Class.lean fourteenDivOption}}` となりますが、これはあまり有用なエラーメッセージではありません。
`>>=` を使用して `none` コンストラクタを明示的に扱うのではなく、コードが書かれているため、失敗時にエラーメッセージを提供するためには小さな修正だけが必要です：
```lean
{{#example_decl Examples/Monads/Class.lean evaluateExcept}}
```
唯一の違いは、型シグネチャが `Option` ではなく `Except String` を参照していること、および失敗する場合は `Except.error` を使用することです。
`evaluate` をそのモナドについて多相的にし、それに `applyPrim` を引数として渡すことで、単一の評価者はエラーメッセージの形式の両方を報告することができます：
```lean
{{#example_decl Examples/Monads/Class.lean evaluateM}}
```
`applyPrimOption` を使用すると、`evaluate` の最初のバージョンと同じように動作します：
```lean
{{#example_in Examples/Monads/Class.lean evaluateMOption}}
```
```output info
{{#example_out Examples/Monads/Class.lean evaluateMOption}}
```
同じように、`applyPrimExcept` で使用するとエラーメッセージバージョンと同じように動作します：
```lean
{{#example_in Examples/Monads/Class.lean evaluateMExcept}}
```
```output info
{{#example_out Examples/Monads/Class.lean evaluateMExcept}}
```

コードはまだ改善される可能性があります。
関数 `applyPrimOption` と `applyPrimExcept` は除算の扱いだけが異なりますが、評価者への別のパラメータを抽出することによって異なることを示すことができます：
```lean
{{#example_decl Examples/Monads/Class.lean evaluateMRefactored}}
```

このリファクタリングされたコードでは、２つのコードパスが失敗の扱いにおいてのみ異なることが完全に明らかとなっています。

### 更なる副作用

失敗と例外は評価者で作業する際に興味深い唯一の種類の副作用ではありません。
除算の唯一の副作用は失敗ですが、他の基本的な演算子を表現に加えることは他の副作用を表現する可能性があります。

最初のステップは、プリミティブのデータ型から除算を抽出するという追加のリファクタリングです：
```lean
{{#example_decl Examples/Monads/Class.lean PrimCanFail}}
```
名前 `CanFail` は除算によって導入された副作用が潜在的な失敗であることを示唆しています。

二つ目のステップは、`evaluateM` への除算の処理アーギュメントのスコープを拡大して、特別な演算子を処理することができるようにすることです：
```lean
{{#example_decl Examples/Monads/Class.lean evaluateMMorePoly}}
```

#### 副作用無し

型 `Empty` はコンストラクタを持たず、値もない、ScalaやKotlinの `Nothing` 型のようです。
ScalaとKotlinでは、`Nothing` はプログラムをクラッシュさせる関数や、例外を投げる、または常に無限ループに陥る計算を表すことができます。
関数またはメソッドへの `Nothing` 型の引数は、適切な引数値が決して存在しないため、使用されないコードを指示します。
Leanは無限ループや例外をサポートしていませんが、`Empty` は型システムに対して関数を呼び出すことができないことを示唆するために依然として役立ちます。
型のコンストラクタを持たない式 `E` の場合、構文 `nomatch E` はLeanに現在の式が結果を返す必要がないことを示唆します、なぜならそれは決して呼び出され得ないからです。

`Empty` を `Prim` へのパラメータとして使用する場合、`Prim.plus`、`Prim.minus`、`Prim.times` を超えた追加のケースがないことを示します。`Empty` の型の値を `Prim.other` コンストラクタに配置することは不可能であるためです。
二つの整数に `Empty` 型の演算子を適用する関数が呼び出されることは決してないため、結果を返す必要はありません。
したがって、どんなモナドでも使用できます：
```lean
{{#example_decl Examples/Monads/Class.lean applyEmpty}}
```
これは `Id`、アイデンティティモナドと一緒に使用され、全く副作用のない表現を評価することができます：
```lean
{{#example_in Examples/Monads/Class.lean evalId}}
```
```output info
{{#example_out Examples/Monads/Class.lean evalId}}
```

#### 非決定的探索

ゼロでの除算に遭遇した際に単に失敗する代わりに、異なる入力を試してバックトラックすることも理にかなっています。
適切なモナドを与えられた場合、全く同じ `evaluateM` は失敗しない解の_セット_に対して非決定的探索を実行することができます。
この場合、除算に加えて、非失敗の結果を探索する間にいずれかの引数を選択するように評価者に指示する式言語への `choose` 関数の追加が必要です。

評価者の結果は今や値のマルチセットであり、単一の値ではありません。
マルチセットへの評価規則は以下の通りです：
 * 定数 \\( n \\) はシングルトンセット \\( \{n\} \\) に評価されます。
 * 除算以外の算術演算子は演算子のデカルト積の各ペアに対して呼び出され、したがって \\( X + Y \\) は \\( \\{ x + y \\mid x ∈ X, y ∈ Y \\} \\) に評価されます。
 * 除算 \\( X / Y \\) は \\( \\{ x / y \\mid x ∈ X, y ∈ Y, y ≠ 0\\} \\) に評価されます。つまり、\\( Y \\) の全ての \\( 0 \\) の値は省かれます。
 * 選択 \\( \\mathrm{choose}(x, y) \\) は \\( \\{ x, y \\} \\) に評価されます。

たとえば、\\( 1 + \\mathrm{choose}(2, 5) \\) は \\( \\{ 3, 6 \\} \\) に評価され、\\(1 + 2 / 0 \\) は \\( \\{\\} \\) に評価され、\\( 90 / (\\mathrm{choose}(-5, 5) + 5) \\) は \\( \\{ 9 \\} \\) に評価されます。
真のセットの代わりにマルチセットを使用することは、要素の一意性のチェックを省略することでコードを単純化します。

非決定的な影響を表すモナドは、回答がない状況と、少なくとも1つの回答とそれ以外の全ての回答がある状況を表すことができなければなりません：
```lean
{{#example_decl Examples/Monads/Many.lean Many}}
```
このデータ型は `List` と非常によく似ています。
違いは、`cons` がリストの残りを保存するところで、`more` が次の値を要求時に計算する関数を保存している点です。
これは `Many` の消費者が見つけた結果のいくつかを得た時点で探索を止めることができることを意味します。

単一の結果はこれ以上の結果を返さない `more` コンストラクタで表されます：
```lean
{{#example_decl Examples/Monads/Many.lean one}}
```
２つのマルチセットの結果のユニオンは、最初のマルチセットが空かどうかを調べることによって計算できます。
それが空の場合、２つ目のマルチセットがユニオンです。
それが空でない場合、ユニオンは最初のマルチセットの最初の要素に続いて最初のマルチセットの残りと２つ目のマルチセットのユニオンで構成されます：
```lean
{{#example_decl Examples/Monads/Many.lean union}}
```

開始する検索プロセスで値のリストがあると便利なことがあります。
`Many.fromList` はリストを結果のマルチセットに変換します：
```lean
{{#example_decl Examples/Monads/Many.lean fromList}}
```
同様に、検索が指定された後に、いくつかの値または全ての値を抽出すると便利なことがあります：
```lean
{{#example_decl Examples/Monads/Many.lean take}}
```

`Monad Many` インスタンスには `bind` 演算子が必要です。
非決定的な検索で２つの操作を連続するには、最初のステップから全ての可能性を取り、それぞれに残りのプログラムを実行し、結果のユニオンを取ることが含まれます。
言い換えると、最初のステップが３つの可能な答えを返す場合、それぞれについて次のステップを試みる必要があります。
２つ目のステップは各入力に対していくつでも答えを返すことができるので、ユニオンを取ることは全探索空間を表します。
```lean
{{#example_decl Examples/Monads/Many.lean bind}}
```

`Many.one` と `Many.bind` はモナド契約を遵守します。
`Many.bind (Many.one v) f` が `f v` と同じであることを確認するために、式を可能な限り評価してみます：
```lean
{{#example_eval Examples/Monads/Many.lean bindLeft}}
```
空のマルチセットは `union` の右のアイデンティティであり、答えは `f v` と等価です。すべての関数 `f` について、`fun x => f x` は `f` と同じなので、最初の部分の契約が満たされます。
`Reader.bind r Reader.pure` が `r` と同じであることをチェックするには、同様の技術が機能します：
```lean
{{#example_eval Examples/Monads/Class.lean ReaderMonad2}}
```
リーダアクション `r` 自体が関数であるため、これは `r` と同じです。
結合性をチェックするために、`{{#example_eval Examples/Monads/Class.lean ReaderMonad3a 0}}` と `{{#example_eval Examples/Monads/Class.lean ReaderMonad3b 0}}` の両方に対して同じことを行います：
```lean
{{#example_eval Examples/Monads/Class.lean ReaderMonad3a}}
```

```lean
{{#example_eval Examples/Monads/Class.lean ReaderMonad3b}}
```

したがって、`Monad (Reader ρ)` インスタンスが正当化されています：
```lean
{{#example_decl Examples/Monads/Class.lean MonadReaderInst}}
```

式評価器に渡されるカスタム環境は、ペアのリストとして表現できます：
```lean
{{#example_decl Examples/Monads/Class.lean Env}}
```
例えば、`exampleEnv` には最大値と剰余の機能が含まれています：
```lean
{{#example_decl Examples/Monads/Class.lean exampleEnv}}
```

Leanにはすでに `List.lookup` という関数があり、キーに対応する値をペアのリストから見つけるため、`applyPrimReader` は環境内にカスタム関数が存在するかどうかをチェックするだけで十分です。もし関数が見知らぬものであれば `0` を返します：
```lean
{{#example_decl Examples/Monads/Class.lean applyPrimReader}}
```

`evaluateM` を `applyPrimReader` と式を使って使用すると、環境を期待する関数の結果が得られます。
幸いにも、`exampleEnv` が使用できます：
```lean
{{#example_in Examples/Monads/Class.lean readerEval}}
```
```output info
{{#example_out Examples/Monads/Class.lean readerEval}}
```

`Many` のように、`Reader` はほとんどの言語でエンコードするのが困難な効果の一例ですが、型クラスとモナドを使うことで他のどんな効果と同じくらい便利になります。
Common Lisp、Clojure、Emacs Lispで見られる動的または特別な変数は、`Reader` のように使用できます。
同様に、SchemeやRacketのパラメータオブジェクトは、`Reader` に完全に対応している効果です。
Kotlinのコンテキストオブジェクトの慣用句は同様の問題を解決することができますが、それは自動的に関数引数を渡す手段であるため、この慣用句は言語の効果よりもリーダモナドとしてのエンコーディングに似ています。

## 練習

### 契約のチェック

`State σ` と `Except ε` についてモナド契約をチェックしてください。

### 失敗を伴うリーダー

リーダーモナドの例を適応させて、カスタムオペレータが定義されていない場合に失敗を示すことができるようにしてください。つまり、ゼロを単に返すのではなく、これらの定義を与えられた場合に：
```lean
{{#example_decl Examples/Monads/Class.lean ReaderFail}}
```
次のことを行います：
 1. 適切な `pure` と `bind` 関数を書く
 2. これらの関数が `Monad` 契約を満たすことを確認する
 3. `ReaderOption` と `ReaderExcept` のための `Monad` インスタンスを書く
 4. 適切な `applyPrim` 演算子を定義し、いくつかの例題の式に `evaluateM` でテストする

### トレース評価器

`WithLog` 型は、評価器と共に使用されることで、特定の操作のオプションのトレースを追加することができます。
特に、`ToTrace` 型は、与えられたオペレータをトレースする信号として機能することができます：
```lean
{{#example_decl Examples/Monads/Class.lean ToTrace}}
```

トレース評価器の場合、式は型 `Expr (Prim (ToTrace (Prim Empty)))` を持つべきです。これは、追跡されたそれぞれのバージョンと増やされた追加、減少、および乗算の表現内のオペレータであり、内部の引数が `Empty` は `trace` の中にそれ以上の特別なオペレータがないことを示すためです。

以下を行います：
 1. `Monad (WithLog logged)` インスタンスを実装する
 2. トレースされたオペレータをその引数に適用する `{{#example_in Examples/Monads/Class.lean applyTracedType}}` 関数を書き、型 `{{#example_out Examples/Monads/Class.lean applyTracedType}}` で運用する

この練習が正しく完了している場合、
```lean
{{#example_in Examples/Monads/Class.lean evalTraced}}
```
は次の結果になります：
```output info
{{#example_out Examples/Monads/Class.lean evalTraced}}
```

ヒント：結果のログには `Prim Empty` の型の値が表示されることがあります。`#eval` の結果としてそれらを表示するためには、以下のインスタンスが必要です：
 ```lean
 {{#example_decl Examples/Monads/Class.lean ReprInstances}}
 ```