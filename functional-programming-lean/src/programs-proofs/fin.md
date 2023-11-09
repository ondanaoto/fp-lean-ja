# 安全な配列インデックス

`Array` および `Nat` に対する `GetElem` インスタンスは、提供された `Nat` が配列より小さいことの証明を必要とします。
実際には、これらの証明はしばしばインデックスと一緒に関数に渡されます。
インデックスと証明を別々に渡すのではなく、`Fin` と呼ばれる型を使ってインデックスと証明を単一の値にまとめることができます。
これにより、コードが読みやすくなります。
さらに、配列のビルトイン操作の多くは、インデックス引数を `Nat` ではなく `Fin` として取るので、これらのビルトイン操作を使用するには `Fin` の使用方法を理解する必要があります。

`Fin n` 型は `n` より厳格に小さい数を表します。
言い換えると、`Fin 3` は `0`、`1`、`2` を表し、`Fin 0` には値が全くありません。
`Fin` の定義は `Subtype` に似ており、`Fin n` は `Nat` とそれが `n` より小さいことの証明を含む構造です：
```lean
{{#example_decl Examples/ProgramsProofs/Fin.lean Fin}}
```

Lean には `ToString` と `OfNat` のインスタンスが含まれており、`Fin` の値を数値として便利に使用できます。
つまり、`{{#example_in Examples/ProgramsProofs/Fin.lean fiveFinEight}}` の出力は `{val := 5, isLt := _}` といったものではなく、`{{#example_out Examples/ProgramsProofs/Fin.lean fiveFinEight}}` となります。

`OfNat` の `Fin` に対するインスタンスでは、提供された数が境界より大きい場合は失敗するのではなく、境界に対するモジュロの値を返します。
これは、`{{#example_in Examples/ProgramsProofs/Fin.lean finOverflow}}` がコンパイル時エラーではなく、`{{#example_out Examples/ProgramsProofs/Fin.lean finOverflow}}` を結果とすることを意味します。

戻り値の型では、発見されたインデックスとして返される `Fin` は、そのインデックスが発見されたデータ構造との関連をより明確にします。
[前のセクション](./arrays-termination.md#proving-termination)での `Array.find` は、その妥当性に関する情報が失われてしまうため、呼び出し側が直ちに配列へのルックアップに使用できないインデックスを返します。
より具体的な型を使用することで、プログラムを著しく複雑にすることなく、使用できる値が得られます：
```lean
{{#example_decl Examples/ProgramsProofs/Fin.lean ArrayFindHelper}}

{{#example_decl Examples/ProgramsProofs/Fin.lean ArrayFind}}
```

## 練習問題

境界内であれば次に大きい `Fin` を、そうでなければ `none` を返す関数 `Fin.next? : Fin n → Option (Fin n)` を書いてみてください。
以下のコードが
```lean
{{#example_in Examples/ProgramsProofs/Fin.lean nextThreeFin}}
```
以下の出力をすることを確認してください。
```output info
{{#example_out Examples/ProgramsProofs/Fin.lean nextThreeFin}}
```
そして、以下のコードが
```lean
{{#example_in Examples/ProgramsProofs/Fin.lean nextSevenFin}}
```
以下の出力をすることを確認してください。
```output info
{{#example_out Examples/ProgramsProofs/Fin.lean nextSevenFin}}
```
