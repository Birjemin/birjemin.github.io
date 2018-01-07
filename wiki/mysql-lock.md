# MYSQL GET_LOCK锁

## 示例：
```
<?php
    # CodeIgniter/system/libraries/Session/drivers/CI_Session_database_driver.php
    $this->_db->query("SELECT GET_LOCK('".$arg."', 300) AS ci_session_lock")->row()->ci_session_lock);
    $this->_db->query("SELECT RELEASE_LOCK('".$this->_lock."') AS ci_session_lock")->row()->ci_session_lock)
?>
```

## 简介
GET_LOCK(str,timeout)

`释义`：Tries to obtain a lock with a name given by the string str, using a timeout of timeout seconds. A negative timeout value means infinite timeout. The lock is exclusive. While held by one session, other sessions cannot obtain a lock of the same name.

试图分配一个名为name，超时时间为timeout秒的锁。timeout为负数时表示永不超时。该锁是独立的，即当有一个连接在使用名为str的锁时，其他的连接无法获得该锁。

`返回值`：Returns 1 if the lock was obtained successfully, 0 if the attempt timed out (for example, because another client has previously locked the name), or NULL if an error occurred (such as running out of memory or the thread was killed with mysqladmin kill).

1:获得成功； 0:获取超时（比如有其他客户端已经锁住了该锁）； NULL:发生错误（比如内存溢出或者线程被mysqladmin杀掉了）。

`释放锁`：A lock obtained with GET_LOCK() is released explicitly by executing RELEASE_LOCK() or implicitly when your session terminates (either normally or abnormally). Lock release may also occur with another call to GET_LOCK():

执行RELEASE_LOCK()或者你的连接中断（正常或非正常情况）都可以直接释放通过GET_LOCK()获得的锁。其他连接使用GET_LOCK()时可能也会释放。

> 注意：With the capability of acquiring multiple named locks in MySQL 5.7.5, it is possible for a single statement to acquire a large number of locks. For example:

```
INSERT INTO ... SELECT GET_LOCK(t1.col_name) FROM t1;
```

These types of statements may have certain adverse effects. For example, if the statement fails part way through and rolls back, locks acquired up to the point of failure will still exist. If the intent is for there to be a correspondence between rows inserted and locks acquired, that intent will not be satisfied. Also, if it is important that locks are granted in a certain order, be aware that result set order may differ depending on which execution plan the optimizer chooses. For these reasons, it may be best to limit applications to a single lock-acquisition call per statement.

伴随着MySQL 5.7.5中的获取多重命令锁的能力，可能会使单个语句获取很多个锁，比如：

```
INSERT INTO ... SELECT GETLOCK(t1.colname) FROM t1;
```

这列类型的语句可能潜藏不利的因素。比如，如果这个语句执行失败发生回滚，虽然语句失败但是锁依然存在。如果你本来的意图是行插入成功和锁获取成功，这个时候就不满足不了。比如如果使用锁来控制一定的命令顺序，请注意根据优化器选择的执行计划可能会使结果集有所不同。由于这些原因，最好将应用限制为获取每个语句的单个锁。

## 参考：
1. [http://blog.csdn.net/tangtong1/article/details/51792617 ](http://blog.csdn.net/tangtong1/article/details/51792617)
2. [https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html](https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html)
3. [https://dev.mysql.com/doc/refman/5.7/en/locking-service.html](https://dev.mysql.com/doc/refman/5.7/en/locking-service.html)