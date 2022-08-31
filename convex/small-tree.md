<!--
title: new title
tags: クソ記事,初心者
private: false
-->

this article is written at 2022 08 31

# 最小木問題

酔ってる. だから最小木問題について考えよう.
グラフだとか, 頂点だとか辺だとか, 木だとかはなんとなくで読んでくれ.

ネットワーク上の全ての頂点を最も少ない道で繋ぐためにはどうしたらいいだろうか?
例えば, 各頂点がデータセンターでデータセンター同士を結ぶケーブルがあるとする.
無向グラフを想定している.
ケーブルはなるべく短くしたい.


つまり, ネットワークの各辺に重みがあり, その総和を最小にする全域木をどのように選ぶかが問題となる.

この問題の答えとなる全域木を最小木と呼ぶ.
最小木を求めるから最小木問題という.

余談だが, "力"とは何かを考えるから力学, "数"とは何かを考えるから数学, という話をどこかで聞いたことがある.

最小木について考えよう.
その前提となる全域木について考えよう.

果たして全域木とはなんだろう. いや, ふわっと理解してくれって言ったのは自分だけどもちょっと考えようぜ.
木と普通のグラフが違うのはループがあるかないかだ.
全域木っていうのは全ての頂点を通る(連結する)木のことだ.
<!-- つまり, 辺集合Tが全域木であるとは Tが閉路を含まず, グラフ(T, V)が連結であることと定義される. -->
<!-- ここでVは頂点集合を意味してる. -->

<!-- e\in E に長さd(e)が与えられているとする. 辺集合Tの全長はd(T)=\sum_{e\in E}d(e)として表される. -->
<!-- d(T)を最小にする全域木Tを最小木と呼ぶのだ. -->

こんなことを考えてると, 連結と全域木について同じものを考えてる気がする.

(命題)
グラフG=(V, E)に全域木が存在するための必要十分条件は, Gが連結であることである.

ところで木を見ていると, なんだかポキって折れそう.
つまり, 頂点集合を二分するような辺があるはず.
この概念は木だけでなく普通のグラフにもついても考えられそうだ.

頂点集合VをV'とV''に分ける. V'とV''を結ぶ辺をE(V', V'')と表し, カットセットと呼ぶ.
