# インデックス付き族

多相的帰納型は型引数を取ります。
例えば、`List` はリスト内のエントリの型を決定する引数を取り、`Except` は例外または値の型を決定する引数を取ります。
これらの型引数は、データ型の構造の中で同じであり、_パラメータ_と呼ばれています。

しかし、帰納型への引数がすべての構造で同じである必要はありません。
構造によって型への引数が異なる帰納型は、_インデックス付き族_と呼ばれ、変化する引数は_インデックス_と呼ばれます。
インデックス付き族の「こんにちは世界」は、リストの長さに加えてエントリの型を含むリストの型で、通常「ベクトル」と呼ばれます：
```lean
{{#example_decl Examples/DependentTypes.lean Vect}}
```

関数宣言は、コロンの前にいくつかの引数を取り、それが定義全体で利用可能であることを示します。また、コロンの後に引数を取り、それらにパターンマッチングしたいという意思を示したり、ケースバイケースで関数を定義します。
帰納的データ型も同様の原則を持っています：引数 `α` は、データ型宣言の先頭、コロンの前に名前付けされ、これは `Vect` のすべての出現で最初の引数として提供されなければならないパラメータであることを示しています。一方、`Nat` 引数はコロンの後に現れ、これはインデックスであり、場合によっては異なる可能性があることを示します。
実際には、`nil` と `cons` のコンストラクタ宣言での `Vect` の三つの出現は、最初の引数として `α` を一貫して提供していますが、二つ目の引数はそれぞれ異なります。

`nil` の宣言は、それが型 `Vect α 0` のコンストラクタであることを述べています。
これは、`Vect.nil` を `Vect String 3` が期待される文脈で使用することは型エラーであることを意味します。ちょうど、`[1, 2, 3]` が `List String` を期待する文脈で型エラーであるように：
```lean
{{#example_in Examples/DependentTypes.lean nilNotLengthThree}}
```
```output error
{{#example_out Examples/DependentTypes.lean nilNotLengthThree}}
```
この例での `0` と `3` の不一致は、`0` と `3` が自体型ではないにも関わらず、他のどの型の不一致としても、全く同じ役割を果たしています。

インデックス付き族は、異なるインデックス値で異なるコンストラクタが使用可能になるため、型の_族_と呼ばれます。
ある意味では、インデックス付き族は型ではなく、関連する型のコレクションであり、インデックスの値を選択することによってコレクションから型が選ばれます。
`Vect` にインデックス `5` を選択すると、コンストラクタ `cons` のみが使用可能ですし、`0` を選択すると `nil` のみが使用可能になります。

インデックスがまだ不明である場合（例えば、それが変数であるために）、それが分かるまでどのコンストラクタも使用することができません。
長さ `n` を使用すると、`Vect.nil` も `Vect.cons` も使用できないため、変数 `n` は `0` か `n + 1` に一致する `Nat` を示すべきかわかりません：
```lean
{{#example_in Examples/DependentTypes.lean nilNotLengthN}}
```
```output error
{{#example_out Examples/DependentTypes.lean nilNotLengthN}}
```
```lean
{{#example_in Examples/DependentTypes.lean consNotLengthN}}
```
```output error
{{#example_out Examples/DependentTypes.lean consNotLengthN}}
```

リストの長さをその型の一部とすることは、型をより情報量の多いものにします。
例えば、`Vect.replicate` は指定された値のコピーが何個も含まれる `Vect` を作成する関数です。
これを正確に言う型は：
```lean
{{#example_in Examples/DependentTypes.lean replicateStart}}
```
引数 `n` は結果の長さとして現れます。
アンダースコアのプレースホルダーに関連付けられたメッセージは、手がかりとなるタスクを述べています：
```output error
{{#example_out Examples/DependentTypes.lean replicateStart}}
```

インデックス付き族を扱うとき、Leanがコンストラクタのインデックスが期待される型のインデックスと一致していることを確認できる時にのみ、コンストラクタを適用することができます。
しかし、どちらのコンストラクタにも `n` に一致するインデックスはありません—`nil` は `Nat.zero` に、そして `cons` は `Nat.succ` に一致します。
先述の型エラーの例のように、変数 `n` は引数として関数に提供される `Nat` に基づいて、いずれかの値になる可能性があります。
解決策は、パターンマッチングを使用して、両方の可能性のあるケースを考慮することです：
```lean
{{#example_in Examples/DependentTypes.lean replicateMatchOne}}
```
期待される型に `n` が出現するので、`n` にパターンマッチングを行うことは、マッチの二つのケースで期待される型を_洗練_します。
最初のアンダースコアでは、期待される型は `Vect α 0` になっています：
```output error
{{#example_out Examples/DependentTypes.lean replicateMatchOne}}
```
二つ目のアンダースコアでは、それは `Vect α (k + 1)` になっています：
```output error
{{#example_out Examples/DependentTypes.lean replicateMatchTwo}}
```
パターンマッチングが値の構造を発見するだけでなく、プログラムの型を洗練するとき、それは_依存型パターンマッチング_と呼ばれます。

