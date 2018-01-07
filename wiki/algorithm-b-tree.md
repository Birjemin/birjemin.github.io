# B树和B+树

## 简介

在计算机科学中，B树（英语：B-tree）是一种自平衡的树，能够保持数据有序。这种数据结构能够让查找数据、顺序访问、插入数据及删除的动作，都在对数时间内完成。B树，概括来说是一个一般化的二叉查找树（binary search tree），可以拥有多于2个子节点。与自平衡二叉查找树不同，B树为系统大块数据的读写操作做了优化。B树减少定位记录时所经历的中间过程，从而加快存取速度。B树这种数据结构可以用来描述外部存储。这种数据结构常被应用在数据库和文件系统的实现上。

## 定义

```
B-Tree is a self-balanced search tree with multiple keys in every node and more than two children for every node.
```
B树是一种自平衡的搜索树,具有多每一个节点node都有多个keys，并且每个节点有2个子节点或者多于2个子节点。

```
A B+ tree is an N-ary tree with a variable but often large number of children per node. A B+ tree consists of a root, internal nodes and leaves.The root may be either a leaf or a node with two or more children
```
B+树是一个n叉排序树，通常每个节点有多个孩子，一棵B+树包含一个根节点、多个内部节点和叶子节点。根节点可能是一个叶子节点，也可能是一个包含两个或两个以上孩子节点的节点。

## 特征

B-Tree of Order m has the following properties...
* `Property #1` - All the leaf nodes must be at same level.
* `Property #2` - All nodes except root must have at least [m/2]-1 keys and maximum of m-1 keys.
* `Property #3` - All non leaf nodes except root (i.e. all internal nodes) must have at least m/2 children.
* `Property #4` - If the root node is a non leaf node, then it must have at least 2 children.
* `Property #5` - A non leaf node with n-1 keys must have n number of children.
* `Property #6` - All the key values within a node must be in Ascending Order.

## 搜寻方法

In a B-Ttree, the search operation is similar to that of Binary Search Tree. In a Binary search tree, the search process starts from the root node and every time we make a 2-way decision (we go to either left subtree or right subtree). In B-Tree also search process starts from the root node but every time we make n-way decision where n is the total number of children that node has. In a B-Ttree, the search operation is performed with O(log n) time complexity. The search operation is performed as follows...

* Step 1: Read the search element from the user
* Step 2: Compare, the search element with first key value of root node in the tree.
* Step 3: If both are matching, then display "Given node found!!!" and terminate the function
* Step 4: If both are not matching, then check whether search element is smaller or larger than that key value.
* Step 5: If search element is smaller, then continue the search process in left subtree.
* Step 6: If search element is larger, then compare with next key value in the same node and repeate step 3, 4, 5 and 6 until we found exact match or comparision completed with last key value in a leaf node.
* Step 7: If we completed with last key value in a leaf node, then display "Element is not found" and terminate the function.

大致的意思就是查询的条件A,和root节点值比较，如果相等则success,不相等则判断是否小于该节点，小于则查询左子树，大于则查询此时节点的下一个值，以此类推，直到此时节点最后一个值还是小于A,则查询右子树，直到找到或者结束查找。

## 插入方法

Insertion Operation in B-Tree
In a B-Tree, the new element must be added only at leaf node. That means, always the new keyValue is attached to leaf node only. The insertion operation is performed as follows...

* Step 1: Check whether tree is Empty.
* Step 2: If tree is Empty, then create a new node with new key value and insert into the tree as a root node.
* Step 3: If tree is Not Empty, then find a leaf node to which the new key value cab be added using Binary Search Tree logic.
* Step 4: If that leaf node has an empty position, then add the new key value to that leaf node by maintaining ascending order of key value within the node.
* Step 5: If that leaf node is already full, then split that leaf node by sending middle value to its parent node. Repeat tha same until sending value is fixed into a node.
* Step 6: If the spilting is occuring to the root node, then the middle value becomes new root node for the tree and the height of the tree is increased by one.

![B树插入数据的gif](https://files.cnblogs.com/files/yangecnu/btreebuild.gif)

![B+树插入数据的gif](https://files.cnblogs.com/files/yangecnu/Bplustreebuild.gif)

## 删除方法

参考8的链接,有图有真相，这里就摘录一些重点文字条件吧。

```
k：删除的值
x: k所在的节点
x.n: k所在节点的长度
t: k所在节点的层级
```

* If k is in the node x which is a leaf and x.n>=t,
Here you can straightaway delete k from x.

* If k is in the node x which is a leaf and x.n == (t-1)
  
  ```
  Find the immediate sibling y of x, the extreme key m of y, the parent p of x and the parent key l of k
  If y.n >= t:
      Move l into x
      Move m into p
      Delete k from x
  ```

  ```
  Find the immediate sibling y of x, the extreme key m of y, the parent p of x and the parent key l of k
  If y.n == t-1:
      Merge x and y
      Move down l to the new node as the median key
      Delete k from the new node
  ```

* If k is in the node x and x is an internal node (not a leaf)

  ```
  Find the child node y that precedes k (the node which is on the left side of k)
  If y.n >= t:
      Find the key k’ in y which is the predecessor of k
      Delete k’ recursively. (Here k’ can be another internal node as well. So we have to delete it recursively in the same way)
      Replace k with k’ in x
  ```

  ```
  Find the child node y that precedes k
  If y.n < t (or y.n == (t-1)):
      Find the child node z that follows k (the node which is on the right side of k)
      If z.n >= t:
          Find k’’ in z which is the successor of k
          Delete k’’ recursively
          Replace k with k’’ in x
  ```

  ```
  Find the child node y that precedes k and the child node z that follows k
  If y.n == (t-1) AND z.n == (t-1):
      Merge k and z to y
      Free memory of node z
      Recursively delete k from y
  ```

## B-树和B+树区别
B和B+树的区别在于，B+树的非叶子结点只包含key信息，不包含data，所有的叶子结点和相连的节点使用链表相连，便于区间查找和遍历。
![B树和B+树的区别](http://upload.ouliu.net/i/201801072121242o53t.png)

## 参考：
1. [https://zh.wikipedia.org/wiki/B%E6%A0%91](https://zh.wikipedia.org/wiki/B%E6%A0%91)
2. [http://blog.codinglabs.org/articles/theory-of-mysql-index.html](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)
3. [http://btechsmartclass.com/DS/U5_T3.html](http://btechsmartclass.com/DS/U5_T3.html)
4. [http://www.cnblogs.com/yangecnu/p/Introduce-B-Tree-and-B-Plus-Tree.html](http://www.cnblogs.com/yangecnu/p/Introduce-B-Tree-and-B-Plus-Tree.html)
5. [https://www.geeksforgeeks.org/b-tree-set-1-introduction-2/](https://www.geeksforgeeks.org/b-tree-set-1-introduction-2/)
6. [https://www.geeksforgeeks.org/b-tree-set-1-insert-2/](https://www.geeksforgeeks.org/b-tree-set-1-insert-2/)
7. [https://www.geeksforgeeks.org/b-tree-set-3delete/](https://www.geeksforgeeks.org/b-tree-set-3delete/)
8. [https://medium.com/@vijinimallawaarachchi/all-you-need-to-know-about-deleting-keys-from-b-trees-9090f3334b5c](https://medium.com/@vijinimallawaarachchi/all-you-need-to-know-about-deleting-keys-from-b-trees-9090f3334b5c)