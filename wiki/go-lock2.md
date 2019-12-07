# Golang中的Lock的实现2

## 简介

这是阅读lock源码的第二版本

理解时一定要记住，同时会有多个goroutine会来要这把锁，所以锁的状态state是可能随时被更改的。

## 释义和思想

1. 关于Lock函数和RLock函数

```
Lock(): only one go routine read/write at a time by acquiring the lock.
RLock(): multiple go routine can read(not write) at a time by acquiring the lock.
```

2. 关于runtime的四个函数

```
runtime_canSpin: runtime_canSpin reports whether spinning makes sense at the moment.（是否可以自旋(类似发动机空转)，短暂占用cpu时间避免当前`goroutine`进入睡眠状态。)

runtime_doSpin: runtime_doSpin does active spinning.（让CPU自旋，即空转，配合`runtime_canSpin`使用。）

runtime_SemacquireMutex: SemacquireMutex is like Semacquire, but for profiling contended Mutexes.（睡眠，阻塞本goroutine）

runtime_Semrelease: Semrelease atomically increments *s and notifies a waiting goroutine.（唤醒一个阻塞的goroutine去获取锁）

```

#### 原理(参考3)

```
互斥锁有两种状态：正常状态和饥饿状态。
在正常状态下，所有等待锁的goroutine按照FIFO(先进先出)顺序等待。唤醒的goroutine不会直接拥有锁，而是会和新请求锁的goroutine竞争锁的拥有。新请求锁的goroutine具有优势：它正在CPU上执行，而且可能有好几个，所以刚刚唤醒的goroutine有很大可能在锁竞争中失败。在这种情况下，这个被唤醒的goroutine会加入到等待队列的前面。 如果一个等待的goroutine超过1ms没有获取锁，那么它将会把锁转变为饥饿模式。

在饥饿模式下，锁的所有权将从unlock的gorutine直接交给交给等待队列中的第一个。新来的goroutine将不会尝试去获得锁，即使锁看起来是unlock状态, 也不会去尝试自旋操作，而是放在等待队列的尾部。

如果一个等待的goroutine获取了锁，并且满足一以下其中的任何一个条件：(1)它是队列中的最后一个；(2)它等待的时候小于1ms。它会将锁的状态转换为正常状态。

正常状态有很好的性能表现，饥饿模式也是非常重要的，因为它能阻止尾部延迟的现象。
```

#### 阅读

```golang
// 基础操作
Locker *sync.Mutex
Locker.Lock()
defer Locker.Unlock()
```