洗練された型により、コンストラクタを適用することが可能になります。
最初のアンダースコアは `Vect.nil` にマッチし、二つ目は `Vect.cons` にマッチします：
```lean
{{#example_in Examples/DependentTypes.lean replicateMatchFour}}
```
`.cons` の下の最初のアンダースコアは型 `α` であるべきです。
使用可能な `α` があります、すなわち `x`：
```output error
{{#example_out Examples/DependentTypes.lean replicateMatchFour}}
```
二つ目のアンダースコアは `Vect α k` であるべきで、これは `replicate` への再帰的な呼び出しによって生成されます：
```output error
{{#example_out Examples/DependentTypes.lean replicateMatchFive}}
```
これが `replicate` の最終定義です：
```lean
{{#example_decl Examples/DependentTypes.lean replicate}}
```

関数を書くときに有益な情報提供をするだけでなく、`Vect.replicate` の情報量の多い型は、クライアントのコードがソースコードを読まなくても多くの予期しない関数を排除することを可能にします。
`List` に対する `replicate` のバージョンは、間違った長さのリストを生成するかもしれません：
```lean
{{#example_decl Examples/DependentTypes.lean listReplicate}}
```
しかし、`Vect.replicate` でこの間違いを犯すことは型エラーです：
```lean
{{#example_in Examples/DependentTypes.lean replicateOops}}
```
```output error
{{#example_out Examples/DependentTypes.lean replicateOops}}
```


関数 `List.zip` は二つのリストを組み合わせることで、最初のリストの最初のエントリと二つ目のリストの最初のエントリをペアリングし、最初のリストの2番目のエントリを二つ目のリストの2番目のエントリとペアリングし、以下同様に続けます。
`List.zip` は米国オレゴン州の最も高い3つのピークとデンマークの最も高い3つのピークを組み合わせるために使用できます：
```lean
{{#example_in Examples/DependentTypes.lean zip1}}
```
その結果は、3つのペアのリストです：
```lean
{{#example_out Examples/DependentTypes.lean zip1}}
```
リストの長さが異なる場合に何が起こるべきかはいくぶん明らかではありません。
多くの言語と同様に、Leanはリストの片方に余分なエントリがある場合、それを無視することを選択します。
例えば、オレゴン州で最も高い5つのピークの高さとデンマークで最も高い3つのピークの高さを組み合わせると、3つのペアが得られます。
特に、
```lean
{{#example_in Examples/DependentTypes.lean zip2}}
```
は、次のように評価されます
```lean
{{#example_out Examples/DependentTypes.lean zip2}}
```

このアプローチは常に答えを返すので便利ですが、リストが意図せず異なる長さを持っている場合にデータを失うリスクを伴います。
F#は異なるアプローチを取ります：そのバージョンの `List.zip` は長さが一致しないときに例外を投げます。この `fsi` セッションで見ることができます：
```fsharp
> List.zip [3428.8; 3201.0; 3158.5; 3075.0; 3064.0] [170.86; 170.77; 170.35];;
```
```output error
System.ArgumentException: The lists had different lengths.
list2 is 2 elements shorter than list1 (Parameter 'list2')
   at Microsoft.FSharp.Core.DetailedExceptions.invalidArgDifferentListLength[?](String arg1, String arg2, Int32 diff) in /builddir/build/BUILD/dotnet-v3.1.424-SDK/src/fsharp.3ef6f0b514198c0bfa6c2c09fefe41a740b024d5/src/fsharp/FSharp.Core/local.fs:line 24
   at Microsoft.FSharp.Primitives.Basics.List.zipToFreshConsTail[a,b](FSharpList`1 cons, FSharpList`1 xs1, FSharpList`1 xs2) in /builddir/build/BUILD/dotnet-v3.1.424-SDK/src/fsharp.3ef6f0b514198c0bfa6c2c09fefe41a740b024d5/src/fsharp/FSharp.Core/local.fs:line 918
   at Microsoft.FSharp.Primitives.Basics.List.zip[T1,T2](FSharpList`1 xs1, FSharpList`1 xs2) in /builddir/build/BUILD/dotnet-v3.1.424-SDK/src/fsharp.3ef6f0b514198c0bfa6c2c09fefe41a740b024d5/src/fsharp/FSharp.Core/local.fs:line 929
   at Microsoft.FSharp.Collections.ListModule.Zip[T1,T2](FSharpList`1 list1, FSharpList`1 list2) in /builddir/build/BUILD/dotnet-v3.1.424-SDK/src/fsharp.3ef6f0b514198c0bfa6c2c09fefe41a740b024d5/src/fsharp/FSharp.Core/list.fs:line 466
   at <StartupCode$FSI_0006>.$FSI_0006.main@()
