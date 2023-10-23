---
title: Functional Graphのサイクル検出をいい感じに
# description: 

date: 2023-10-13
# lastmod: yyyy-mm-dd
# hidedate: true

# ogimage: https://hoge/fuga/piyo.img

tags:
  - 競技プログラミング
  - グラフ
  - 典型テク
archives:
  - 2023
  - 2023-10
# sample
# - yyyy
# - yyyy-mm

# math: true
# toc: false
# build: {list: never}
---

## はじめに
グラフ表現に帰着できる問題を考察していると、Functional Graphと呼ばれるグラフのサイクル検出に帰着する場合があります。
本稿では、Functional Graphのサイクル検出の比較的簡単な実装を紹介します。

## 前提条件
本稿における、Functional Graphの定義は次のとおりです。
- 各頂点の出次数がちょうど1である。

## 方法
まず、画像でイメージを掴んでもらいます。

![](https://res.cloudinary.com/dqoqdn2sk/image/upload/v1697171658/pictures/functional-graph/graph_wq6ymb.png)

これがFunctional Graphです。各頂点から一つだけ有向辺があるのが確認できると思います。

ここで、頂点{1, 2, 3, 4, 5}は閉路をなしていますが、それ以外は閉路に含まれていません。
これをどうやって検出するかが本題です。

UnionFind(dsu)を使います。
まず、Functional Graphの重要な性質に、連結成分がちょうど一つの閉路を持つというものがあります。

<details>
<sammary>証明(厳密でない)</sammary>
1. 連結成分が少なくとも一つの閉路を持つこと

頂点1つから考えて、まだ連結成分でない頂点へ有向辺を張ると、必ず連結成分のサイズが1増えます。
つまり、サイズkの連結成分を成す最小の有向辺はk-1本です。

一方、Functional Graphにおけるサイズkの連結成分は必ずk本の有向辺を持ちます。

これは、少なくとも1本はすでに連結である頂点へと有向辺を張っていることになり、閉路を含みます。

2. 高々1つの閉路を持つこと

有向グラフがサイズkの閉路を成すとき、有向辺は最小でk本必要です。
2つのサイクルが連結であるとき、
サイクル間をつなぐ辺が少なくとももう一本必要であるから、
サイズk, lの閉路を含む連結成分は最小でもk+l+1本の有向辺が必要です。
しかし、このような連結成分はFunctional Graphの仮定から存在できません。(辺が多すぎる)

証明終わり。
</details>

UnionFindで連結成分を増やしていくと、どこかですでに同じ連結成分に属する頂点が見つかるはずです。
このとき見つかる頂点は、必ず閉路の中に入っている頂点になります。

Functional Graphにおける連結成分はちょうど一つだけ閉路を持ち、
出次数が必ず1であるため、閉路を構成する1頂点が見つかれば、
閉路に入っている他の全ての頂点もたどっていくだけで全て見つけることができます。

よって、以下のアルゴリズムを得ます。

以下、G[i]は頂点iに隣接する頂点を指す。
i=0,1,...,Nに対して、次を行う。
1. 頂点iと頂点G[i]が同じ連結成分に属していなければ、頂点iと頂点G[i]をマージして終了。同じ連結成分に属していれば、2に進む。
2. pos=iとして、再びpos=iとなるまでpos=G[pos]として更新する。
その途中でposに代入された頂点は、全て記録しておく。
記録された頂点は、一つの連結成分の閉路をなす頂点集合と一致する。

時間計算量はO(Nα(N))です。(α(N))はアッカーマン逆関数)

また、本稿では詳しく紹介しませんが、強連結成分分解によって時間と空間O(N)を達成することもできます。
しかし、実装のコストは(強連結成分分解ライブラリがなければ)こちらのほうが軽いです。
どちらもおすすめです。

## 実装例
D言語による実装例を示します。
```D
auto UF = UnionFind!int();
bool[int] NumberInCycle;

foreach (i; 0..N) {
    if (!UF.is_same_group(i, A[i])) {
        UF.merge_group(i, A[i]);
        continue;
    }

    // 閉路検出
    int cur = i;
    do {
        NumberInCycle[cur] = true;
        cur = A[cur];
    } while (cur != i);
}
```
このコードを実行したあとに`NumberInCycle`に存在する要素は、Functional Graphのいずれかの連結成分における閉路を構成する頂点である。

## 練習問題
練習問題を3問用意しました。ぜひ解いてみてください。
1問はまさにこのアルゴリズムを用いる問題ですが、他の2問は考察部分があり、Functional Graphに帰着するのは自明ではないです。

なので、ネタバレが嫌だという方は1問目だけ解くと良いです。

1. [ABC311C](https://atcoder.jp/contests/abc311/tasks/abc311_c)
<details>
<summary>解法</summary>
状況設定はほとんど同じです。
紹介したアルゴリズムを用いるだけで解くことができます。

なお、この問題に関しては列を求める必要があるため、強連結成分分解よりもUnionFindの方が直接的に求められます。
</details>

2. [ABC296E](https://atcoder.jp/contests/abc296/tasks/abc296_e)
<details>
<summary>解法</summary>
このゲームにおけるK_i回の操作は、Functional Graphにおける辺を移動していくことだとみなせます。

そこで、もしこのグラフのある閉路に頂点iが含まれる場合、ゲームiにおいて任意のK_iに対して勝てる初期配置が存在します。

逆に、そうでないとき、十分に大きなK_iを指定するだけで必ず青木くんが勝てます。(例えば、K_i=10^9を考えてみてください。)

すなわち、この問題はFunctional Graphの閉路に含まれる頂点の種類数を数える問題に帰着します。
</details>

3. [ABC256E](https://atcoder.jp/contests/abc256/tasks/abc256_e)
<details>
<summary>解法</summary>
人iから人X_iに有向辺を張ったグラフを考えます。すると、これはFunctional Graphになります。

うまく順列を選ぶことで、必ず一つの連結成分あたり一人以外の不満度を0にすることができます。
連結成分は閉路をなしますから、全員を不満度0にはできません。

そこで、連結成分の誰を不満にするかを選ぶ問題に帰着します。
これは明らかに最小の不満度を取る貪欲法が有効です。

よって、Functional Graphの閉路に含まれる頂点を列挙する問題に帰着されました。
</details>

## 終わりに
本アイディアはABC256Eの[nyaanさんの公式解説](https://atcoder.jp/contests/abc256/editorial/4135)で紹介されていたものです。