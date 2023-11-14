# 挿入ソートと配列の変更

挿入ソートは、ソーティングアルゴリズムにおいて最悪のケースでの時間複雑性が最適ではないものの、それでも多くの有用な特性があります。
 * 理解と実装が簡単で直感的
 * 追加のスペースを必要とせず、その場で処理するアルゴリズム
 * 安定したソートアルゴリズム
 * 入力がほぼソートされている場合に迅速
 
その場で処理するアルゴリズムは、Leanがメモリを管理する方法によって、特に有用です。
ある場合では、配列を複製することが通常必要とされる操作を変異に最適化できるかもしれません。
これには、配列での要素の交換が含まれます。

JavaScript、JVM、.NETを含む自動メモリ管理を持つほとんどの言語やランタイムシステムは、トレース型ガベージコレクションを使用しています。
メモリの回収が必要になると、システムはコールスタックやグローバル値のようなルートから始めて、再帰的にポインタを追うことでどの値に到達できるかを決定します。
到達できない値は取り除かれ、メモリが解放されます。

参照カウントは、Python、Swift、Leanを含む多くの言語で使用される、トレース型ガベージコレクションの代替方法です。
参照カウントを使用するシステムでは、メモリ上の各オブジェクトにそのオブジェクトへの参照数を追跡するフィールドがあります。
新しい参照が設定されると、カウンターは増加されます。
参照がなくなると、カウンターは減少します。
カウンターがゼロに達すると、オブジェクトは即座に解放されます。

参照カウントには、トレース型ガベージコレクターと比較して、循環参照によるメモリリークへと繋がる可能性があるという大きな欠点があります。
オブジェクト\\( A \\)がオブジェクト\\( B \\)を参照していて、オブジェクト\\( B \\)がオブジェクト\\( A \\)を参照している場合、プログラム内の他の何もオブジェクト\\( A \\)または\\( B \\)を参照していなくても、これらは決して解放されません。
循環参照は、制御されていない再帰または可変参照から生じます。
Leanはどちらもサポートしていないため、循環参照を構築することは不可能です。

参照カウントは、Leanランタイムシステムのプリミティブがデータ構造の割り当てと解放をチェックし、参照カウントがゼロになりそうな場合、新たにオブジェクトを割り当てる代わりに既存のオブジェクトを再利用することを意味します。
これは、特に大規模な配列を扱う際に特に重要です。

Leanの配列に対する挿入ソートの実装は、以下の基準を満たすべきです：
 1. `partial`アノテーションなしでLeanに関数を受け入れさせる
 2. 他に参照がない配列が渡された場合、新しい配列を割り当てるのではなく、その場で配列を変更するべきです
 
最初の基準は簡単にチェックできます：Leanが定義を受け入れるならば、それは満たされています。
しかし、2番目はテストする方法を必要とします。
Leanは`dbgTraceIfShared`と呼ばれる組み込み関数を提供しており、以下のシグネチャを持っています：
```lean
{{#example_in Examples/ProgramsProofs/InsertionSort.lean dbgTraceIfSharedSig}}
```
```output info
{{#example_out Examples/ProgramsProofs/InsertionSort.lean dbgTraceIfSharedSig}}
```
これは文字列と値を引数として取り、値が複数の参照を持っている場合に文字列を使用して標準エラーにメッセージを出力し、値を返します。
これは、厳密には純粋な関数ではありません。
ただし、関数が実際にメモリを再利用できることを開発中にチェックするためにのみ使用されることを意図しています。

`dbgTraceIfShared`の使い方を学ぶ際には、`#eval`がコンパイルされたコードよりも多くの値を共有していると報告することがあるため、混乱する可能性があります。
エディターで実験するのではなく、`lake`で実行可能ファイルをビルドすることが重要です。

挿入ソートは二つのループで構成されています。
外側のループはソートされる配列を左から右にポインタを移動させます。
各繰り返しの後、ポインタの左側の配列の領域はソート済みであり、右側の領域がまだソートされていない可能性があります。
内側のループは、ポインタによって指し示される要素を適切な位置が見つかるまで左に移動させ、ループ不変量が復元されます。
つまり、各反復は配列の次の要素をソート済みの領域に挿入します。

