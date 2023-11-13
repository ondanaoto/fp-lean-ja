# 追加の便利機能

## 共有引数タイプ

同じタイプの複数の引数を取る関数を定義するとき、両方の引数を同じコロンの前に書くことができます。
たとえば、
```lean
{{#example_decl Examples/Monads/Conveniences.lean equalHuhOld}}
```
は以下のように書くことができます。
```lean
{{#example_decl Examples/Monads/Conveniences.lean equalHuhNew}}
```
これは、特に型のシグネチャが大きい場合に便利です。

## 先頭ドット記法

帰納型のコンストラクタは名前空間に含まれます。
これにより、関連する複数の帰納型が同じコンストラクタ名を使用することができますが、プログラムが冗長になる可能性があります。
帰納型が既に分かっているコンテキストでは、コンストラクタの名前の先頭にドットを置くことで名前空間を省略でき、Leanは期待される型を使ってコンストラクタ名を解決します。
たとえば、二分木を反転する関数は次のように書かれます：
```lean
{{#example_decl Examples/Monads/Conveniences.lean mirrorOld}}
```
名前空間を省略することで、大幅に短くすることができますが、Leanコンパイラを含まないコードレビューツールのようなコンテキストでは、プログラムの可読性が低下するコストがあります。
```lean
{{#example_decl Examples/Monads/Conveniences.lean mirrorNew}}
```

期待される表現型を使って名前空間を曖昧にすることは、コンストラクタ以外の名前にも適用されます。
もし `BinTree.empty` が `BinTree` を生成する別の方法として定義されていれば、ドット記法でも使用することができます：
```lean
{{#example_decl Examples/Monads/Conveniences.lean BinTreeEmpty}}

{{#example_in Examples/Monads/Conveniences.lean emptyDot}}
```
```output info
{{#example_out Examples/Monads/Conveniences.lean emptyDot}}
```

## オア・パターン

複数のパターンを許すコンテキスト、例えば `match` 式では、複数のパターンが結果の表現を共有することがあります。
週の日を表すデータ型 `Weekday` があります：
```lean
{{#example_decl Examples/Monads/Conveniences.lean Weekday}}
```

パターンマッチングを使用して、ある日が週末かどうかを確認できます：
```lean
{{#example_decl Examples/Monads/Conveniences.lean isWeekendA}}
```
これは、コンストラクタのドット記法を使うことですでに単純化することができます：
```lean
{{#example_decl Examples/Monads/Conveniences.lean isWeekendB}}
```
両方の週末パターンが同じ結果の表現（`true`）を持っているので、一つに凝縮することができます：
```lean
{{#example_decl Examples/Monads/Conveniences.lean isWeekendC}}
```
これは、引数が名前付けされていないバージョンにさらに簡略化することができます：
```lean
{{#example_decl Examples/Monads/Conveniences.lean isWeekendD}}
```

シーンの背後では、結果の表現が各パターンに単純に複製されます。
これは、パターンが変数をバインドすることができることを意味し、次の例では、同じタイプの値を含む和型から `inl` と `inr` コンストラクタを除去しています：
```lean
{{#example_decl Examples/Monads/Conveniences.lean condense}}
```
結果の表現が複製されるため、パターンによってバインドされる変数が同じタイプである必要はありません。
複数のタイプで機能するオーバーロードされた関数は、異なるタイプの変数をバインドするパターンに対して単一の結果表現を書くために使用されるかもしれません：
```lean
{{#example_decl Examples/Monads/Conveniences.lean stringy}}
```
実際には、結果の表現において参照されるのは、全てのパターンで共有される変数のみです。なぜなら結果は各パターンに対して意味をなす必要があるからです。
`getTheNat` では、`n` のみがアクセス可能であり、`x` や `y` のいずれかを使用しようとする試みはエラーにつながります。
```lean
{{#example_decl Examples/Monads/Conveniences.lean getTheNat}}
```
同様の定義で `x` にアクセスしようとすると、第二のパターンでは `x` が利用できないためエラーが発生します：
```lean
{{#example_in Examples/Monads/Conveniences.lean getTheAlpha}}
```
```output error
{{#example_out Examples/Monads/Conveniences.lean getTheAlpha}}
```

結果の表現がパターンマッチの各分岐に実質的にコピー＆ペーストされるという事実は、いくつかの驚くべき振る舞いにつながる可能性があります。
例えば、以下の定義は受け入れられますが、`inr` バージョンの結果の表現は `str` のグローバル定義を参照します：
```lean
{{#example_decl Examples/Monads/Conveniences.lean getTheString}}
```
この関数を両方のコンストラクタで呼び出すことで、混乱する振る舞いが明らかになります。
最初のケースでは、Leanにどのタイプの `β` であるべきかを伝えるためにタイプ注釈が必要です：
```lean
{{#example_in Examples/Monads/Conveniences.lean getOne}}
```
```output info
{{#example_out Examples/Monads/Conveniences.lean getOne}}
```
二つ目のケースでは、グローバル定義が使用されます：
```lean
{{#example_in Examples/Monads/Conveniences.lean getTwo}}
```
```output info
{{#example_out Examples/Monads/Conveniences.lean getTwo}}
```

オア・パターンを使用すると、`Weekday.isWeekend` のように、いくつかの定義を大幅に簡単にし、その明確さを増すことができます。
しかし、複数のタイプの変数や解離した変数のセットが関係しているときは特に、混乱する振る舞いの可能性があるため、それらを使用する際には注意することが賢明です。