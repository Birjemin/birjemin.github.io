## Go Slices: usage and internals
简单总结一下Slices

### 数组
#### 简述
举个🌰:
```golang
var a [4]int
a[0] = 1
i := a[0]
// i == 1
```

补充，数组不是一定要指定长度：
```golang
// 指定长度
b := [2]string{"Penn", "Teller"}
// 由编译器计算长度
b := [...]string{"Penn", "Teller"}
```
#### 与C的区别
- 数组是值，将一个数组赋值到另一个数组中会是全部拷贝。
- 特别是，数组在函数调用时是传值调用，而不是引用调用。
- 大小size是数组的一个属性，比如[10]int 和[20]int 截然不同

### 数组内部结构
该数组a的结构为：
![内存图](https://blog.golang.org/slices-intro/slice-array.png)

可以看出数组是一段连续的内存（有序，可以通过index访问指定的内存值）。

### 切片
由于数组灵活性偏低（需要通过索引访问，且需要注意越界），所以Golang中提供了切片这个数据结构。本质上来说切片是数组的抽象（对数组的描述）。

举个🌰
```golang
letters := []string{"a", "b", "c", "d"}
func make([]T, len, cap) []T
```

len为切片长度，cap为深度，默认初始化时cap等于len。

常用的使用方式：
```golang
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b

// 如何为数组创建切片
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```
### 切片内部结构
本质上来说切片是数组的抽象（对数组一部分的描述）。包括一个指向数组数组位置的指针，数组一部分的长度和它的深度（最大长度）。

![切片结构](https://blog.golang.org/slices-intro/slice-struct.png)

当执行`s := make([]byte, 5)`时
![切片结构](https://blog.golang.org/slices-intro/slice-1.png)

再执行`s = s[2:4]`时
![切片结构](https://blog.golang.org/slices-intro/slice-2.png)

可以看出，当cap小于数组最大长度时，即改变切片值也会影响数组值
```golang
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:]
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'}
```

### 持续增加切片容量的情况
举个例子
```golang
package main

import (
	"log"
)

func main() {

	b := []int{1,2,3}
    c := b[1:]
    // c := b[0:2]
	c[0] = 3

	log.Println("b: ", b, len(b), cap(b))
	log.Println("c: ", c, len(c), cap(c))

  // append函数做了啥？
	c = append(c, 4)
	log.Println("b: ", b, len(b), cap(b))
	log.Println("c: ", c, len(c), cap(c))
}
```
问题：给切片增加新的元素会发生什么？len和cap变成了多少？

#### append函数的作用
```golang
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) { // if necessary, reallocate
        // allocate double what's needed, for future growth.
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
p := []byte{2, 3, 5}
p = AppendByte(p, 7, 11, 13)
// p == []byte{2, 3, 5, 7, 11, 13}


a := make([]int, 1)
// a == []int{0}
a = append(a, 1, 2, 3)
// a == []int{0, 1, 2, 3}
```

#### copy函数的作用
重新生成一个新的切片，将原切片的值重新复制到新的切片上。（此时两个切片分别操作两个不同的数组）
```golang
t := make([]int, len(s), (cap(s)+1)*2)
copy(t, s)
s = t

t := make([]int, len(s), (cap(s)+1)*2) // +1 in case cap(s) == 0
for i := range s {
  t[i] = s[i]
}
s = t
```

#### 补充
一个Filter函数，重新生成一个切片。
```golang 
package main

import (
	"log"
)

func main() {

  b := []int{1,2,3}
	d := Filter(b, func(i int) bool {
		return i % 2 == 0
	})
	log.Println("d: ", d, len(d), cap(d))
}

// Filter returns a new slice holding only
// the elements of s that satisfy fn()
func Filter(s []int, fn func(int) bool) []int {
    var p []int // == nil
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

### 注意点
#### 原数据常驻内存
切片重新分配不会改变原有的数组，除非数组没有被引用。所以有时候会导致数组中的所有数据常驻内存，但是切片只是用了一点点的情况。

举个🌰
```golang
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}

// 使用以下方式回收使得GC回收b
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```

#### 切片在改变了原有的值
尤其是在dfs和bfs遍历中（leetcode39题），回溯时元切片会被改变

```golang
package main

import (
    "sort"
)

func combinationSum(candidates []int, target int) [][]int {
    var ret [][]int
    sort.Ints(candidates)
    dfs(&ret, candidates, []int{}, target)
    return ret
}

func dfs(ret *[][]int, candidates []int, solution []int, target int) {
    if target == 0 {
        // *ret = append(*ret, solution)
        b := make([]int, len(solution))
        copy(b, solution)
        *ret = append(*ret, b)
        return
    }
    for i, v := range candidates {
        // adding this expression could reduce the memory storage
        if v > target {
            return
        }
        dfs(ret, candidates[i:], append(solution, v), target-v)
    }
}
```

## 参考
1. [https://www.ardanlabs.com/all-posts/](https://www.ardanlabs.com/all-posts/)
2. [https://www.ardanlabs.com/blog/2013/08/understanding-slices-in-go-programming.html](https://www.ardanlabs.com/blog/2013/08/understanding-slices-in-go-programming.html)
3. [https://golang.org/doc/effective_go.html#slices](https://golang.org/doc/effective_go.html#slices)
4. [https://golang.org/ref/spec](https://golang.org/ref/spec)
5. [https://halfrost.com/go_slice/](https://halfrost.com/go_slice/)