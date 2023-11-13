# 安全な配列インデックス

`Array`と`Nat`のための`GetElem`インスタンスは、提供された`Nat`が配列より小さいという証明が必要です。
実際には、これらの証明はしばしば関数にインデックスと共に渡される場合があります。
インデックスと証明を別々に渡すのではなく、`Fin`と呼ばれる型を使ってインデックスと証明をひとつの値に束ねることができます。
これによってコードが読みやすくなることがあります。
さらに、配列に対する多くの組み込み操作はインデックス引数を`Nat`ではなく`Fin`として取るので、これらの組み込み操作を使用するには`Fin`の使用方法を理解する必要があります。

型`Fin n`は`n`未満の数を表します。
言い換えると、`Fin 3`は`0`、`1`、`2`を意味し、`Fin 0`は全く値を持ちません。
`Fin`の定義は`Subtype`に似ており、`Fin n`は`Nat`とそれが`n`より小さいことの証明を含む構造です：
```lean
{{#example_decl Examples/ProgramsProofs/Fin.lean Fin}}
```

Leanには、`ToString`と`OfNat`のインスタンスが含まれており、`Fin`値を数値として便利に使用できるようになっています。
つまり、`{{#example_in Examples/ProgramsProofs/Fin.lean fiveFinEight}}`の出力は`{val := 5, isLt := _}`のようなものではなく、`{{#example_out Examples/ProgramsProofs/Fin.lean fiveFinEight}}`です。

与えられた数が上限より大きい場合、`OfNat`インスタンスの`Fin`は実行時エラーではなく、上限をモジュロとした値を返します。
これは、`{{#example_in Examples/ProgramsProofs/Fin.lean finOverflow}}`がコンパイル時エラーではなく、`{{#example_out Examples/ProgramsProofs/Fin.lean finOverflow}}`の結果になることを意味します。

返り値として、見つかったインデックスを`Fin`として返すことは、そのデータ構造にそれがどのように関連しているかをより明確にします。
[前節](./arrays-termination.md#proving-termination)の`Array.find`は、呼び出し元が直ちに配列を検索するために使用できないインデックスを返します。有効性についての情報が失われているからです。
より具体的な型を使うと、プログラムを著しく複雑にすることなく使用できる値が得られます：
```lean
{{#example_decl Examples/ProgramsProofs/Fin.lean ArrayFindHelper}}

{{#example_decl Examples/ProgramsProofs/Fin.lean ArrayFind}}
```

## 練習問題

`Fin.next? : Fin n → Option (Fin n)`という関数を書いてください。これは、次に大きい`Fin`が範囲内であればそれを返し、そうでなければ`none`を返します。
以下のように
```lean
{{#example_in Examples/ProgramsProofs/Fin.lean nextThreeFin}}
```
が出力するのが
```output info
{{#example_out Examples/ProgramsProofs/Fin.lean nextThreeFin}}
```
となり、また
```lean
{{#example_in Examples/ProgramsProofs/Fin.lean nextSevenFin}}
```
が出力するのが
```output info
{{#example_out Examples/ProgramsProofs/Fin.lean nextSevenFin}}
```
であることを確認します。