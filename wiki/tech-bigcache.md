## 简介
Efficient cache for gigabytes of data written in Go

## 特点
The BigCache provides shards, eviction and it omits GC for cache entries. As a result it is very fast cache even for large number of entries.
Freecache is the only one of the available in-memory caches in Go which provides that kind of functionality(only one ???)

1. 使用分片
2. 删除机制
3. 关闭GC

## 思想

### 设计思想
Our service would receive many requests concurrently, so we needed to provide concurrent access to the cache. The easy way to achieve that would be to put sync.RWMutex in front of the cache access function to ensure that only one goroutine could modify it at a time. However other goroutines which would also like to make modifications to it, would be blocked, making it a bottleneck. To eliminate this problem, shards could be applied. The idea behind shards is straightforward. Array of N shards is created, each shard contains its own instance of the cache with a lock. When an item with unique key needs to be cached a shard for it is chosen at first by the function hash(key) % N. After that cache lock is acquired and a write to the cache takes place. Item reads are analogue. When the number of shards is relatively high and the hash function returns properly distributed numbers for unique keys then the locks contention can be minimized almost to zero. This is the reason why we decided to use shards in the cache

使用分片的思想，每一个分片有自己的对象和锁，每次操作时使用hash方法` hash(key) % N`进行选择相应的分片（问题1.这是不是意味着一旦N指定，就不能改变？？）

### 删除机制
The simplest way to evict elements from the cache is to use it together with FIFO queue. When an entry is added to the cache then two additional operations take place:

- An entry containing a key and a creation timestamp is added at the end of the queue.
- The oldest element is read from the queue. Its creation timestamp is compared with current time. When it is later than eviction time, the element from the queue is removed together with its corresponding entry in the cache.
- 
Eviction is performed during writes to the cache since the lock is already acquired.

在写数据时（分片锁已获得，其他线程只能等待），使用FIFO算法进行删除元素，当添加一个元素时，将会执行以下操作：
- （元素+创建时间）会追加到FIFO队首
- 判断队尾元素是否超时，从队列中移除该元素、删除数据

### 关于垃圾回收的优化
Golang的GC(垃圾回收机制)在对Map标记和扫描阶段即有可能会出发GC机制，并且在Map达到千万级别时对性能消耗是巨大的（文中举例：`Metrics indicated that there were over 40 mln objects in the heap and GC mark and scan phase took over four seconds`），所以如果关闭GC?

- GC作用于heap(堆), 所以可以关掉堆，[仓库](https://godoc.org/github.com/glycerine/offheap)，该仓库提供了新的的方式（非堆）来分配和释放内存的内置方法来管理内存。（问题2.需要关注怎么实现的）
- 第二个仓库，[仓库](https://github.com/coocood/freecache)，通过减少指针的数量来达到零GC开销（问题3.如何减少），将key和value保存到ring_buffer（问题4.这是啥）中，使用索引切片查找条目。
- 移除GC，Go version 1.5中给出了方式和条件(This optimization states that if map without pointers in keys and values is used then GC will omit its content)

Bigcache采用了第三种方式。

### 关于碰撞
BigCache does not handle collisions. When new item is inserted and it's hash collides with previously stored item, new item overwrites previously stored value.

没有处理，会被复写。

### 关于评测
参考6，文中提到：`BigCache divides the data into shards based on the hash of the key. Each shard contains a map and a ring buffer`，看了一下源码，并不是环，而是队列。

## 备注
1. 问题1解答：
代码位置[code](https://github.com/allegro/bigcache/blob/master/fnv.go)，其实是对参考3的实现，N被指定。这是也能理解为什么说`When the number of shards is relatively high and the hash function returns properly distributed numbers for unique keys then the locks contention can be minimized almost to zero`，因为N真的好大。。

2. 问题2解答：实现了自己的一套方式，不使用golang提供的Malloc和Free方式。
`Writing our own Malloc() and Free() implementation (see malloc.go) which requests memory directly from the OS. The keys, the values, and the entire hash table itself is kept in this off-heap storage.`

3. 问题3解答：和Bigcache类似，分成了256块，每块2个指针，指针数目是一个定值。阻塞等待还有有很大几率发生的[code](https://github.com/coocood/freecache/blob/master/cache.go)，set方法中确实如此，并发256以上就发生了竞争）
`The data set is sharded into 256 segments by the hash value of the key. Each segment has only two pointers, one is the ring buffer that stores keys and values, the other one is the index slice which used to lookup for an entry. Each segment has its own lock, so it supports high concurrent access.`

4. 问题4解答：循环缓冲（环形数据结构，参考5）,freecache使用ring buffer数据结构来实现存储键值对和查找。

## 参考
1. [https://github.com/allegro/bigcache](https://github.com/allegro/bigcache)
2. [https://allegro.tech/2016/03/writing-fast-cache-service-in-go.html](https://allegro.tech/2016/03/writing-fast-cache-service-in-go.html)
3. [https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function)
4. [https://github.com/glycerine/offheap](https://github.com/glycerine/offheap)
5. [https://en.wikipedia.org/wiki/Circular_buffer](https://en.wikipedia.org/wiki/Circular_buffer)
6. [https://dgraph.io/blog/post/caching-in-go/](https://dgraph.io/blog/post/caching-in-go/)