```
# プログラミング、証明、そしてパフォーマンス

この章ではプログラミングに焦点を当てます。
プログラムは正しい結果を計算する必要がありますが、それを効率的に実行することも大切です。
効率的な関数型プログラムを書くためには、データ構造を適切に利用する方法と、プログラムの実行に必要な時間とメモリを考慮することが重要です。

また、この章では証明についても触れています。
Leanで効率的なプログラミングにおいて、最も重要なデータ構造の一つは配列です。配列を安全に使うためには、配列のインディックスが有効範囲内にあることを証明する必要があります。
さらに、多くの興味深い配列アルゴリズムは、構造的再帰のパターンに従わず、代わりに配列を繰り返し処理します。
これらのアルゴリズムは終了するものですが、Leanが自動で終了を保証するわけではありません。
証明はプログラムがなぜ終了するのかを示すために役立ちます。

プログラムを書き換えて速度を上げると、しばしば理解が難しいコードになってしまいます。
証明はまた、まったく異なるアルゴリズムや実装技術が使われた場合でも、二つのプログラムが同じ結果を計算することを示すことができます。
このようにして、遅いけれども理解しやすいプログラムは、速く複雑なバージョンの仕様書として機能することが可能です。

証明とプログラミングを組み合わせることで、プログラムはより安全かつ効率的になります。
証明によって、ランタイム時の境界チェックを省略できるようになり、さまざまなテストが不要になることもあり、ランタイムパフォーマンスのコストをかけずに非常に高い信頼性をプログラムに提供できます。
しかし、プログラムに関する定理を証明する作業は時間がかかることもあり、コストがかかる場合もあるため、他のツールの方が経済的な選択となることがよくあります。

対話型定理証明は複雑なトピックです。
この章は、Leanを用いてプログラミングを行う中で実際に遭遇する証明の一端を垣間見るだけです。
多くの興味深い定理はプログラミングとは直接関係がありません。
さらに学びたい方のために、リソースのリストは「[次のステップ](next-steps.md)」をご覧ください。
しかし、プログラミングを学ぶ際同様に、実際に証明を書く経験が学びにとって代わることはありません。始める時が来たのです！
```