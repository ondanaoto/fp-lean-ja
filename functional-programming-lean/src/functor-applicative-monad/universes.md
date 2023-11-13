# 宇宙 (Universes)

簡単のため、この本はここまでLeanの重要な特徴である_宇宙_ (universes) を省略してきました。
宇宙とは、他の型を分類する型です。
`Type` と `Prop` の2つは馴染み深いでしょう。
`Type` は `Nat`、`String`、`Int → String × Char`、`IO Unit` のような通常の型を分類します。
`Prop` は `"nisse" = "elf"` や `3 > 2` のような真か偽かの命題を分類します。
`Prop` の型は `Type` です:
```lean
{{#example_in Examples/Universes.lean PropType}}
```
```output info
{{#example_out Examples/Universes.lean PropType}}
```

技術的な理由から、これら2つよりも多くの宇宙が必要です。
特に、`Type`自体は`Type`にはなり得ません。
これを許すと論理的パラドックスが構築され、Leanの定理証明者としての有用性が損なわれることになります。

このための公式な議論は _ジラールのパラドックス_ (Girard's Paradox) として知られています。
これはセット理論の初期バージョンが矛盾していたことを示すために使用された、よりよく知られているパラドックスである _ラッセルのパラドックス_ (Russell's Paradox) に関連しています。
これらのセット理論では、セットはプロパティによって定義することができます。
たとえば、全ての赤いもののセット、全ての果物のセット、全ての自然数のセット、または全てのセットのセットなどです。
セットが与えられたら、特定の要素がそれに含まれているかどうかを尋ねることができます。
例えば、青い鳥は全ての赤いもののセットには含まれませんが、全ての赤いもののセットは全てのセットのセットに含まれます。
事実、全てのセットのセットはそれ自体をも含んでいます。

では、自己を含まない全てのセットのセットはどうでしょうか？
それは全ての赤いもののセットを含んでいます。なぜならそのセットは赤くはありません。
しかし、全てのセットのセットは含んでいません。なぜならそれ自体を含んでいるからです。
しかし、そのセットは自分自身を含んでいるのでしょうか？
もし含んでいるならば、それ自体を含むことはできません。
しかし含んでいないのならば、それを含むべきです。

これは矛盾であり、初期の前提に何か間違いがあったことを示しています。
特に、任意のプロパティを提供してセットを構築することを許すのは強力さが過ぎます。
後のバージョンのセット理論はパラドックスを取り除くためにセットの形成を制限しています。

依存型理論のバージョンで`Type`に`Type`という型を割り当てると同様のパラドックスが構築されます。
Leanが一貫した論理的基盤を持ち、数学のためのツールとして利用できるようにするためには、`Type`には別の型が必要です。
この型は`Type 1`と呼ばれます:
```lean
{{#example_in Examples/Universes.lean TypeType}}
```
```output info
{{#example_out Examples/Universes.lean TypeType}}
```
同様に、`{{#example_in Examples/Universes.lean Type1Type}}`は`{{#example_out Examples/Universes.lean Type1Type}}`であり、
`{{#example_in Examples/Universes.lean Type2Type}}`は`{{#example_out Examples/Universes.lean Type2Type}}`であり、
`{{#example_in Examples/Universes.lean Type3Type}}`は`{{#example_out Examples/Universes.lean Type3Type}}`です、など。

関数型は両方の引数型と戻り値型を含むことのできる最小の宇宙に配置されます。
これは`{{#example_in Examples/Universes.lean NatNatType}}`が`{{#example_out Examples/Universes.lean NatNatType}}`であり、`{{#example_in Examples/Universes.lean Fun00Type}}`が`{{#example_out Examples/Universes.lean Fun00Type}}`であり、`{{#example_in Examples/Universes.lean Fun12Type}}`が`{{#example_out Examples/Universes.lean Fun12Type}}`であることを意味します。

このルールには一つの例外があります。
関数の戻り値型が`Prop`の場合、その関数型全体は`Prop`になります。たとえ引数がより大きな宇宙である`Type`や`Type 1`にあったとしてもです。
特に、これは通常の値に対する述語が`Prop`にあることを意味します。
例えば、型`{{#example_in Examples/Universes.lean FunPropType}}`は`Nat`からそれが自分自身プラスゼロと等しい証拠への関数を表しています。
`Nat`は`Type`にありますが、この関数型はこのルールにより`{{#example_out Examples/Universes.lean FunPropType}}`にあります。
同様に、`Type`は`Type 1`にありますが、関数型`{{#example_in Examples/Universes.lean FunTypePropType}}`は`{{#example_out Examples/Universes.lean FunTypePropType}}`にあります。

## ユーザー定義型

構造や帰納的データ型は特定の宇宙に属するように宣言することができます。
Leanはそれぞれのデータ型が十分に大きな宇宙にあり、自身の型を含むことを防ぐことによってパラドックスを避けるかをチェックします。
例えば、次の宣言では`MyList`が`Type`に居住すると宣言されており、その型引数`α`もそうです:
```lean
{{#example_decl Examples/Universes.lean MyList1}}
```
`{{#example_in Examples/Universes.lean MyList1Type}}`自体は`{{#example_out Examples/Universes.lean MyList1Type}}`です。
これは実際の型を含むことに使われることができないことを意味します、なぜならその引数が`Type`になり、それは`Type 1`だからです:
```lean
{{#example_in Examples/Universes.lean myListNat1Err}}
```
```output error
{{#example_out Examples/Universes.lean myListNat1Err}}
```

`MyList`を更新して、その引数が`Type 1`である結果はLeanによって拒絶される定義です:
```lean
{{#example_in Examples/Universes.lean MyList2}}
```
```output error
{{#example_out Examples/Universes.lean MyList2}}
```
このエラーは`cons`の引数となる型`α`が`MyList`より大きな宇宙のものだから発生します。
`MyList`自身を`Type 1`に配置することでこの問題が解決しますが、`Type`が期待されるコンテキストでの`MyList`の利用が不便になるという代償があります。

データ型が許可されるかどうかを決める具体的なルールはやや複雑です。
一般的には、最大の引数と同じ宇宙からそのデータ型を始めるのが最も簡単です。
そして、もしその定義がLeanに拒絶されたら、そのレベルを1つ上に上げることが普通は通ります。

## 宇宙多相性 (Universe Polymorphism)

特定の宇宙内でデータ型を定義すると、コードが重複する場合があります。
`MyList`を`Type → Type`内で配置することは、実際の型のリストに使用できないことを意味します。
それを`Type 1 → Type 1`に配置すると、型のリストのリストには使用できません。
`Type`、`Type 1`、`Type 2`などにバージョンをコピー＆ペーストする代わりに、任意のこれらの宇宙でインスタンス化できる単一の定義を書くために宇宙多相性という機能を使用することができます。

通常の多相型は定義で型を表す変数を使用します。
これによりLeanは変数を異なる方法で埋めることができ、これらの定義がいろいろな型で使用されることを可能にします。
同様に、宇宙多相性によって、変数が定義の宇宙を表すことを可能にし、それらが多様な宇宙で使用されることを可能にします。
型引数が一般的にギリシャ文字で命名されるのと同じように、宇宙引数は慣習的に`u`、`v`、`w`で命名されます。

この`MyList`の定義は特定の宇宙レベルを指定せず、代わりに任意のレベルを表す変数`u`を使用しています。
生成されたデータ型が`Type`で使われる場合`u`は`0`であり、`Type 3`で使用される場合`u`は`3`です:
```lean
{{#example_decl Examples/Universes.lean MyList3}}
```

この定義により、自然数自体と自然数型自体の両方を含む`MyList`の同じ定義を使用することができます:
```lean
{{#example_decl Examples/Universes.lean myListOfNat3}}
```
それ自身を含むことさえできます:
```lean
{{#example_decl Examples/Universes.lean myListOfList3}}
```

これにより論理的パラドックスを書くことが可能になるかのように思われるかもしれません。
結局のところ、宇宙システムの全体的な目的は自己参照型を排除することです。
しかし裏では、`MyList`の各出現に宇宙レベル引数が提供されます。
本質的に、宇宙多相性の`MyList`の定義は各レベルでのデータ型の_コピー_を作り、レベル引数は使用するコピーを選ぶためにあります。
これらのレベル引数はドットと中括弧で書かれるので、`{{#example_in Examples/Universes.lean MyListDotZero}} : {{#example_out Examples/Universes.lean MyListDotZero}}`、`{{#example_in Examples/Universes.lean MyListDotOne}} : {{#example_out Examples/Universes.lean MyListDotOne}}`、`{{#example_in Examples/Universes.lean MyListDotTwo}} : {{#example_out Examples/Universes.lean MyListDotTwo}}`です。

レベルを明示的に書くと、前の例は以下のようになります:
```lean
{{#example_decl Examples/Universes.lean myListOfList3Expl}}
```

宇宙多相性の定義が複数の型を引数として取る場合、それぞれの引数にそれぞれのレベル変数を与えることが最大の柔軟性のために良い考えです。
たとえば、単一のレベル引数を持つ`Sum`のバージョンは次のように書かれます:
```lean
{{#example_decl Examples/Universes.lean SumNoMax}}
```
この定義は複数のレベルで使用できます:
```lean
{{#example_decl Examples/Universes.lean SumPoly}}
```
ただし、両方の引数が同じ宇宙にある必要があります:
```lean
{{#example_in Examples/Universes.lean stringOrTypeLevels}}
```
```output error
{{#example_out Examples/Universes.lean stringOrTypeLevels}}
```

このデータ型は2つの型引数の宇宙レベルに異なる変数を使用し、その結果のデータ型が2つの中で最大の宇宙にあることを宣言することでより柔軟にできます:
```lean
{{#example_decl Examples/Universes.lean SumMax}}
```
これにより`Sum`は異なる宇宙からの引数で使用できます:
```lean
{{#example_decl Examples/Universes.lean stringOrTypeSum}}
```
    
Leanが宇宙レベルを期待する位置では、次のどれかが使用できます:
 * 具体的なレベル、例えば`0`や`1`
 * レベルを表す変数、例えば`u`や`v`
 * 2つのレベルの最大値、`max`がレベルに適用される形
 * レベルの増加、`+ 1`と書かれる形

### 宇宙多相性定義の書き方

これまでのこの本で定義されているデータ型は、データの最小宇宙である`Type`にあります。
Lean標準ライブラリから多相型、例えば`List`や`Sum`を紹介する際、この本はそれらの非宇宙多相性バージョンを作成しました。
実際のバージョンは宇宙多相性を使用して型レベルおよび非型レベルプログラム間でのコードの再利用を可能にします。

宇宙多相性型を書く際にはいくつか一般的な指針に従うと良いでしょう。
まず、独立した型引数は異なる宇宙変数を持つべきで、これにより多相性の定義がより多様な引数で使用されることを可能にし、コード再利用の可能性を高めます。
次に、全体の型自体は通常、全ての宇宙変数の最大値か、この最大値よりも1つ大きい値になります。最初に小さい方を試してみてください。
最後に、できるだけ小さい宇宙で新しい型を定義することで、他の文脈での使用が柔軟になります。
非多相型、例えば`Nat`や`String`などは直接`Type 0`に配置します。

### `Prop`と多相性

`Type`、`Type 1`などがプログラムやデータを分類する型を記述するのと同じように、`Prop`は論理的な命題を分類します。
`Prop`の型は、ステートメントの真実性に対する説得力のある証拠が何であるかを記述します。
命題は多くの面で通常の型と似ています：帰納的に宣言でき、コンストラクタがあり、関数が命題を引数として取ることができます。
しかし、データ型と異なり、ステートメントの真実性に対する証拠が提供されることは通常重要であるものの、証拠が_どのようなものであるか_は通常は重要ではありません。
一方で、プログラムが`Nat`を返す場合には、それが_正しい_ `Nat`であることは非常に重要です。

`Prop`は宇宙階層の最下位にあり、`Prop`の型は`Type`です。
これは`Prop`が`Nat`と同じ理由で`List`に提供するための適切な引数であることを意味します。
命題のリストは `List Prop`型を持ちます:
```lean
{{#example_decl Examples/Universes.lean someTrueProps}}
```
宇宙の引数を明示的に記入することにより、`Prop`が`Type`であることが示されます:
```lean
{{#example_decl Examples/Universes.lean someTruePropsExp}}
```

内部的には、`Prop`と`Type`は`Sort`と呼ばれる単一の階層に統合されています。
`Prop`は`Sort 0`と同じで、`Type 0`は`Sort 1`、`Type 1`は`Sort 2`であり、その先も続きます。
実際には、`Type u`は`Sort (u+1)`と同じです。
Leanでプログラムを書くときにはこれは通常関係ありませんが、時々エラーメッセージに現れることがあり、また`CoeSort`クラスの名前の由来を説明するものでもあります。
さらに、`Prop`を`Sort 0`として持つことで、`Prop`と`Type`の宇宙の間でできる限り移植性のあるコードを書くときに`Prop`を返す関数に特別なルールを使用するようになります。

## 実践における多相性

この本の残りの部分では、Lean標準ライブラリと一致するために、多相的なデータ型、構造体、およびクラスの定義に宇宙多相性が使用されます。
これにより`Functor`、`Applicative`、`Monad`クラスの完全な提示が実際の定義と完全に一致することが可能になります。