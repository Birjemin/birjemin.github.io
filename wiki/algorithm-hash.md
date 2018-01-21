# 哈希表

## 简介
散列表（Hash table，也叫哈希表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。
给定表M，存在函数f(key)，对任意给定的关键字值key，代入函数后若能得到包含该关键字的记录在表中的地址，则称表M为哈希(Hash）表，函数f(key)为哈希(Hash) 函数。
![散列表示意图](http://upload.ouliu.net/i/20180121172555cwwxz.png)

## 基本概念
1. 若关键字为k，则其值存放在f(k)的存储位置上。由此，不需比较便可直接取得所查记录。称这个对应关系f为散列函数，按这个思想建立的表为散列表。
2. 对不同的关键字可能得到同一散列地址，即k1≠k2，而f(k1)=f(k2)，这种现象称为碰撞（英语：Collision）。具有相同函数值的关键字对该散列函数来说称做同义词。综上所述，根据散列函数f(k)和处理碰撞的方法将一组关键字映射到一个有限的连续的地址集（区间）上，并以关键字在地址集中的“像”作为记录在表中的存储位置，这种表便称为散列表，这一映射过程称为散列造表或散列，所得的存储位置称散列地址。
3. 若对于关键字集合中的任一个关键字，经散列函数映象到地址集合中任何一个地址的概率是相等的，则称此类散列函数为均匀散列函数（Uniform Hash function），这就是使关键字经过散列函数得到一个“随机的地址”，从而减少碰撞。

## 特点
```
数组的特点是：寻址容易，插入和删除困难；
而链表的特点是：寻址困难，插入和删除容易
```

## 散列法
元素特征转变为数组下标的方法就是散列法。三种比较常用的：
1. 除法散列法 
最直观的一种，上图使用的就是这种散列法，公式： 
      index = value % 16 
2. 平方散列法 
求index是非常频繁的操作，而乘法的运算要比除法来得省时，公式： 
      index = (value * value) >> 28   （右移，除以2^28。记法：左移变大，是乘。右移变小，是除。）
3. 斐波那契（Fibonacci）散列法

## 参考
1. [https://yq.aliyun.com/articles/353770?spm=a2c4e.11155472.0.0.6de58ccaqNgvfs](https://yq.aliyun.com/articles/353770?spm=a2c4e.11155472.0.0.6de58ccaqNgvfs)
2. [https://segmentfault.com/a/1190000007783628](https://segmentfault.com/a/1190000007783628)
3. [https://www.cnblogs.com/gaochundong/p/hashtable_and_perfect_hashing.html](https://www.cnblogs.com/gaochundong/p/hashtable_and_perfect_hashing.html)
4. [https://baike.baidu.com/item/%E5%93%88%E5%B8%8C%E8%A1%A8/5981869?fr=aladdin](https://baike.baidu.com/item/%E5%93%88%E5%B8%8C%E8%A1%A8/5981869?fr=aladdin)