```golang
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

// Package sync provides basic synchronization primitives such as mutual
// exclusion locks. Other than the Once and WaitGroup types, most are intended
// for use by low-level library routines. Higher-level synchronization is
// better done via channels and communication.
//
// Values containing the types defined in this package should not be copied.
package sync

import (
  "internal/race"
  "sync/atomic"
  "unsafe"
)

func throw(string) // provided by runtime

// A Mutex is a mutual exclusion lock. 
// The zero value for a Mutex is an unlocked mutex.
//
// A Mutex must not be copied after first use.
type Mutex struct {
  state int32 // 当前Mutex状态
  // 32~4数据位标记当前Mutex前面有等待的goroutine数量
  // 第3位标记是否是饥饿状态 第2位标记是否是唤醒状态 第1位标记是否是加锁状态
  sema  uint32 // 信号量 一个非负整数的全局变量
  // P(s)：如果s非零，那么P将s减1，并且立即返回。如果s为零，那么就挂起这个线程，直到s为非零。
  // V(s)：将s加1。如果有任何线程阻塞在P操作等待s非零，那么V将重启其中线程中的一个。
}

// A Locker represents an object that can be locked and unlocked.
type Locker interface {
  Lock()
  Unlock()
}

const (
  mutexLocked = 1 << iota // 等于二进制 1<<0 = 1，mutex is locked 对应state的第1位（是否是加锁状态）
  mutexWoken              // 等于二进制 1<<iota = 10，对应state的第2位（是否是唤醒状态）
  mutexStarving           // 等于二进制 1<<iota = 100, 对应state的第3位（是否是饥饿模式状态，锁模式）
  mutexWaiterShift = iota // 等于十进制的 3 标记等待的goroutine的数量的起始位，比如state左移三位即，state的32~4位记录等待的goroutine的数量的值

  // Mutex fairness.
  //
  // Mutex can be in 2 modes of operations: normal and starvation.
  // In normal mode waiters are queued in FIFO order, but a woken up waiter
  // does not own the mutex and competes with new arriving goroutines over
  // the ownership. New arriving goroutines have an advantage -- they are
  // already running on CPU and there can be lots of them, so a woken up
  // waiter has good chances of losing. In such case it is queued at front
  // of the wait queue. If a waiter fails to acquire the mutex for more than 1ms,
  // it switches mutex to the starvation mode.
  //
  // In starvation mode ownership of the mutex is directly handed off from
  // the unlocking goroutine to the waiter at the front of the queue.
  // New arriving goroutines don't try to acquire the mutex even if it appears
  // to be unlocked, and don't try to spin. Instead they queue themselves at
  // the tail of the wait queue.
  //
  // If a waiter receives ownership of the mutex and sees that either
  // (1) it is the last waiter in the queue, or (2) it waited for less than 1 ms,
  // it switches mutex back to normal operation mode.
  //
  // Normal mode has considerably better performance as a goroutine can acquire
  // a mutex several times in a row even if there are blocked waiters.
  // Starvation mode is important to prevent pathological cases of tail latency.
  starvationThresholdNs = 1e6 // 互斥锁转换状态的时间等待的临界值：一百万纳秒，也就是1ms
)

// Lock locks m.
// If the lock is already in use, the calling goroutine
// blocks until the mutex is available.
func (m *Mutex) Lock() {
  // m不是锁状态，申请锁处理逻辑，如果加锁成功则结束(goroutine申请锁成功)
  if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
    if race.Enabled {
      race.Acquire(unsafe.Pointer(m))
    }
    return
  }
  // m处于锁状态，申请锁处理逻辑（之后进来的goroutine申请锁，需要等待之前申请成功的goroutine释放锁）
  // Slow path (outlined so that the fast path can be inlined)
  m.lockSlow()
}

func (m *Mutex) lockSlow() {
  var waitStartTime int64
  // 标记是否是饥饿模式状态
  starving := false
  // 标记是否是已唤醒状态
  awoke := false
  // 标记自旋次数
  iter := 0
  // 先保存申请锁成功的m信息（m的state属性包含很多信息：是否加锁、是否唤醒，锁模式，等待申请锁的goroutine数量），后面简称old
  old := m.state
  for {
    // 表达式：mutexLocked|mutexStarving 加锁状态并且是饥饿模式
    // 表达式：old&(mutexLocked|mutexStarving) 获取old的锁状态和锁模式（即获取old的第1位和第3位的）
    // 表达式：old&(mutexLocked|mutexStarving) == mutexLocked 判断old的锁状态是加锁状态(等于1)并且锁模式是正常模式（等于0）
    // 表达式：runtime_canSpin(iter) 可以自旋
    // 条件：1.old是正常模式并且是加锁状态 2.可以自旋（iter超过一定次数就不能自旋）
    // 目的：m是正常模式、是加锁状态、并且可以自旋，自旋操作没有实际意义
    // Don't spin in starvation mode, ownership is handed off to waiters
    // so we won't be able to acquire the mutex anyway.
    if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
      // 表达式：old&mutexWoken == 0 old没有被唤醒
      // 表达式：old>>mutexWaiterShift != 0 old前面等待的goroutine的数量不能为0
      // 表达式：old|mutexWoken 如何将old唤醒的的方法
      // 表达式：atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) 尝试将old唤醒
      // 目的：自旋过程中，如果awoke变量不为true，old不是唤醒状态（即其他goroutine也没有唤醒old），并且前面有等待的goroutine，将old设置为唤醒状态，并将awoke变量设置为true
      // Active spinning makes sense.
      // Try to set mutexWoken flag to inform Unlock
      // to not wake other blocked goroutines.
      if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
        atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
        // 设置为唤醒标记为true，并且将old的设置为唤醒状态
        awoke = true
      }
      // 进行runtime自旋
      runtime_doSpin()
      // 累计自旋次数
      iter++
      // 其实就是将唤醒的state重新复制给old,进行下一次循环，m.state可能已经被改变
      old = m.state
      continue
    }
    
    // 先拷贝old得到new，用来设置新的状态
    new := old
    // Don't try to acquire starving mutex, new arriving goroutines must queue.
    // 如果m是正常模式，则先加锁（如果已经加锁了，再加一次也没事儿）
    if old&mutexStarving == 0 {
      // 先加锁
      new |= mutexLocked
    }

    // 如果old为锁状态或者处于饥饿模式，则将new的等待的goroutine数量+1
    if old&(mutexLocked|mutexStarving) != 0 {
      new += 1 << mutexWaiterShift
    }

    // The current goroutine switches mutex to starvation mode.
    // But if the mutex is currently unlocked, don't do the switch.
    // Unlock expects that starving mutex has waiters, which will not
    // be true in this case.
    // 如果等待时间超过了阈值，下方的某处代码会将 starving 设置为true，从而标记进入饥饿模式
    // 如果是时候进入饥饿模式并且old是锁定状态，则将new的设置为饥饿模式
    if starving && old&mutexLocked != 0 {
      new |= mutexStarving
    }
    
    // 如果唤醒标记是true
    if awoke {
      // The goroutine has been woken from sleep,
      // so we need to reset the flag in either case.
      // new的不是唤醒状态
      if new&mutexWoken == 0 {
        throw("sync: inconsistent mutex state")
      }
      // 如果goroutine已经被唤醒，将其重置为不唤醒状态
      new &^= mutexWoken
    }

    // 更新state
    if atomic.CompareAndSwapInt32(&m.state, old, new) {
      // old正常模式并且未被锁定并且并不是出于饥饿状态，跳出~~~（状态是从空闲到被获取）
      if old&(mutexLocked|mutexStarving) == 0 {
        break // locked the mutex with CAS
      }


      // If we were already waiting before, queue at the front of the queue.
      //runtime_SemacquireMutex(s *uint32, lifo bool)函数类似P操作，如果lifo为true则将等待goroutine插入到队列的前面。在这里，对于每一个到达的goroutine，如果CompareAndSawpInt32成功，并且到达时候如果锁出于锁定状态，那么将该goroutine插入到等待队列的最后，否则插入到最前面
      queueLifo := waitStartTime != 0
      if waitStartTime == 0 {
        waitStartTime = runtime_nanotime()
      }
      // 重新放回队首（queueLifo=true）
      runtime_SemacquireMutex(&m.sema, queueLifo, 1)
      // 如果时间超过1ms就会将starving设置为true，标记进入饥饿模式
      starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
      old = m.state
      
      // 判断被唤醒的线程是否是饥饿模式，也就是等待时间超过1ms
      if old&mutexStarving != 0 {
        // If this goroutine was woken and mutex is in starvation mode,
        // ownership was handed off to us but mutex is in somewhat
        // inconsistent state: mutexLocked is not set and we are still
        // accounted as waiter. Fix that.
        if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
          throw("sync: inconsistent mutex state")
        }
        // -1
        delta := int32(mutexLocked - 1<<mutexWaiterShift)
        if !starving || old>>mutexWaiterShift == 1 {
          // Exit starvation mode.
          // Critical to do it here and consider wait time.
          // Starvation mode is so inefficient, that two goroutines
          // can go lock-step infinitely once they switch mutex
          // to starvation mode.
          delta -= mutexStarving
        }
        atomic.AddInt32(&m.state, delta)
        break
      }
      

      // 继续循环
      awoke = true
      iter = 0
    } else {
      old = m.state
    }
  }

  if race.Enabled {
    race.Acquire(unsafe.Pointer(m))
  }
}

// Unlock unlocks m.
// It is a run-time error if m is not locked on entry to Unlock.
//
// A locked Mutex is not associated with a particular goroutine.
// It is allowed for one goroutine to lock a Mutex and then
// arrange for another goroutine to unlock it.
func (m *Mutex) Unlock() {
  if race.Enabled {
    _ = m.state
    race.Release(unsafe.Pointer(m))
  }

  // Fast path: drop lock bit.
  // 能够解除
  new := atomic.AddInt32(&m.state, -mutexLocked)
  if new != 0 {
    // Outlined slow path to allow inlining the fast path.
    // To hide unlockSlow during tracing we skip one extra frame when tracing GoUnblock.
    m.unlockSlow(new)
  }
}

func (m *Mutex) unlockSlow(new int32) {
  if (new+mutexLocked)&mutexLocked == 0 {
    throw("sync: unlock of unlocked mutex")
  }
  // 不为饥饿模式
  if new&mutexStarving == 0 {
    old := new
    for {
      // If there are no waiters or a goroutine has already
      // been woken or grabbed the lock, no need to wake anyone.
      // In starvation mode ownership is directly handed off from unlocking
      // goroutine to the next waiter. We are not part of this chain,
      // since we did not observe mutexStarving when we unlocked the mutex above.
      // So get off the way.
      // 如果等待的goroutine为零，或者已经被锁定、或者唤醒、或者已经变成饥饿状态，直接返回
      if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
        return
      }
      // Grab the right to wake someone.
      // 修改等待的goroutine数量，并且设置为唤醒
      new = (old - 1<<mutexWaiterShift) | mutexWoken
      // 恢复挂起的goroutine
      if atomic.CompareAndSwapInt32(&m.state, old, new) {
        // 通过信号量会唤醒一个阻塞的goroutine去获取锁.
        runtime_Semrelease(&m.sema, false, 1)
        return
      }
      old = m.state
    }
  } else {
    // Starving mode: handoff mutex ownership to the next waiter.
    // Note: mutexLocked is not set, the waiter will set it after wakeup.
    // But mutex is still considered locked if mutexStarving is set,
    // so new coming goroutines won't acquire it.
    // 通过信号量会唤醒第一个阻塞的goroutine去获取锁.
    runtime_Semrelease(&m.sema, true, 1)
  }
}

```

## 备注
参考7可以看一看，挺好的

## 参考
1. [https://golang.org/src/sync/runtime.go](https://golang.org/src/sync/runtime.go)
2. [https://studygolang.com/articles/19370](https://studygolang.com/articles/19370)
3. [https://colobu.com/2018/12/18/dive-into-sync-mutex/](https://colobu.com/2018/12/18/dive-into-sync-mutex/)
4. [https://segmentfault.com/a/1190000020036415](https://segmentfault.com/a/1190000020036415)
5. [https://github.com/cch123/golang-notes/blob/master/sync.md](https://github.com/cch123/golang-notes/blob/master/sync.md)
6. [https://studygolang.com/articles/17017](https://studygolang.com/articles/17017)
7. [https://github.com/bingoohuang/blog/issues/132](https://github.com/bingoohuang/blog/issues/132)
