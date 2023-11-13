# インスタンス検索の制御

`Add` クラスのインスタンスは、型 `Pos` を持つ二つの式を便利に追加することを可能にし、別の `Pos` を生産します。
しかし、多くの場合、より柔軟になり、引数のタイプが異なることを許容する_異種_オペレーターオーバーロードを可能にした方が役に立ちます。
たとえば、`Nat` に `Pos` を加えたり、`Pos` に `Nat` を加えたりすると、常に `Pos` が得られます：
```lean
{{#example_decl Examples/Classes.lean addNatPos}}
```
これらの関数は自然数を正の数に追加することができますが、両方の引数が同じタイプであることを期待している `Add` タイプクラスでは使用できません。

## 異種オーバーロード

[オーバーロードされた加算](pos.md#overloaded-addition)に関するセクションで言及されているように、Leanは異種加算をオーバーロードするために `HAdd` と呼ばれるタイプクラスを提供しています。
`HAdd` クラスは3つのタイプパラメータを取ります：2つの引数タイプと返却タイプ。
`HAdd Nat Pos Pos` および `HAdd Pos Nat Pos` のインスタンスにより、タイプを混在させるために通常の加算記法を使用することができます：
```lean
{{#example_decl Examples/Classes.lean haddInsts}}
```
上記の2つのインスタンスを使用すると、以下の例が機能します：
```lean
{{#example_in Examples/Classes.lean posNatEx}}
```
```output info
{{#example_out Examples/Classes.lean posNatEx}}
```
```lean
{{#example_in Examples/Classes.lean natPosEx}}
```
```output info
{{#example_out Examples/Classes.lean natPosEx}}
```

`HAdd` タイプクラスの定義は、以下の `HPlus` の定義とその対応するインスタンスと非常によく似ています：
```lean
{{#example_decl Examples/Classes.lean HPlus}}

{{#example_decl Examples/Classes.lean HPlusInstances}}
```
しかし、`HPlus` のインスタンスは `HAdd` のインスタンスよりはるかに役に立たないです。
`#eval` を使用してこれらのインスタンスを使用しようとすると、エラーが発生します：
```lean
{{#example_in Examples/Classes.lean hPlusOops}}
```
```output error
{{#example_out Examples/Classes.lean hPlusOops}}
```
これはタイプにメタ変数があり、Leanがそれを解決する方法がないために発生します。

[ポリモーフィズムの最初の説明](../getting-to-know/polymorphism.md)で議論されたように、メタ変数はプログラムの推測できなかった未知の部分を表します。
`#eval` の後に式を記述すると、Leanはそのタイプを自動的に決定しようとします。
この場合、これができませんでした。
`HPlus` の三番目のタイプパラメータが未知だったので、Leanはタイプクラスのインスタンス検索を実行できませんでしたが、インスタンス検索は、Leanが式のタイプを決定する唯一の方法です。
つまり、`HPlus Pos Nat Pos` のインスタンスは、式がタイプ `Pos` を持つべき場合にのみ適用できますが、インスタンス自体以外にはそれがこのタイプを持つべきであることを示すものはプログラムにはありません。

この問題の一つの解決策は、式全体にタイプ注釈を追加して、全ての3つのタイプを利用可能にすることです：
```lean
{{#example_in Examples/Classes.lean hPlusLotsaTypes}}
```
```output info
{{#example_out Examples/Classes.lean hPlusLotsaTypes}}
```
しかし、この解決策は正の数ライブラリのユーザーにとって非常に便利ではありません。