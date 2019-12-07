# Golang中的Lock的实现1

## 简介

这是阅读lock源码的第一版本

## 释义和思想

刚开始理解挺麻烦的，尤其是并发之前的关系有点绕，不过先搞清如何使用锁，然后根据如果使用再去理解，就会好很多。

state为复合使用字段，第一位为锁状态，第二位为是否唤醒，其余位为等待的goroutine数量，

```
atomic.CompareAndSwapInt32(&m.state, p1, p2) 方法：判断state是否为p1，如果是则原子操作改为p2

runtime.Semacquire(&m.sema) 方法：申请信号量

runtime.Semrelease(&m.sema) 方法：释放信号量
```

## 阅读

```golang
// 基础操作（要理解如何使用锁！必须先lock，然后unlock）
// 不同的goroutine会存在竞争关系，具体如下：goroutine都在尝试lock（竞争），当有一个unlock时，又开始竞争
var Locker sync.Mutex
Locker.Lock()
defer Locker.Unlock()
```

```golang
package sync

import (
  "runtime"
  "sync/atomic"
)

// A Mutex is a mutual exclusion lock.
// Mutexes can be created as part of other structures;
// the zero value for a Mutex is an unlocked mutex.
type Mutex struct {
  state int32
  sema  uint32
}

// A Locker represents an object that can be locked and unlocked.
type Locker interface {
  Lock()
  Unlock()
}

// mutexLocked = 1 << iota = 1 << 0 = 1（二进制）第一位(下面称lock位)
// mutexWoken = 1 << iota = 1 << 1 = 10(二进制) 第二位（下面称woken位）
// mutexWaiterShift = iota = 2(十进制)，标记state存储 等待的goroutine数量 的位置（等待数量）
const (
  mutexLocked = 1 << iota // mutex is locked
  mutexWoken
  mutexWaiterShift = iota
)

// lock方法存在竞争！
// Lock locks m.
// If the lock is already in use, the calling goroutine
// blocks until the mutex is available.
func (m *Mutex) Lock() {
  // Fast path: grab unlocked mutex.
  // 第一次lock时就会成功，还有一种就是有一个unlock时，就会lock成功
  if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
    return
  }

  awoke := false
  for {
    // 现将state存储old
    old := m.state
    // 新的期望lock位为1
    new := old | mutexLocked
    // 如果old已经是lock状态，说明已经存在竞争了，等待数量+1
    if old&mutexLocked != 0 {
      new = old + 1<<mutexWaiterShift
    }
    // 是否是被信号唤醒的，如果是唤醒的，将重置唤醒位，不理解没事，可以调到unlock方法那里查看这个操作的意义，unlock中由将其唤醒的操作
    if awoke {
      // The goroutine has been woken from sleep,
      // so we need to reset the flag in either case.
      new &^= mutexWoken
    }
    // 尝试去修改state
    if atomic.CompareAndSwapInt32(&m.state, old, new) {
      // 如果old不是lock状态，说明可以获取，直接跳出
      if old&mutexLocked == 0 {
        break
      }
      // 都在lock，在这里信号量操作，其实就是等待。。。
      runtime.Semacquire(&m.sema)
      // 有一个会被唤醒，唤醒之后会重置woken位
      awoke = true
    }
  }
}

// 不存在竞争，不存在两个goroutine同时调用unlock！！！
// Unlock unlocks m.
// It is a run-time error if m is not locked on entry to Unlock.
//
// A locked Mutex is not associated with a particular goroutine.
// It is allowed for one goroutine to lock a Mutex and then
// arrange for another goroutine to unlock it.
func (m *Mutex) Unlock() {
  // Fast path: drop lock bit.
  // 将lock位重置，其实就是减掉，会影响到lock方法中的，比如这时候进来一个goroutine调用lock方法
  // 运行到if old&mutexLocked == 0 时，会率先获取到锁，而不需要申请信号量等待
  new := atomic.AddInt32(&m.state, -mutexLocked)
  // 这里校验一下，主要是否为了防止乱用，比如没有调用lock方法就直接调用unlock方法了
  if (new+mutexLocked)&mutexLocked == 0 {
    panic("sync: unlock of unlocked mutex")
  }

  old := new
  for {
    // If there are no waiters or a goroutine has already
    // been woken or grabbed the lock, no need to wake anyone.
    // 等待的goroutine数量为0 或者没有锁住或者已经被唤醒，说明都可以直接释放(不需要下面再次唤醒)
    if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken) != 0 {
      return
    }
    // Grab the right to wake someone.
    // 新期望 等待数量-1，并且唤醒
    new = (old - 1<<mutexWaiterShift) | mutexWoken
    if atomic.CompareAndSwapInt32(&m.state, old, new) {
      // 成功就直接release
      runtime.Semrelease(&m.sema)
      return
    }
    // 不成功则下一次再继续，这里不成功是因为state可能被lock方法改变了，woken位可能已经被唤醒，下一次循环时可以率先释放！
    old = m.state
  }
}
```

## 参考
1. [https://github.com/golang/go/blob/dd2074c82acda9b50896bf29569ba290a0d13b03/src/pkg/sync/mutex.go](https://github.com/golang/go/blob/dd2074c82acda9b50896bf29569ba290a0d13b03/src/pkg/sync/mutex.go)