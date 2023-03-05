---
layout: post
title: "使用Key Indexing来避免初始化"
date: 2013-08-04 16:59
comments: true
categories: algorithms
---

在c/c++中, 使用`new`或者`malloc`来分配int数组是很快的, 因为没有初始化过程. 但是在使用时会有问题: 如果
不初始化, 如何知道哪一项的数据是有效的? 如果初始化, 那么效率又会比较低下.

假设有一个比较大数组a[0..n-1], 数组里每项是一个int变量, 但只有k项被使用(k << n). 如何设计一个数据
结构, 避免初始化a中所有的元素, 又能正确的访问数组中的每一项? 假设内存足够.

可以使用两个长度为n的辅助数组from和to. 例如:

            0 1 2 3 4 5 6 7 8 9
    a    = [?|1|?|?|2|?|?|9|?|?|]
    from = [?|0|?|?|2|?|?|1|?|?|]
    to   = [1|7|4|?|?|?|?|?|?|?|]
                  top

from数组表示的是, a[i]被初始化时, a数组中元素的个数, 也就是说, a[i]是第from[i]个被初始化的.

to数组表示的是, 第i个被初始化的元素, 它在a中的下标是to[i].

top表示的是to数组中有效元素的个数, 指向to数组中所有有效元素的下一项, 它的值等于目前a数组中有效元素的个数. 

图中为先后设置`a[1]=1; a[7]=9; a[4]=2;`后, a, from, to中的结果.

在设置一个元素时, 使用如下方法:

    a[i] = number;
    if (!is_initialized(i)) {
        from[i] = top;
        to[top] = i;
        top++;
    }
    
- `a[1] = 1`后, from[1] = 0, to[0] = 1, top = 1;
- `a[7] = 9`后, from[7] = 1, to[1] = 7, top = 2;
- `a[4] = 2`后, from[4] = 2, to[2] = 4, top = 3.

如何判断一个`a[i]`是否被初始化? 注意到如果`a[i]`被初始化了, 一定有`to[from[i]] == i`. 但是光有这个还不够, 因为`from[i]`可能指向`to`中某一个没初始化的位置, 使得`to[from[i]] == i`碰巧成立. 所以, 还得加上一个条件, 就是`from[i] < top`. 这样, 如果`from[i]`不在`0..top-1`之内, 那么就一定没被初始化. 如果from[i]在`0..top`之内, 有两种情况, 一种是i在1,4,7之内, 这样一定有`to[from[i]] == i`, 那么a[i]已经被初始化过了; 一种是i不在1,4,7之内, 那么一定有`to[from[i]] != i`. 因此要判断a[i]是否被初始化, 可以用`from[i] < top && to[from[i]] == top`.

这个技巧被称为key indexing, 在Programming Pearls里有提到过.

换个角度想想, key indexing其实是hashing的一个特例! 它的hash function就返回原本的key. 这里的i就是key, from数组其实是hash function, 而to数组告诉我们元素i是否被初始化过. 于是from, to这两个数组可以用一个hash table来代替:

    a[i] = number;
    init_hash[i] = true;

    if (init_hash[i]) {
        // 第i个元素被初始化过了
        ...
    }

因此, 如果需要更省空间话, 可以用hashing来优化.
