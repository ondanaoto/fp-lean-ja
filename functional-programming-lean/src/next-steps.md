# 次のステップ

この本は、Leanにおける関数型プログラミングの非常に基本的な内容、そしてほんの少しの対話型定理証明について紹介しています。
依存型関数言語であるLeanを使うことは深いトピックであり、多くのことが言えます。
あなたの興味に依存して、Lean 4を学ぶための以下のリソースが役立つかもしれません。

## Leanの学習

Lean 4自体については以下のリソースで説明されています：

 * [Lean 4での定理証明](https://leanprover.github.io/theorem_proving_in_lean4/) はLeanを使った証明を書くためのチュートリアルです。
 * [Lean 4マニュアル](https://leanprover.github.io/lean4/doc/) は言語とその機能のリファレンスを提供しています。執筆時点ではまだ不完全ですが、この本よりも詳細に多くのLeanの側面について説明しています。
 * [Leanでそれを証明する方法](https://djvelleman.github.io/HTPIwL/) は、ペーパーペンによる数学的証明の書き方を紹介する著名な教科書 [_How To Prove It_](https://www.cambridge.org/highereducation/books/how-to-prove-it/6D2965D625C6836CD4A785A2C843B3DA#overview) のLeanベースの対応物です。
 * [Lean 4のメタプログラミング](https://github.com/arthurpaulino/lean4-metaprogramming-book) は、インフィックスオペレーターや記法からマクロ、カスタムタクティクス、そして完全なカスタム組み込み言語に至るまで、Leanの拡張メカニズムの概観を提供します。
 * [Leanでの関数型プログラミング](https://leanprover.github.io/functional_programming_in_lean/) は、再帰に関するジョークを楽しむ読者にとって興味深いものかもしれません。

しかしながら、Leanの学習を続ける最良の方法は、コードを読んで書き始めること、そして立ち往生した時にはドキュメントを参照することです。
さらに、[Lean Zulip](https://leanprover.zulipchat.com/) は他のLeanユーザーと出会い、助けを求め、他の人を助けるための優れた場所です。

## 標準ライブラリ

Lean自体は、箱から出してすぐには非常に最小限のライブラリしか含んでいません。
Leanは自己ホスト型で、含まれているコードはLean自体を実装するのにちょうど十分です。
多くのアプリケーションでは、より大きな標準ライブラリが必要です。

[std4](https://github.com/leanprover/std4) は進行中の標準ライブラリで、Leanのコンパイラ自体の範囲外である多くのデータ構造、タクティクス、型クラスのインスタンス、関数を含んでいます。
`std4`を使うためには、最初にあなたが使用しているLean 4のバージョンと互換性のあるコミットをその履歴から見つけることがステップの一つです（つまり、`lean-toolchain`ファイルがプロジェクトのものと一致しているもの）。
そして、適切なバージョンの`COMMIT_HASH`を以下のようにあなたの`lakefile.lean`のトップレベルに追加します：
```lean
require std from git
  "https://github.com/leanprover/std4/" @ "COMMIT_HASH"
```


## Leanにおける数学

数学者向けのほとんどのリソースはLean 3用に書かれています。
広範囲にわたる選択肢は[コミュニティサイト](https://leanprover-community.github.io/learn.html)で利用可能です。
Lean 4で数学を始めるには、数学ライブラリ`mathlib`をLean 3からLean 4に移植するプロセスに参加することがおそらく最も簡単です。
さらなる情報については、[`mathlib4` README](https://github.com/leanprover-community/mathlib4)をご覧ください。

## コンピューターサイエンスでの依存型の使用

CoqはLeanと多くの共通点がある言語です。
コンピューターサイエンティストにとって、対話型の教科書である[Software Foundations](https://softwarefoundations.cis.upenn.edu/) シリーズは、コンピューターサイエンスにおけるCoqの適用に関して素晴らしい導入を提供します。
LeanとCoqの基礎的なアイデアは非常に似ており、スキルはシステム間で容易に転用することができます。

## 依存型を使ったプログラミング

インデックス付きファミリーや依存型を使用してプログラムを構築することに興味のあるプログラマーにとって、Edwin Bradyの[_Type Driven Development with Idris_](https://www.manning.com/books/type-driven-development-with-idris) は優れた導入書を提供しています。
Coqのように、IdrisもLeanの近い親戚ですが、タクティクスを欠いています。

## 依存型の理解

[_The Little Typer_](https://thelittletyper.com/) は、公式には論理学やプログラミング言語理論を研究していないが、依存型理論の核となるアイデアを構築することに関心のあるプログラマー向けの本です。
上記のリソースはできるだけ実践的であろうとしていますが、_The Little Typer_はプログラミングからの概念を使って非常に基本的から構築される依存型理論へのアプローチを提示します。
免責事項：_Leanでの関数型プログラミング_ の著者は、_The Little Typer_ の著者でもあります。