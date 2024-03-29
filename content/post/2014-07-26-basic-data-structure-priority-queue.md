+++
title = "來複習一下資料結構 - Priority Queue"
date = "2014-07-26T18:30:00+08:00"
type = "post"
tags = ["Data structure"]
+++

Priority queue 有別於一般的 queue，每一個 element 都額外帶有優先值，愈優先的
(一般來說是優先值愈低的) 放在愈前面。Priority queue 是一個 Abstract Data Type，
有許多不同的實作方法，以 array 或 linked list 實作的時間複雜度如下表，

<table class="table">
  <thead>
    <tr>
      <th></th>
      <th>Insertion</th>
      <th>Removal</th>
      <th>Construction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>array or linked list</td>
      <td>$O(n)$</td>
      <td>$O(1)$</td>
      <td>$O(n^2)$</td>
    </tr>
  </tbody>
</table>

### Binary Search Tree

Priority queue 也可以用 binary search tree 來實作，前提是必須是一棵
self-balancing binary search tree，否則效果可能會很差，例如長成一棵
degenerate tree (每個 node 皆只有一個 child)，幾乎可以看作是一個 linked list。
以 self-balancing BST 實作的時間複雜度如下表，

<table class="table">
  <thead>
    <tr>
      <th></th>
      <th>Insertion</th>
      <th>Removal</th>
      <th>Construction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>array or linked list</td>
      <td>$O(n)$</td>
      <td>$O(1)$</td>
      <td>$O(n^2)$</td>
    </tr>
    <tr>
      <td>self-balancing BST</td>
      <td>$O(logn)$</td>
      <td>$O(logn)$</td>
      <td>$O(nlogn)$</td>
    </tr>
  </tbody>
</table>

### Heap

最常用來實作 priority queue 莫過於 heap，heap 是一種樹狀的資料結構，當每一個
parent nodes 都大於 child nodes 時，稱為 max heap，反之，則稱為 min heap。
Heap 中又以 binary heap 最常見，binary heap 由兩個特性構成

1. complete binary tree
2. 上述的 heap 特性

以 binary heap 實作的時間複雜度如下表，要特別注意的是，若以一般建 tree 的方式來
建 binary heap，時間複雜度其實是 $O(nlogn)$，但若以 bottom-up 的方式 heapify，
則可以壓在 $O(n)$ 以內，大致上的概念如下，細節可以參考 [這個][1] 和 [這個][2]

1. 第 $h$ 層，最多需作 $2^h * 0$  次 heapify
2. 第 $h - 1$ 層，最多需作 $2^{h-1} * 1$ 次 heapify
3. 第 $h - 2$ 層，最多需作 $2^{h-2} * 2$ 次 heapify
4. 第 $h - i$ 層，最多需作 $2^{h-i} * i$ 次 heapify
5. 第 $h - h$ 層，最多需作 $2^{h-h} * h$ 次 heapify

總共所需的 heapify 次數為
$\sum\_{i=0}^{h} i2^{h-i}
= \sum\_{i=0}^{h} i\frac{2^h}{2^i}
= 2^h\sum\_{i=0}^{h} \frac{i}{2^i}
\leq 2^h\sum\_{i=0}^{\infty} \frac{i}{2^i}
\leq 2^h * 2
= 2^{h+1} \in O(n)$


<table class="table">
  <thead>
    <tr>
      <th></th>
      <th>Insertion</th>
      <th>Removal</th>
      <th>Construction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>array or linked list</td>
      <td>$O(n)$</td>
      <td>$O(1)$</td>
      <td>$O(n^2)$</td>
    </tr>
    <tr>
      <td>self-balancing BST</td>
      <td>$O(logn)$</td>
      <td>$O(logn)$</td>
      <td>$O(nlogn)$</td>
    </tr>
    <tr>
      <td>binary heap</td>
      <td>$O(logn)$</td>
      <td>$O(logn)$</td>
      <td>$O(n)$</td>
    </tr>
  </tbody>
</table>

Heap 常以 array 的方式儲存，下圖是一個 complete binary tree 以 array 儲存的示
意，$ith$ node 的 children 和 parent 可以很容易的算出來

* children: $2i + 1, 2i + 2$
* parent: $\lfloor \frac{i - 1}{2} \rfloor$

![A complete binary tree stored in an array](http://upload.wikimedia.org/wikipedia/commons/thumb/8/86/Binary_tree_in_array.svg/370px-Binary_tree_in_array.svg.png)

本來想找找 linux kernel 裡面使用到 heap 的部份出來看，翻了一下
[cstheory.stackexchange.com 上有名的討論串][3]，發現似乎只有 cgroup 有用到，而
且還是 reference kernel 2.6 的 source code，在 master branch 找了一下卻沒找到，
才發現原來[在今年年初被拔掉了][4]，不過 heap library 還在，可以參考
[prio\_heap.c][5] 和 [prio\_heap.h][6]。

### References

* [Wikipedia - Priority Queue](http://en.wikipedia.org/wiki/Priority_queue)
* [Wikipedia - Heap](http://en.wikipedia.org/wiki/Heap_(data_structure))
* [Wikipedia - Binary Heap](http://en.wikipedia.org/wiki/Binary_heap)

[1]: http://en.wikipedia.org/wiki/Binary_heap#Building_a_heap
[2]: http://www.cs.umd.edu/~meesh/351/mount/lectures/lect14-heapsort-analysis-part.pdf
[3]: http://cstheory.stackexchange.com/questions/19759/core-algorithms-deployed
[4]: https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/commit/?id=889ed9ceaa97bb02bf5d7349e24639f7fc5f4fa0
[5]: https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/tree/lib/prio_heap.c?id=889ed9ceaa97bb02bf5d7349e24639f7fc5f4fa0
[6]: https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/tree/include/linux/prio_heap.h?id=889ed9ceaa97bb02bf5d7349e24639f7fc5f4fa0
