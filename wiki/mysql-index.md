# MYSQL索引简介

## 示例：

```
<?php
    # CodeIgniter/system/libraries/Session/drivers/CI_Session_database_driver.php
    $this->_db->query("SELECT GET_LOCK('".$arg."', 300) AS ci_session_lock")->row()->ci_session_lock);
    $this->_db->query("SELECT RELEASE_LOCK('".$this->_lock."') AS ci_session_lock")->row()->ci_session_lock)
?>
```
## 简介


1. [http://blog.csdn.net/tangtong1/article/details/51792617 ](http://blog.csdn.net/tangtong1/article/details/51792617)
2. [https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html](https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html)
3. [https://dev.mysql.com/doc/refman/5.7/en/locking-service.html](https://dev.mysql.com/doc/refman/5.7/en/locking-service.html)