## 内側のループ

挿入ソートの内側のループは尾再帰関数として実装できます。これは、挿入される要素の配列とインデックスを引数とします。
挿入される要素は繰り返し左にある要素と交換され、もしそれが左にある要素より小さいか配列の始まりに達するまで交換が続きます。
内側のループは内部に持っている`Nat`について構造的再帰的です。これは配列にインデックスをつけるために使用される`Fin`にあります：
```leantac
{{#example_decl Examples/ProgramsProofs/InsertionSort.lean insertSorted}}
```
もしインデックス`i`が`0`ならば、ソートされた領域に挿入される要素は領域の始まりに達しており、最小です。
もしインデックスが`i' + 1`ならば、`i'`にある要素は`i`にある要素と比較されるべきです。
`i`が`Fin arr.size`である一方で、`i'`は`Nat`です。それは`i`の`val`フィールドから得られるからです。
それゆえ、`i'`を`arr`にインデックスとして使う前に`i' < arr.size`を証明する必要があります。

`i' < arr.size`を示す証明を含む`have`表現を省略すると、以下の目標が明らかになります：
```output error
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insertSortedNoProof}}
```

ヒント`Nat.lt_of_succ_lt`はLeanの標準ライブラリからの定理です。
そのシグネチャは`{{#example_in Examples/ProgramsProofs/InsertionSort.lean lt_of_succ_lt_type}}`で見つけられ、以下のようになります：
```output info
{{#example_out Examples/ProgramsProofs/InsertionSort.lean lt_of_succ_lt_type}}
```
言い換えれば、これはもし`n + 1 < m`ならば、`n < m`であることを示しています。
`simp`に渡された`*`は、`i`から`isLt`フィールドを使って最終的な証明を得るために`Nat.lt_of_succ_lt`を組み合わせます。

`i'`を使用して挿入される要素の左の要素が見つかり、二つの要素が比較された後、左にある要素が挿入される要素より小さいか等しいならば、ループは終了し不変量が復元されます。
もし左にある要素が挿入される要素より大きいならば、要素は交換され内側のループは再び始まります。
`Array.swap`はその両方のインデックスを`Fin`として取り、`i' < arr.size`を確立する`have`によって使用されます。
次の内側ループを通して検討されるインデックスも`i'`ですが、この場合`by assumption`だけでは十分ではありません。
これは証明が元の配列`arr`のために記述されたからであり、二つの要素を交換した結果ではありません。
`simp`戦術のデータベースには配列の二つの要素を交換するとそのサイズは変わらないという事実が含まれており、`[*]`引数はそれに加えて`have`によって導入された前提を利用するように指示します。

## 外側のループ

挿入ソートの外側のループは、ポインタを左から右に移動させ、各反復で`insertSorted`を呼び出して、ポインタにある要素を配列の正しい位置に挿入します。
ループの基本的な形は`Array.map`の実装に似ています：
```lean
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insertionSortLoopTermination}}
```
結果として得られるエラーも、`Array.map`に`termination_by`節がない場合に起こるエラーと同じです。これは再帰的な呼び出しの中で毎回減少する引数がないためです：
```output error
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insertionSortLoopTermination}}
```

終了証明を構築する前に、`partial`修飾子を使用して定義をテストし、期待通りの答えを返すことを確認すると便利です：
```lean
{{#example_decl Examples/ProgramsProofs/InsertionSort.lean partialInsertionSortLoop}}
```
```lean
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insertionSortPartialOne}}
```
```output info
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insertionSortPartialOne}}
```
```lean
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insertionSortPartialTwo}}
```
```output info
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insertionSortPartialTwo}}
```

### 終了