Stopped due to error
```
これは偶然情報を破棄することを避けますが、プログラムをクラッシュさせることにはそれ自体問題があります。
`Option` や `Except` モナドを使用した Lean の同等のものは、安全性の価値がないかもしれない負担を伴います。

しかし、`Vect` を使って、両方の引数が同じ長さであることを要求する型の `zip` バージョンを書くことができます：
```lean
{{#example_decl Examples/DependentTypes.lean VectZip}}
```
この定義には `Vect.nil` または `Vect.cons` のどちらかがある場合にのみパターンがあり、Leanは「ケース不足」エラーなしで定義を受け入れます。これは `List` に対する類似の定義から結果として生じるものです：
```lean
{{#example_in Examples/DependentTypes.lean zipMissing}}
```
```output error
{{#example_out Examples/DependentTypes.lean zipMissing}}
```
これは、最初のパターンで使用されるコンストラクタ、`nil` または `cons` が、長さ `n` について型チェッカーの知識を_洗練_するためです。
最初のパターンが `nil` の場合、型チェッカーはそれに加えて長さが `0` であったことをさらに決定できるので、2番目のパターンの唯一の可能な選択は `nil` です。
同様に、最初のパターンが `cons` の場合、型チェッカーは長さがいくつかの `Nat` `k` に対して `k+1` であったことを決定できるので、2番目のパターンの唯一の可能な選択は `cons` です。
実際に、`nil` と `cons` を一緒に使用するケースを追加すると型エラーになります、なぜなら長さが一致しないからです：
```lean
{{#example_in Examples/DependentTypes.lean zipExtraCons}}
```
```output error
{{#example_out Examples/DependentTypesれは型エラーです。また、`nil`と`cons`を一緒に使用するパターンを追加するのも型エラーになります。これは長さが一致しないからです。長さの洗練が観察されるのは、`n`を明示的な引数にすることでです：
```lean
{{#example_decl Examples/DependentTypes.lean VectZipLen}}
```

## 練習問題

依存型とともにプログラミングを行う感覚を掴むには経験が必要であり、このセクションの練習問題は非常に重要です。
各練習問題については、コードを進めながら型チェッカーがどのような間違いを捉え、どのような間違いを捉えられないのかを試しながら体験してみることで、エラーメッセージへの感覚を養う良い方法です。

 * `Vect.zip`がオレゴン州の最高峰3つとデンマークの最高峰3つを組み合わせる際に正しい答えを出すかダブルチェックしてください。`List`のようなシンタックスシュガーが`Vect`にはないため、`oregonianPeaks : Vect String 3`と`danishPeaks : Vect String 3`を定義することから始めると役立つかもしれません。

 * `(α → β) → Vect α n → Vect β n`の型を持つ関数`Vect.map`を定義してください。

 * 各`Vect`のエントリを函数で一つずつ組み合わせる関数`Vect.zipWith`を定義します。
   この関数には`(α → β → γ) → Vect α n → Vect β n → Vect γ n`という型があります。

 * ペアの`Vect`をペアの`Vect`に分割する関数`Vect.unzip`を定義してください。この関数には`Vect (α × β) n → Vect α n × Vect β n`という型があります。

 * `Vect`の_終わり_にエントリを追加する関数`Vect.snoc`を定義してください。その型は`Vect α n → α → Vect α (n + 1)`であり、`{{#example_in Examples/DependentTypes.lean snocSnowy}}`は`{{#example_out Examples/DependentTypes.lean snocSnowy}}`を生成するべきです。`snoc`という名前は、トラディショナルな関数型プログラミングの洒落であり、`cons`を逆にしたものです。

 * `Vect`の順番を逆にする関数`Vect.reverse`を定義してください。
 
 * 次の型を持つ関数`Vect.drop`を定義してください：`(n : Nat) → Vect α (k + n) → Vect α k`。
   `{{#example_in Examples/DependentTypes.lean ejerBavnehoej}}`が`{{#example_out Examples/DependentTypes.lean ejerBavnehoej}}`を生成することを確認することで動作を検証してください。

 * `Vect`の最初の`n`エントリを返す`(n : Nat) → Vect α (k + n) → Vect α n`型の関数`Vect.take`を定義してください。例で動作することを確認してください。