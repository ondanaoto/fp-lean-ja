# インデックス、パラメータ、そして宇宙レベル

帰納型のインデックスとパラメータの区別は、構成要素の間で変化するかしないかという型の引数を記述する方法以上のものです。
帰納型への引数がパラメータなのかインデックスなのかは、それらのユニバースレベルの関係を決定する時にも重要です。
特に、帰納型はパラメータと同じユニバースレベルを持つことができますが、インデックスよりも大きいユニバースにある必要があります。
この制約は、Leanが定理証明者であると同時にプログラミング言語として使われるために必要です—これがないと、Leanの論理は矛盾をおこします。
エラーメッセージを試すことは、これらのルールを示すだけでなく、型の引数がパラメータなのかインデックスなのかを決定する正確なルールを説明する良い方法です。

一般的に、帰納型の定義はそのパラメータをコロンの前に、そしてそのインデックスをコロンの後に取ります。
パラメータは関数引数のように名前を与えられますが、インデックスはその型のみが記述されます。
これは `Vect` の定義で見ることができます：
```lean
{{#example_decl Examples/DependentTypes.lean Vect}}
```
この定義では、`α` はパラメータであり、`Nat` はインデックスです。
パラメータは定義全体を通じて参照することができます（例えば、`Vect.cons` はその最初の引数の型として `α` を使用します）、しかし常に一貫して使用されなければなりません。
インデックスは変わることを期待されているため、データ型定義の先頭で引数として提供されるのではなく、各構造体で個別の値が割り当てられます。

パラメータを持つ非常にシンプルなデータ型は `WithParameter` です：
```lean
{{#example_decl Examples/DependentTypes/IndicesParameters.lean WithParameter}}
```
ユニバースレベル `u` はパラメータと帰納型自体のために使われますが、それがパラメータがデータ型のユニバースレベルを増加させないことを示しています。
同様に、複数のパラメータが存在する場合、帰納型は最大のユニバースレベルを受け取ります：
```lean
{{#example_decl Examples/DependentTypes/IndicesParameters.lean WithTwoParameters}}
```
パラメータがデータ型のユニバースレベルを増加させないので、取り扱うのがより便利です。
Leanはコロンの後（インデックスとして記述されていますが）に記述されたような引数がパラメータのように使用される場合にそれをパラメータに変換しようとします：
以下の両方の帰納データ型は、コロンの後にパラメータを記述しています：
```lean
{{#example_decl Examples/DependentTypes/IndicesParameters.lean WithParameterAfterColon}}

{{#example_decl Examples/DependentTypes/IndicesParameters.lean WithParameterAfterColon2}}
```

パラメータが最初のデータ型宣言で名前が付けられていない場合、それは各構造体で違う名前を使用することができますが、それらは一貫して使用されなければなりません。
次の宣言は受け入れられます：
```lean
{{#example_decl Examples/DependentTypes/IndicesParameters.lean WithParameterAfterColonDifferentNames}}
```
しかし、この柔軟性はパラメータの名前を明示的に宣言するデータ型には及ばないものです：
```lean
{{#example_in Examples/DependentTypes/IndicesParameters.lean WithParameterBeforeColonDifferentNames}}
```
```output error
{{#example_out Examples/DependentTypes/IndicesParameters.lean WithParameterBeforeColonDifferentNames}}
```
同様に、インデックスに名前を付けようとするとエラーが発生します：
```lean
{{#example_in Examples/DependentTypes/IndicesParameters.lean WithNamedIndex}}
```
```output error
{{#example_out Examples/DependentTypes/IndicesParameters.lean WithNamedIndex}}
```

適切なユニバースレベルを使用し、コロンの後にインデックスを配置することにより、受け入れられる宣言が結果します：
```lean
{{#example_decl Examples/DependentTypes/IndicesParameters.lean WithIndex}}
```

Leanは時々、帰納型宣言のコロンの後の引数が全ての構造体で一貫して使用され、どのインデックスの後ろにも来ない場合にパラメータであると判断できますが、全てのパラメータはインデックスの前になければなりません。
インデックスの後にパラメータを置くことを試みると、引数はインデックスと見なされることになり、その結果、データ型のユニバースレベルが増加する必要があります：
```lean
{{#example_in Examples/DependentTypes/IndicesParameters.lean ParamAfterIndex}}
```
```output error
{{#example_out Examples/DependentTypes/IndicesParameters.lean ParamAfterIndex}}
```

パラメータは型である必要はありません。
次の例は、`Nat` のような普通のデータ型がパラメータとして使用できることを示しています：
```lean
{{#example_in Examples/DependentTypes/IndicesParameters.lean NatParamFour}}
```
```output error
{{#example_out Examples/DependentTypes/IndicesParameters.lean NatParamFour}}
```
提案されている `n` を使用すると、宣言が受け入れられます：
```lean
{{#example_decl Examples/DependentTypes/IndicesParameters.lean NatParam}}
```


これらの実験から何が結論付けられるでしょうか？
パラメータとインデックスのルールは以下の通りです：
 1. 各構成要素の型でパラメータは同じように使われなければなりません。
 2. 全てのパラメータは全てのインデックスの前になければなりません。
 3. 定義されているデータ型のユニバースレベルは、最も大きいパラメータと同じかそれ以上でなければならず、最も大きいインデックスより厳密に大きくなければなりません。
 4. コロンの前に書かれた名前のある引数は常にパラメータであり、コロンの後の引数は通常インデックスです。Leanはコロンの後の引数の使用が一貫している場合、それをパラメータにすると判断するかもしれませんが、それがインデックスの後ろに来ない場合です。

疑わしい場合、Leanのコマンド `#print` を使用して帰納型引数のうちどれがパラメータであるかをチェックできます。
例えば、`Vect` の場合、パラメータの数が1であることを指摘しています：
```lean
{{#example_in Examples/DependentTypes/IndicesParameters.lean printVect}}
```
```output info
{{#example_out Examples/DependentTypes/IndicesParameters.lean printVect}}
```

データ型に引数の順番を決める際には、どの引数がパラメータでどの引数がインデックスであるべきかを考えることが重要です。
できるだけ多くの引数をパラメータにすることはユニバースレベルを制御下に保つのに役立ち、これにより複雑なプログラムがタイプチェックしやすくなることがあります。
これを可能にする一つの方法は、引数リストで全てのパラメータが全てのインデックスの前に来るようにすることです。

さらに、Leanがコロンの後の引数がパラメータであることを使用法によって決定できるにもかかわらず、パラメータを明確な名前で書くことは良い考えです。
これにより、読者に意図が明確になり、構造体間で引数が間違って不一致で使用されていた場合にLeanがエラーを報告することになります。