再度、関数は処理される配列のインデックスとサイズの差が各再帰呼び出しで減少するために終了することになります。
ただし、今回はLeanは`termination_by`を受け入れません：
```lean
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insertionSortLoopProof1}}
```
```output error
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insertionSortLoopProof1}}
```
問題はLeanが`insertSorted`が受け取ったものと同じサイズの配列を返すとは知りようがないことです。
`insertionSortLoop`が終了することを証明するためには、`insertSorted`が配列のサイズを変えないことをまず証明する必要があります。
エラーメッセージから未証明の終了条件を関数にコピーして`sorry`で「証明」し、関数を一時的に受け入れることを許可します：

```leantac
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insertionSortLoopSorry}}
```
```output warning
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insertionSortLoopSorry}}
```

`insertSorted`は挿入される要素のインデックスに構造的再帰をしているため、基本ケースの証明は帰納法によって行われるべきです。
基本ケースでは、配列は変更されずに返されるため、その長さは変わらないに違いありません。
帰納法のステップのためには、次に小さいインデックスでの再帰呼び出しが配列の長さを変えないことが帰納法の仮定です。
考慮すべきは二つのケースです：要素がソートされた領域に完全に挿入され、配列は変更されずに返された場合はその長さも変わらず、あるいは要素は次のものと交換される前に再帰呼び出しがあります。
しかしながら、配列の二つ要素を交換することはサイズを変えませんし、帰納法の仮定は次のインデックスでの再帰呼び出しは引数としての同じサイズの配列を返すと述べています。
したがって、サイズは変わりません。

英語の定理のステートメントをLeanに翻訳し、この章のテクニックを使って進むことで基本ケースの証明を確立し帰納法ステップで進展を遂げることが十分です：
```leantac
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insert_sorted_size_eq_0}}
```
帰納法ステップでの`insertSorted`を使用した単純化は`insertSorted`のパターンマッチを明らかにしました：
```output error
{{#example_out ExamplesProgramsProofs/InsertionSort.lean insert_sorted_size_eq_0}}
```
あなたが`if`や`match`を含むゴールに直面したとき、`split`タクティックはコントロールフローの各パスについて1つの新しいゴールを置き換えます。
```leantac
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insert_sorted_size_eq_1}}
```
さらに、各新しいゴールはそのゴールに到達したブランチを示す仮定を持っています。この場合の名前は`heq✝`です。
```output error
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insert_sorted_size_eq_1}}
```
両方の単純なケースに対する証明を書くのが面倒な場合に`split`の後に`<;> try rfl`を追加すると、2つの単純なケースがすぐになくなり、唯一の1つのゴールだけが残ります：
```leantac
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insert_sorted_size_eq_2}}
```
```output error
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insert_sorted_size_eq_2}}
```

残念ながら、帰納仮説はこのゴールを証明するのに十分ではありません。帰納仮説は`insertSorted`を`arr`に対して呼び出すとサイズが変化しないと述べているだけで、証明目標はスワップの結果に対する再帰的呼び出しがサイズを変化させないことを示すことです。証明を成功させるためには、減少したインデックスを引数として使用する_どの_配列に対しても有効な帰納仮説が必要です。

`induction`タクティックの`generalizing`オプションを使用することで、より強い帰納仮説を得ることができます。このオプションは追加の仮定をコンテキストからベースケース、帰納仮説、および推論ステップの目標を生成するためのステートメントに結合します。`arr`について一般化することで、より強い仮説をもたらします：
```leantac
{{#example_in Examples/ProgramsProofs/InsertionSort.lean insert_sorted_size_eq_3}}
```
この結果として生じるゴールでは、`arr`は帰納仮説の「すべてのための」ステートメントの一部になっています：
```output error
{{#example_out Examples/ProgramsProofs/InsertionSort.lean insert_sorted_size_eq_3}}
```

しかしながら、この証明全体は扱いにくいものになりつつあります。次のステップは、スワップの結果の配列の長さを表す変数を導入し、それが`arr.size`と等しいことを示し、次にその変数が再帰呼び出しの結果から生じる配列の長さとも等しいことを示すことです。これらの等式ステートメントは連鎖させられ、最終的なゴールを証明するために使用することができます。しかしながら、帰納仮説が自動的に十分に強く、変数がすでに導入されているような定理ステートメントを注意深く再構成する方がはるかに容易です。再構成されたステートメントは次のように読めます：